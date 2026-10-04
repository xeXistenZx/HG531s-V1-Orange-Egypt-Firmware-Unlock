# Bootloader Analysis

Reverse-engineering notes for the B012 Realtek RTL867X Loader.

## Overview

The bootloader decompresses itself from flash (LZMA) into RAM at `0x80D00000` on every boot. All patches modify this RAM copy only — flash is never touched. Every power-cycle restores the original bootloader.

## Address Map

Base: `0x80D00000`, MIPS big-endian 32-bit (analysed with Ghidra).

| Address | Function |
|---|---|
| `0x80d00000` | Entry / reset vector |
| `0x80d02434` | Banner printer |
| `0x80d02488` | Web / multicast init |
| `0x80d02638` | **Signature verifier** |
| `0x80d027f0` | **Firmware writer** |
| `0x80d02ac0` | Bootloader writer |
| `0x80d02cac` | Erase handler |
| `0x80d02d30` | **Format detector / writer dispatch** |
| `0x80d02de4` | Config / factory image writer |
| `0x80d02e84` | **Upgrade dispatcher** ← contains the patch point |
| `0x80d03070` | Decompressor |
| `0x80d0335c` | Flash timing retry helper |
| `0x80d038c0` | Boot / multicast listener |
| `0x80d03dfc` | xmodem handler |
| `0x80d04660` | Command dispatcher |

## The Upgrade Dispatcher (`0x80d02e84`)

This is the function that receives uploaded firmware and decides whether to accept or reject it.

```asm
; --- Signature verification ---
80d02ebc  jal   FUN_80d02638          ; call RSA verifier
80d02ec0  move  a1, s0                ; (delay slot)
80d02ec4  bne   v0, zero, reject      ; if v0 != 0, firmware rejected  ← PATCH HERE
                                       ;   original: 14 40 00 0B
                                       ;   patched:  00 00 00 00 (nop)
80d02ec8  nop                         ; (delay slot)
80d02ecc  ; print "Check DigitalSigned Ok"
80d02ed8  ; fall through to format detection...

; --- Format detection ---
80d02ed8  addiu a0, s1, 0x80
80d02edc  lw    v1, 0x0(a0)           ; v1 = *(buffer + 0x80)
80d02ee0  li    v0, -1                ; v0 = 0xFFFFFFFF
80d02ee4  bne   v1, v0, LAB_80d02f40  ; ← DO NOT PATCH
80d02ee8  li    v1, 0x3e              ; '>'
80d02eec  j     LAB_80d02f08          ; byte-comparison path
```

## The Signature Verifier (`FUN_80d02638`)

Performs full RSA signature verification:
1. Looks up certificate ID in an internal table
2. Performs RSA verification against the embedded key
3. Validates checksum after decompression
4. Returns 0 for valid, non-zero for invalid

Has exactly **one call site** — `0x80d02ebc`. Byte-searching for the `jal` encoding (`0C 34 09 8E`) confirms this.

## Format Detection

After the signature check, the dispatcher identifies the firmware package type:

For `.bin` files: `*(buffer + 0x80) == 0xFFFFFFFF`
- The `bne` at `0x80d02ee4` is **not taken**
- Falls through to byte-comparison path
- `'>'` byte at offset `0x88` is detected
- Writer called correctly

For `.w` files: different magic number at `0x80`
- `bne` at `0x80d02ee4` **is taken**
- Goes to `.w` handler

**This is why `0x80d02ee4` must not be patched.** Patching it sends `.bin` files down the `.w` handler, which fails to recognise them.

## B011 vs B012 Comparison

The Tedata B011 bootloader and the Orange B012 bootloader have the same structure with a constant offset:

| Symbol | B011 address | B012 address | Difference |
|---|---|---|---|
| Upgrade dispatcher | `0x80d02e84` | `0x80d02e84` | same |
| `jal` to verifier | `0x80d02ebc` | `0x80d02ebc` | same |
| **Signature branch** | `0x80d02efc` | `0x80d02ec4` | −0x38 |
| Format branch | `0x80d02f1c` | `0x80d02ee4` | −0x38 |

**B011 patch (not tested live, derived from analysis):** `w 0x80d02efc 0x00000000`

## What Was Tried Before

**Approach 1 — RSA key replacement (~1500 RAM writes):**
Located the RSA key in RAM and overwrote it with a key whose corresponding private key was known. This worked on the B011 bootloader but became unreliable on B012, possibly due to the key being cached in a different location or the table lookup changing.

**Approach 2 — Multiple code patches:**
Patched several instructions in combination with the key replacement. Complex, fragile, hard to reproduce.

**Approach 3 — Branch patch (this project):**
Patch the single branch instruction that acts on the verifier's return value. The verifier still runs — its result is simply ignored. One write. Reproducible every time.

The branch patch works because the verifier's return value is only used in one place. Patching that one place is both necessary and sufficient.
