# The Patch, Explained

## What It Does

The bootloader RSA signature verifier returns 0 for a valid signature and non-zero for an invalid one. The upgrade dispatcher then branches on that result:

```asm
80d02ebc  jal   FUN_80d02638     ; run the RSA verifier
80d02ec4  bne   v0, zero, reject ; if result != 0, reject the firmware
```

The patch replaces `bne` with `nop`:

```asm
80d02ec4  nop                    ; ignore the result — always continue
```

Command:
```
w 0x80d02ec4 0x00000000
```

| | Hex value | Meaning |
|---|---|---|
| Original | `14 40 00 0B` | Branch if v0 ≠ 0 (i.e. reject invalid firmware) |
| Patched | `00 00 00 00` | Do nothing (nop) |

## Why It Works

The signature verifier does everything: certificate ID lookup, RSA verification, post-decompression checksum. But its output — a return value in register `v0` — is only used in one place: the branch at `0x80d02ec4`.

Patching that one branch means the verifier's verdict is never acted upon. The code falls through to "Check DigitalSigned Ok" regardless of what the verifier found.

## Why The Key Doesn't Matter

The original approach to this problem was to find the RSA public key in RAM and replace it with a key whose private key was known — then sign the target firmware with that private key. This requires:

1. Finding the key in RAM (it may be in multiple locations, cached in ways that aren't obvious)
2. Overwriting every copy (~1500 writes in some versions)
3. Knowing the exact key format and structure
4. Having a matching signed firmware

The branch patch sidesteps all of this. The key can be anything. The verifier can return whatever it wants. The result is ignored.

**Simpler and more correct.**

## What Is NOT Patched

**Format detection at `0x80d02ee4`:**
```asm
80d02ee4  bne   v1, v0, LAB_80d02f40
```
This branch checks whether the uploaded file is `.bin` format (magic `0xFFFFFFFF` at offset `0x80`). For `.bin` files, `v1 == v0 == 0xFFFFFFFF`, so the branch is **not taken** — the correct writer path runs.

Patching this instruction was tried during development. It sends `.bin` files to the `.w` handler which doesn't recognise them — flash write fails. **Do not patch `0x80d02ee4`.**

## Why The Patch Is RAM-Only

The bootloader decompresses itself from flash into RAM at boot. The patched instruction lives in the decompressed RAM copy. Flash is never modified.

This has two consequences:

1. **Non-persistent** — every power-cycle restores the original bootloader. The patch must be reapplied each session. This takes about 10 seconds.

2. **Safe** — you cannot permanently damage the bootloader with this approach. The worst outcome is a failed firmware flash, which leaves the original firmware in flash and the original bootloader fully intact.

## Making The Patch Persistent (Not Recommended)

To survive reboots, the bootloader in flash would need to be modified:

1. Extract the bootloader region (`0x000000` to `0x030000`)
2. Decompress the LZMA stream starting at `0x006020`
3. Patch offset `0x2EC4` in the decompressed payload
4. Recompress with the original LZMA parameters (lc=3, lp=0, pb=2, dict=32 MiB)
5. Rebuild the bootloader region with the original pre-LZMA header
6. Write back to flash via xmodem or hardware programmer

Risks:
- Unknown checksum or signature on the bootloader region itself
- Unknown boot ROM verification that may reject a modified bootloader
- A failed write to the bootloader region bricks the device — hardware programmer required for recovery

The RAM patch takes 10 seconds per session. It is not worth the risk.

## The Full History Of Approaches

**Attempt 1 — RSA key replacement:**
Worked on the B011 bootloader. Required locating the key in RAM and overwriting ~1500 bytes. When a newer B012 bootloader appeared on a replacement Orange firmware, the key location changed and the approach became unreliable.

**Attempt 2 — Multiple instruction patches + key replacement:**
Added several code patches alongside the key replacement. Complex. Fragile. Not well documented due to the excitement of first success.

**Attempt 3 — Branch patch only (this project):**
Observation: the RSA key location is irrelevant if the branch that checks the result is patched. One write. No key replacement. Works on B012. Tested multiple times.

The insight came from asking the question: *what is the minimum change needed?* The answer was a single NOP at the point where the verifier's return value is acted upon.
