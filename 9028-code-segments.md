# Segment-by-segment decode of a Daikin IR command

Sample command: `cool.auto.17` from `9028.json`.

```text
JgCWAAABOAABOgABOQABOpJRCwwLHwsfCwwMHwwMCwwLDAwMCx8LDAwMCh8LDAsMDAwLHwsMCwwLDAsfCx8LDAsMCx8LDAsMDAwLHwsMCwwLDAsMDAwLDAoMCx8MDAsMDAwLDAsMCwwLDAwfDAwLDAsMDB8LHwwfCwwLHwsMCwwMDAsMCwwLHwsMCwwMHwsfCx8LAAKMkwANBQ==
```

Decoded state: Cool mode, 17 C, Auto fan, remote clock 11:31, power toggle off.

The Base64 text is not divided into semantic fields. It first decodes to Broadlink pulse durations, which then decode to the Daikin64 eight-byte state. Base64 groups can cross state-byte boundaries, so the character ranges below overlap; Broadlink byte offsets are the exact boundaries.

Base64 character positions are 1-based. Broadlink byte offsets are 0-based and include the four-byte Broadlink header.

| State byte | Meaning | Broadlink pulse byte offsets | Base64 character span | Base64 excerpt | Decoded value |
|---|---|---:|---:|---|---|
| Byte 0 | Fixed identifier | 18-33 | 25-48 | `CwwLHwsfCwwMHwwMCwwLDAwM` | 0x16 |
| Byte 1 | Mode and fan | 34-49 | 45-68 | `DAwMCx8LDAwMCh8LDAsMDAwL` | 0x12: low nibble 2 = Cool; high nibble 1 = Auto fan |
| Byte 2 | Remote clock minutes | 50-65 | 65-88 | `DAwLHwsMCwwLDAsfCx8LDAsM` | 0x31 BCD = 31 minutes |
| Byte 3 | Remote clock hours | 66-81 | 89-112 | `Cx8LDAsMDAwLHwsMCwwLDAsM` | 0x11 BCD = 11 hours |
| Byte 4 | On timer | 82-97 | 109-132 | `DAsMDAwLDAoMCx8MDAsMDAwL` | 0x10: disabled; stored time 10:00 |
| Byte 5 | Off timer | 98-113 | 129-152 | `DAwLDAsMCwwLDAwfDAwLDAsM` | 0x10: disabled; stored time 10:00 |
| Byte 6 | Target temperature | 114-129 | 153-176 | `DB8LHwwfCwwLHwsMCwwMDAsM` | 0x17 BCD = 17 C |
| Byte 7 | Swing, sleep, toggle, checksum | 130-145 | 173-196 | `DAsMCwwLHwsMCwwMHwsfCx8L` | 0xE4: swing=0, sleep=0, toggle=0, reserved bit=1, checksum=0xE |

## Key fields

- Byte 1 is `0x12`: low nibble `2` is Cool mode; high nibble `1` is Auto fan.
- Byte 2 is `0x31`; Byte 3 is `0x11`. Together they encode 11:31 in BCD.
- Byte 6 is `0x17`, encoding 17 C in BCD. It maps to Broadlink pulse bytes 114-129 and Base64 characters 153-176.
- Byte 7 is `0xE4`: bit 0 vertical swing, bit 1 sleep, bit 2 reserved, bit 3 power toggle, high nibble checksum.

## How the data bits map to pulses

Each state byte has 8 pulse pairs (two Broadlink bytes per bit), least-significant bit first. In this capture, short spaces around 10-13 represent 0 and long spaces around 30-31 represent 1. The first byte of each pair is the mark; the second is the space.

The transmitted bit order for `0x17` is `1 1 1 0 1 0 0 0`; the corresponding spaces are:

`31 31 31 12 31 12 12 12`

## Broadlink envelope

- Bytes 0-3: Broadlink header `26 00 96 00`.
- Bytes 4-17: leader and protocol header pulses.
- Bytes 18-145: 64 data bits represented by 8 state bytes.
- Bytes 146-153: trailing pulses, gap, and Broadlink trailer.

To change a setting, regenerate the full IR waveform and checksum; replacing a Base64 substring is unsafe.

References: [Daikin64 protocol fields](https://github.com/crankyoldgit/IRremoteESP8266/blob/master/src/ir_Daikin.h) and [decoder/checksum implementation](https://github.com/crankyoldgit/IRremoteESP8266/blob/master/src/ir_Daikin.cpp).

Decoded for documentation only; no IR command was transmitted for hardware testing.

