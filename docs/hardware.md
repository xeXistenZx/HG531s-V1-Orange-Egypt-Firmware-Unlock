# Hardware

## Router Specifications

| | |
|---|---|
| **Model** | Huawei HG531s V1 (Orange Egypt variant) |
| **Also known as** | HG531 V1 (Tedata variant — same PCB) |
| **SoC** | Realtek RTL8676S (MIPS big-endian, 450 MHz) |
| **RAM** | 32 MB (internal to SoC) |
| **Flash** | 4 MB Winbond W25Q32BV SPI NOR (SOIC-8) |
| **WiFi** | Realtek RTL8192ER (2T2R 2.4 GHz) |
| **Ethernet switch** | Realtek RTL8271B (5-port) |
| **Bootloader** | Realtek RTL867X Loader v00.00.10 |
| **Orange firmware** | B012 (HG531sV1V100R001C117B012) |
| **Tedata firmware** | B011 (HG531V1V100R001C105B011) |

## UART Header

The UART header is a 4-pin through-hole footprint on the PCB, near the SoC. It is **unpopulated** — you need to solder header pins or wires directly to the holes.

| Pin | Signal | Notes |
|---|---|---|
| 1 | VCC 3.3V | **Do not connect** |
| 2 | GND | Connect to adapter GND |
| 3 | TX | Router transmits → adapter RX |
| 4 | RX | Router receives ← adapter TX |

**Always verify the pinout with a multimeter before connecting your adapter.** Measure pin 1 to ground while the router is powered — it should read ~3.3V. Pin 3 (TX) will show activity when the router is booting.

Serial settings: **115200 baud, 8N1, no flow control.**

## Boot Console

Power on the router. The UART output begins immediately:

```
Booting
Press 'ESC' to enter BOOT console...
 4M flash ================
Ext. phy is not found.
Listening Multicast upgrade packets....4
(c)Copyright Realtek, Inc. 2009
Project RTL867X LOADER (LZMA)
Version 00.00.10 (Feb 24 2017 10:48:51)
```

Press **ESC** within the first 2–3 seconds. You reach the bootloader prompt:

```
<RTL867X>
```

Note: `Ext. phy is not found` is normal — the external PHY is not used.

## Bootloader Commands

```
help                         list commands
info                         print boot info and MAC address
reboot                       reboot the router
r [addr]                     soft reboot — WARNING: wipes all RAM patches
w [addr] [val]               write 4 bytes to RAM (big-endian hex)
d [addr] <len>               dump memory (length in decimal)
ferase [offset] <len>        erase flash region — DANGEROUS, can brick
xmodem [address]             receive file via XMODEM over serial
tftp [ip] [server] [file]    TFTP firmware upload (requires correct baud rate)
web                          start HTTP firmware upload server at 192.168.1.1
```

**Important:** Commands that take arguments (`r`, `w`, `d`, `ferase`, `tftp`) may fail silently if there is a baud rate mismatch. If `w` and `d` don't work, try adjusting your serial terminal baud rate. `web`, `help`, `info`, and `reboot` always work as they take no arguments.

## Boot Info (from `info` command)

```
MAC Address [0]: c4:86:e9:xx:xx:xx
Entry Point: 0x80000000
Load Address: 0x80000000
Application Address: 0xBD000000
Flash Size: 4M
Memory Configuration: ROW:8K COL:512 Bank:4Banks
MII Selection: 0 (0: Int. PHY  1: Ext. PHY)
```

The MAC address survives all firmware flashes — it is stored in the protected factory data region of flash.
