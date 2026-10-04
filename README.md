# HG531s V1 Orange Egypt — Firmware Unlock

> **Rescuing a router from e-waste** — cross-flash the **Huawei HG531s V1 (Orange Egypt variant)** to any compatible firmware, unlocking **Ethernet WAN / FTTH capability** via a single 4-byte RAM patch through the bootloader serial console.

---

## Background & Purpose

The Huawei HG531s V1 is one of the most common routers in Egypt, distributed by Orange Egypt (formerly Mobinil) to millions of subscribers over many years. With Egypt's infrastructure rapidly transitioning from ADSL/VDSL to FTTH (Fiber to the Home), these routers are becoming useless overnight — locked to a single provider and incapable of connecting to fiber.

The result: routers are being thrown away or sold second-hand for 3–5 USD. People don't even consider keeping them. They are heading straight to e-waste.

**This project changes that.**

### The Economic Reality

Egypt is a developing country where a large portion of the population lives under significant financial pressure. A router costing 3–5 USD second-hand represents real money to many families. Forcing people to discard working hardware and buy new equipment is a genuine burden — not a minor inconvenience.

This burden is made worse by ISP practices that prioritise revenue over people. A prominent example: ISPs like WE/Tedata, when installing FTTH, force subscribers to purchase a combined ONT/WiFi router unit costing **2,000–3,000 EGP** (~40–60 USD at current rates) — instead of installing a free standalone ONT and letting subscribers use their existing routers. There is no technical reason for this. A standalone ONT connected to any router works perfectly. The combined unit exists to generate a mandatory purchase.

This project is a direct response to that practice. A router people already own, that was deemed worthless, can be patched and connected to FTTH — no new hardware purchase required.

### The Technical Reality

The hardware inside the HG531s V1 is identical to the Tedata HG531 V1 — same SoC, same WiFi chip, same switch chip, same PCB layout. The Tedata firmware supports Ethernet WAN and FTTH. The only thing preventing the Orange variant from running it is a firmware signature check in the bootloader — one branch instruction that rejects any firmware not signed with Orange's key.

Patch that one instruction. Flash the Tedata firmware. The router works on FTTH.

