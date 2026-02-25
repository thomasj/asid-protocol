# Original ASID protocol
The ASID (pronounced "acid") protocol was designed by Elektron for the SidStation, with the purpose of playing back non sample-based PlaySID files over MIDI, from a computer running a 6510 emulator application.

At each frame (for example the 50 Hz vertical blank), a set of SID register write requests is transmitted. The protocol was optimized for MIDI bandwidth limits, which shaped its design (e.g., packed register updates). 

An ASID message is encapsulated in a SysEx message. For size reasons, a one-byte identifier was used and the number chosen was 45 (the favourite number of Daniel Hansson, founder of Elektron).

Thereby the format looks like this:

| Data   | Description |
|-----------|-------------|
|**0xF0**   | SysEx start |
|**0x2D**   | 45          |
|command| ASID Command |
|payload| Payload, 0 or more bytes |
|**0xF7**   | SysEx end |

Commands are listed in subsections below.

*Note*: All MIDI data bytes are 7-bit values.

## 0x4C — Start playback

### Purpose
Signal that playback is starting. This message is not mandatory.

### Payload
Nothing


## 0x4D — Stop playback

### Purpose
Signal that playback is stopping. This message is not mandatory.

### Payload
Nothing

## 0x4E - SID register data

### Purpose
Write a set of SID registers. The message is packed, so it can be chosen to update some or all of the available registers. There is also room for an extra write for the control registers.

### Payload

| Field | Size | Description |
|-------|------|-------------|
| mask_bytes | 4 byte | Indicates which ASID register IDs that are present in the message, with one bit per ID. See table below for register mapping. The first mask byte (bits 6-0) covers registers 0-6, the second covers 7-13, and so on. 
| msb_bytes | 4 bytes | Similar to `<mask_bytes>`, these contain the 8th bit (MSB) for the connected register data |
| register_data | up to 28 bytes | Lowest 7 bits of the data for each register, as indicated by `<mask_bytes>` |

### ASID to SID register mapping
| ASID register ID | SID register ID | SID Register name |
|------------------|--------------|-----|
| 0 | 0x00 | Voice 1 Freq Lo | 
| 1 | 0x01 | Voice 1 Freq Hi |
| 2 | 0x02 | Voice 1 PW Lo |
| 3 | 0x03 | Voice 1 PW Hi |
| 4 | 0x05 | Voice 1 Env AD |
| 5 | 0x06 | Voice 1 Env SR |
| 6 | 0x07 | Voice 2 Freq Lo | 
| 7 | 0x08 | Voice 2 Freq Hi |
| 8 | 0x09 | Voice 2 PW Lo |
| 9 | 0x0A | Voice 2 PW Hi |
| 10 | 0x0C | Voice 2 Env AD |
| 11 | 0x0D | Voice 2 Env SR |
| 12 | 0x0E | Voice 3 Freq Lo | 
| 13 | 0x0F | Voice 3 Freq Hi |
| 14 | 0x10 | Voice 3 PW Lo |
| 15 | 0x11 | Voice 3 PW Hi |
| 16 | 0x13 | Voice 3 Env AD |
| 17 | 0x14 | Voice 3 Env SR |
| 18 | 0x15 | Filter Cutoff Lo|
| 19 | 0x16 | Filter Cutoff Hi|
| 20 | 0x17 | Filter Reso & Enable|
| 21 | 0x18 | Filter Mode / Volume |
| 22 | 0x04 | Voice 1 Control |
| 23 | 0x0B | Voice 2 Control |
| 24 | 0x12 | Voice 3 Control |
| 25 | 0x04 | Voice 1 Control (second write)|
| 26 | 0x0B | Voice 2 Control (second write)|
| 27 | 0x12 | Voice 3 Control (second write)|


## 0x4F - Display characters

### Purpose
Show ASCII characters on the display of the receiving unit.

### Payload
| Field | Size | Description |
|-------|------|-------------|
|character_bytes| 0 to many | Display these ASCII characters. 
*Note 1*: SysEx end byte serves as the terminator.

*Note 2*: For compatibility with SidStation, implementations may assume a 2x16 display that clears when the command is received.

# Extended ASID protocol

## Background

The commands defined in this section extend the original ASID protocol to support modern SID playback environments and more faithful reproduction of original hardware behavior.

They were initially designed during development of enhanced TherapSID firmware, including support for emulated SID chips, along with updates to players such as DeepSID and SIDFactory II. However, the extensions are intended to be generally applicable and implementation-agnostic.

These extensions enable:

