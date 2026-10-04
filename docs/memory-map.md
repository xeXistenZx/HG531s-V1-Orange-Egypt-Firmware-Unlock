# Memory Map

## Flash Layout (4 MB)

| Offset | Size | Content |
|---|---|---|
| `0x000000` | 192 KB | Bootloader (LZMA-compressed) |
| `0x006020` | ~168 KB | LZMA loader stream (→ 109,480 bytes decompressed) |
| `0x010000` | 20 B | Boot config (5 × 32-bit big-endian words) |
| `0x030000` | ~2.3 MB | SquashFS root filesystem |
| `0x26A000` | 64 B | Kernel header (`A0000003` magic) |
| `0x26A040` | ~1.2 MB | LZMA kernel (→ 4,600,496 bytes decompressed) |
| `0x3C0000` | 192 KB | Encrypted config (**hardware-protected**) |
| `0x3F0000` | 64 KB | Factory data (MAC address, calibration) |

## Boot Config Region (`0x010000`)

```
00010000: 00000000 0023A000 BD030000 0023A000 BD030000
```

- Kernel flash offset: `0x23A000`
- Kernel load base in RAM: `0xBD030000`

## Loader Runtime Memory

| | |
|---|---|
| **Load address** | `0x80D00000` |
| **Decompressed size** | 109,480 bytes |
| **Patch address** | `0x80D02EC4` |
| **Original value** | `0x1440000B` (bne v0, zero, +0x2C) |
| **Patched value** | `0x00000000` (nop) |

## RSA Key Location in Flash

```
0xBD00E48C  ← RSA key start (in flash address space)
```

This is the location the original key-replacement approach targeted. The branch patch makes this location irrelevant — the key can be anything, the firmware is accepted regardless.

## Firmware Package Header (`.bin` format)

Both Orange B012 and Tedata B011 `.bin` packages share the same header structure at offset `0x80`:

```
00000080: ff ff ff ff 3e 00 46 60 5d 52 37 b8 62 69 91 f4
```

- `FF FF FF FF` at offset `0x80` — identifies `.bin` format (compared against `-1`)
- `3E` (`'>'`) at offset `0x88` — byte-comparison signature

The `.w` format uses a different magic number (`0xB000A000`) and is for hardware programmer use only — it cannot be uploaded via the bootloader web interface.

## Flash Protection

The config region (`0x3C0000`–`0x3EFFFF`) is hardware-protected:

```
flash protect type:1, area:9, value:0xc, addr:0x3c0000
```

The `.bin` upgrade path writes only to the kernel and rootfs regions starting at `0xBD030000`. The config region and factory data are never touched by a firmware upgrade — this is why the MAC address and factory calibration survive cross-flashing.

## Full Flash Backup

A full 4 MB backup of the original Orange firmware:

```
Filename:  HG531sV1_Orange_original_fullflash.bin
Size:      4,194,304 bytes (exactly 4 MB)
```

Keep this file safe. It is the only guaranteed recovery path if the bootloader is ever corrupted. Write it back with a CH341A programmer and SOIC-8 clip on the W25Q32BV chip.