**Confirmed test results after unlocking:**
- FTTH connection: ✅ working
- Download speed: **93–94 Mbps** (router handles well above Egypt's 30 Mbps ISP cap)
- Ping/latency: **5 ms**
- WiFi: ✅ working normally
- All LAN ports: ✅ working normally

A router that was headed for the bin is now a fully capable FTTH gateway.

### This Is Part Of A Larger Project

- **Part 1 (this repository):** Unlock the Orange firmware restriction → flash Tedata firmware → connect to FTTH
- **Part 2 (coming soon):** Decrypt and modify the Tedata config file to remove ISP provider lock → use the router with any ISP worldwide, not just Tedata

---

## ⚠️ Disclaimer

**Read this before doing anything.**

This procedure modifies your router's firmware. It works on the hardware described and has been tested successfully multiple times. However:

- **This may brick your router.** If the flash write fails or is interrupted, the router may not boot.
- **Recovery requires a hardware programmer.** A bricked router can be recovered using a CH341A programmer with SOIC-8 clip to rewrite the flash chip directly, but this requires opening the router and soldering skills.
- **Do this at your own risk.** No responsibility is accepted for damaged hardware.
- **Take a full flash backup first.** This is not optional. See `firmware/README.md`.
- **This is for educational and interoperability purposes.** You are modifying hardware you own.

---

## Compatible Firmware

This patch allows flashing **any compatible `.bin` firmware** for the HG531 V1 platform — not just Tedata. The Tedata B011 firmware is documented here because it is the tested target and provides FTTH capability, but other compatible firmware may also work.

**Tested:**
- Tedata HG531 V1 B011 (`HG531V1V100R001C105B011_defaultcast_main.bin`) ✅

**Should work (same platform, not yet tested):**
- Other Tedata HG531 V1 versions
- Other ISP variants of HG531 V1 with the same hardware

### Copyright Notice On Firmware Files

The `.bin` firmware files are **not included in this repository**. They are copyrighted by Huawei Technologies and their respective ISP partners. Distributing them would be a copyright violation.

Do not upload firmware files to this repository or link to direct downloads of copyrighted files. Instead, source them yourself from:
- Your own router backup (taken with a CH341A programmer)
- ISP firmware archives you have legitimate access to
- Community sources where the files are shared under fair use / interoperability principles

The patch and documentation in this repository are original work. The firmware files are not.

---

## Status

- ✅ Confirmed working on real hardware (HG531s V1, Orange Egypt variant, B012 bootloader)
- ✅ Orange B012 → Tedata B011 cross-flash tested end-to-end, multiple times
- ✅ FTTH connection confirmed working after flash
- ✅ 93–94 Mbps throughput confirmed on FTTH
- ✅ One-instruction patch — simpler than any previous approach
- ⚠️ Patch is **RAM-only** — must be reapplied each flash session (~10 seconds)
- ⚠️ Requires **UART serial access** — no Ethernet-only path exists (see below)
- ⚠️ Requires **soldering** to attach wires to unpopulated UART header pads

---

## Hardware

| | |
|---|---|
| **Model** | Huawei HG531s V1 (Orange Egypt variant) |
| **Also compatible** | Huawei HG531 V1 (Tedata variant — identical PCB) |
| **SoC** | Realtek RTL8676S (MIPS big-endian) |
| **RAM** | 32 MB |
| **Flash** | 4 MB Winbond W25Q32BV SPI NOR |
| **WiFi** | Realtek RTL8192ER (2T2R 2.4 GHz) |
| **Ethernet switch** | Realtek RTL8271B (5-port) |
| **Bootloader** | Realtek RTL867X Loader v00.00.10 |
| **Orange firmware** | B012 (HG531sV1V100R001C117B012) |
| **Tedata firmware** | B011 (HG531V1V100R001C105B011) |

---

## What You Need

- USB-to-UART adapter (CH340, CP2102, FTDI, or similar)
- Soldering iron — UART pads are unpopulated holes on the PCB, wires must be soldered
- Serial terminal software — **Tera Term**, minicom, picocom, PuTTY, or any other
- Ethernet cable from your PC to any LAN port on the router
- The target `.bin` firmware file
- PC set to static IP `192.168.1.2 / 255.255.255.0`
- (Recommended) CH341A programmer with SOIC-8 clip for a full flash backup first

---

## UART Connection

Locate the 4-pin through-hole footprint on the PCB near the SoC (usually silkscreened `J1` or `CN1`). The pads are **unpopulated** — solder header pins or wires directly.

| Pin | Signal | Connection |
|---|---|---|
| 1 | VCC 3.3V | **Do not connect** |
| 2 | GND | → adapter GND |
| 3 | TX (router → PC) | → adapter RX |
| 4 | RX (PC → router) | → adapter TX |

**Verify with a multimeter before connecting.** With the router powered, pin 1 should read ~3.3V, pin 2 should read 0V (ground).

Serial settings: **115200 baud, 8N1, no flow control.**

Tested terminal software: **Tera Term** (Windows), minicom / picocom (Linux), PuTTY (Windows/Linux).

---

## Step-by-Step

### Before You Start

Take a full flash backup with your CH341A programmer. Write it to a safe location. If anything goes wrong, this file is how you recover.

### Step 1 — Prepare your PC

Set your PC ethernet adapter to a static IP:
- IP: `192.168.1.2`
- Subnet: `255.255.255.0`
- Gateway: (leave blank)

Connect ethernet cable from PC to any LAN port on the router.

Open your serial terminal (Tera Term or similar) connected to the router's UART adapter.

### Step 2 — Enter the bootloader

Power-cycle the router. Watch the serial console for:

```
Booting
Press 'ESC' to enter BOOT console...
```

Press **ESC** immediately (within 2–3 seconds). You should see:

```
<RTL867X>
```

### Step 3 — Apply the patch

At the `<RTL867X>` prompt, type exactly:

```
w 0x80d02ec4 0x00000000
```

The prompt returns to a new line. No confirmation message — this is normal.

### Step 4 — Verify the patch

```
d 0x80d02ec4 8
```

Expected output:
```
0x80D02EC4: 00 00 00 00 ...
```

If you see `14 40 00 0B` instead, the patch did not apply. Try Step 3 again.

### Step 5 — Start the bootloader web server

```
web
```

The console shows:
```
The local IP is 192.168.1.1
Listening......
Waiting for uploading......
```

You have approximately 60 seconds before it times out.

### Step 6 — Upload firmware

Open a browser on your PC and go to:

```
http://192.168.1.1/upload.html
```

Select your `.bin` firmware file and upload it. Watch the serial console.

A successful flash looks like this:

```
Check DigitalSigned Ok
== write config successfully ==
It is .bin upgrade file!
write sys start:bd030000, data offset:40, len:3492882
.....................................................##############
RESTART ...
```

The dots are write progress. The `#` symbols are the final erase phase. This takes **2–3 minutes**. Do not power off the router during this time.

The router reboots automatically when done.

### Step 7 — First login

Wait ~60 seconds for the router to finish booting. Browse to `http://192.168.1.1`.

**Do a factory reset first** to clear old Orange configuration:
- Maintenance → Factory Reset, or hold the physical reset button

Default credentials for Tedata firmware:
- Username: `admin`
- Password: `admin`

---

## How The Patch Works

The bootloader calls an RSA signature verifier, then branches on the result:

```asm
80d02ebc  jal   FUN_80d02638     ; run RSA verify — returns 0=ok, non-zero=fail
80d02ec4  bne   v0, zero, reject ; if result != 0, reject the firmware  ← PATCH
```

Replacing `bne` with `nop`:

```asm
80d02ec4  nop                    ; ignore result — always continue to accept
```

| | Hex | Effect |
|---|---|---|
| Original | `14 40 00 0B` | Reject firmware if signature invalid |
| Patched | `00 00 00 00` | Ignore result — always accept |

**Why not replace the RSA key?** An earlier approach did exactly that — writing the replacement key to ~1500 RAM addresses. It worked but was fragile, complex, and broke when a newer bootloader appeared. The branch patch is simpler: it makes the key irrelevant. Whatever the verifier returns, the result is ignored.

**Why RAM-only?** The bootloader decompresses from flash into RAM at every boot. The patch modifies RAM only. Flash is unchanged. A power-cycle restores everything — you cannot permanently damage the bootloader this way.

---

## Reverting To Orange Firmware

Same procedure — apply the patch, start `web`, upload the Orange `.bin` file.

---

## Why No Ethernet-Only Path Exists

The signature check runs in the bootloader, before Linux starts. Linux cannot reach bootloader memory. All three firmware delivery paths (bootloader web page, regular web UI upgrade, multicast) pass through the same bootloader check.

Making the patch survive reboots would require modifying the compressed bootloader in flash — risky and unnecessary when the RAM patch takes 10 seconds. **UART is required. This is an architectural constraint, not a missing technique.**

---

## Safety Rules

1. **Take a full flash backup before doing anything**
2. **Never type `r` or `reboot` after applying the patch** — wipes it from RAM
3. **Never patch `0x80d02ee4`** — breaks `.bin` format detection, flash will fail
4. **Never use `ferase`** — raw flash erase, can brick the device
5. **Do not power off during flash write** — wait for `RESTART ...` message

---

## Troubleshooting

**`Check DigitalSigned error` instead of `Ok`**
Patch not applied. Verify: `d 0x80d02ec4 8` — should show `00 00 00 00`. Reapply: `w 0x80d02ec4 0x00000000`.

**`Check DigitalSigned Ok` but nothing after**
You may have accidentally patched `0x80d02ee4`. Revert it: `w 0x80d02ee4 0x14620016` then retry.

**Flash hangs for more than 5 minutes**
Something went wrong. Power-cycle — the original firmware is still in flash. Reapply the patch and retry.

**Browser can't reach `192.168.1.1` during `web` mode**
The web server times out after ~60 seconds. Ensure your PC static IP is set before typing `web`. Try each LAN port — not all ports are active during bootloader mode.

**Router won't boot after flash**
Attempt recovery via serial — watch boot messages. If completely unresponsive, use CH341A programmer to write your full flash backup to the W25Q32BV chip.

---

## Recovery

**Soft:** Power-cycle. RAM reloads from flash. Original bootloader restored. No harm done.

**Hard (router completely unresponsive):**
1. Open the router
2. Locate the W25Q32BV flash chip (SOIC-8 package near the SoC)
3. Connect CH341A programmer with SOIC-8 clip
4. Write your 4 MB full flash backup to the chip
5. Reassemble and power on

---

## The Story

This project went through three complete cycles before arriving at the solution here.

**Round 1:** Starting from the original Orange firmware, a combination of RSA key replacement in RAM plus other instruction patches eventually got Tedata firmware accepted. The bootloader also changed during this process in a way that isn't fully understood. The exact sequence wasn't documented — the excitement of first success got in the way.

**Round 2:** A different Tedata firmware was installed that didn't have the FTTH/Ethernet WAN menu. Attempts to go back to the earlier working version failed.

**Round 3:** A newer Orange firmware was flashed, bringing a newer B012 bootloader. The RSA key replacement approach stopped working. After debugging nearly from scratch — with much more awareness of the loader internals — a simpler insight emerged:

> *Why replace the entire RSA key? Just patch the branch instruction that checks whether verification passed. Make it always take the "accepted" path. The key becomes irrelevant.*

One 4-byte write. Tested multiple times. Works reliably.

---

## What's Next

**Part 2 — ISP Lock Removal (coming soon):**
The Tedata firmware restricts PPPoE to Tedata credentials only. The next repository documents how to decrypt the router's config file, edit it to remove this restriction, and repack it — making the router usable with any ISP worldwide.

---

## Repository Structure

```
├── README.md                    This file
├── LICENSE
├── PUBLISH.md                   GitHub setup instructions
├── docs/
│   ├── hardware.md              UART pinout, PCB details, boot console
│   ├── memory-map.md            Flash layout, offsets, header formats
│   ├── loader-analysis.md       Ghidra analysis of the B012 bootloader
│   └── patch-explained.md       Deep dive: why this patch, what was tried before
├── scripts/
│   └── flash_router.sh          Interactive step-by-step guide script
└── firmware/
    └── README.md                Firmware files info, verification, copyright notice
```

---

## Credits

Reverse engineering, insight, and testing by the project author, 2026.
AI assistance (Claude, DeepSeek) used for analysis and documentation.

The key insight — patch the validity check rather than replace the key — came from stepping back and asking what the minimum effective change actually was.

---

## License

MIT — see `LICENSE`.

*Not affiliated with Huawei Technologies, Orange Egypt, or Tedata.*
*For educational and interoperability use. Flash at your own risk.*