- support for multiple SID chips and optional FM synthesis devices
- higher playback speeds
- a timing “recipe” that allows cycle-accurate playback using standard ASID message streams 
- description of the intended playback environment

### Notes

- ASID was originally designed for streaming over standard MIDI transport. At MIDI DIN speeds, SID playback effectively tops out at approximately 75 Hz update rates. Many of the extensions defined here require higher data rates and therefore benefit from USB-MIDI interfaces or direct hardware implementations.

- ASID remains unsuitable for digi-sample playback or highly arbitrary timing requirements.

## Multi-SID Register Data

Additional SID chip register streams. Same format as **0x4E**. This typically requires bandwidth beyond standard MIDI DIN speed. 

| Command | Function |
|--------|----------|
| **0x50** | SID2 register data |
| **0x51** | SID3 register data |
| **0x52** | SID4 register data |
| … | … |
| **0x5F** | SID17 register data |

## 0x60 — OPL-FM Register Data

### Purpose
Transfers register writes to an OPL-FM compatible chip (YM3526 or YM3812). These chips were originally available in devices such as the SFX Sound Expander and FM-YAM, and are also emulated in the ARM2SID.

### Payload

| Field | Size | Description |
|------|------|-------------|
| num_pairs | 1 byte | Number of address/data pairs (max 16) |
| msb_bytes | variable | Packed MSBs for pair data. See command **0x4E** for reference |
| pairs | `<num_pairs>` * 2 | Register/data pairs (OPL register, data) |

## 0x30 — SID Write Order & Timing

### Purpose
Defines register write order and timing delays. This allows for rearranging the order in which the individual registers in the ASID package is written to the SID chip.

### Payload

| Field | Count | Description |
|------|------|-------------|
| `<pairs>` | 28 × 2 bytes | One pair per ASID register (see table above)|

Each pair:

| Byte | Bits | Meaning |
|------|------|--------|
| `<data0>` | 0–5 | Register index (write order) |
| | 6 | Wait_cycles bit 7 |
| `<data1>` | 0–6 | Wait_cycles bits 0–6 |

### Timing
- `wait_cycles` = delay before next register write  
- Units: C64 CPU cycles (~1 µs)

### Notes
- Indices refer to ASID register IDs, not SID registers.
- Example order: `0, 1, 2, 3` ... `27` corresponds to the regular ASID write order.

## 0x31 — Speed Settings

### Purpose
Defines playback timing. This allows for the client to adapt its playback, for instance to implement a buffered output.

### Payload 

| Field | Size | Description |
|------|------|-------------|
| `<data0>` | 1 byte | System & speed settings |
| `<data1>` | 1 byte | Frame delta (LSB) |
| `<data2>` | 1 byte | Frame delta |
| `<data3>` | 1 byte | Frame delta (MSB + reserved bits) |

### `<data0>` Bit Layout

| Bits | Meaning |
|------|--------|
| bit 0 | 0 = PAL, 1 = NTSC |
| bits 1–4 | Speed multiplier (1×–16×) |
| bit 5 | Reserved (0) |
| bit 6 | Buffering requested |
| bit 7 | Reserved |

### `<data1>` Bit Layout
| Bits | Meaning |
|------|--------|
| bit 0-6 | frame delta bits 0-6 |

### `<data2>` Bit Layout
| Bits | Meaning |
|------|--------|
| bit 0-6 | frame delta bits 7-13 |

### `<data3>` Bit Layout
| Bits | Meaning |
|------|--------|
| bit 0-1 | frame delta bits 14-15 |
| bit 2-6 | reserved |


### Notes
Frame delta is a 16-bit value.

- Maximum: 65535 µs (~15 Hz)

Use the speed multiplier when:

- No advanced timing system exists
- `framedelta` is 0

## 0x32 — SID Type Information

### Purpose
Identifies SID chip type and variants.

### Payload

| Field | Size | Description |
|------|------|-------------|
| `<data0>` | 1 byte | Chip index |
| `<data1>` | 1 byte | Chip type & variant |

### `<data0>` — Chip Index

| Value | Meaning |
|------|--------|
| 0 | SID1 |
| 1 | SID2 |
| … | … |

### `<data1>` Bit Layout

| Bits | Meaning |
|------|--------|
| bit 0 | 0 = 6581, 1 = 8580 |
| bits 1–6 | Reserved for variants (R3, R4AR, etc.) |
| bit 7 | Reserved |

### Notes
- Future expansion may include FM or additional chip types.
- If bits 1–6 are zero, bit 0 alone defines the type.

