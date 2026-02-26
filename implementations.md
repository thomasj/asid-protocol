# ASID Implementations

This document lists known implementations of the ASID protocol and its extensions.

If you find any errors or know about other products supporting ASID, please submit a pull request updating this table.


## Feature Definitions

| Feature | Description |
|----------|-------------|
| 0x4C/0x4D | Start/stop playback |
| 0x4E SID write | Original SID register write |
| 0x4F Character | Character display data |
| Multi-SID | Support for commands 0x50–0x5F |
| 0x30 Timing | SID write-order and wait-cycle control |
| 0x31 Speed | Extended speed and frame delta support |
| 0x32 SID Type | SID chip type reporting |
| 0x60 OPL-FM | OPL (YM3526/YM3812) register streaming |


## Implementation Matrix

### Clients (Receivers)

Hardware or software that receives ASID messages and renders SID audio.

| Product | Type | 0x4C/0x4D Start/stop | 0x4E SID write | 0x4F Character | Multi-SID | 0x30 Timing | 0x31 Speed | 0x32 SID Type | 0x60 OPL-FM | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| [Elektron SidStation](https://www.elektron.se/legacy) | Hardware | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | Original ASID implementation |
| [Twisted Electrons TherapSID](https://www.twistedelectrons.com/therapsid) | Hardware | ❌  | ✅  | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ | Needs Turbo MIDI interface for Multi-SID and OPL-FM|
| [TeensyROM](https://github.com/SensoriumEmbedded/TeensyROM) | Hardware | ✅  | ✅  | ✅ | ✅ | ✅  | ✅  | ✅ | ❌ | |
| [USBSID-Pico](https://github.com/LouDnl/USBSID-Pico) | Hardware | ✅  | ✅  | ❌ | ✅   | ✅  | ✅  | ✅ | ✅ | |
| [Plogue chipsynth C64](https://www.plogue.com/products/chipsynth-c64.html) | Windows, macOS, Linux | ❌  | ✅  | ❌  | ❌ | ❌  | ❌  | ❌ | ❌ | |

### Hosts (Players / Senders)

Software that generates and transmits ASID messages.

| Product | Platform | 0x4C/0x4D Start/stop | 0x4E SID write | 0x4F Character | Multi-SID | 0x30 Timing | 0x31 Speed | 0x32 SID Type | 0x60 OPL-FM | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| [DeepSID](https://deepsid.chordian.net) | Browser | ❌  | ✅  | ❌ | ✅ | ❌ | ❌ | ❌ | ✅ | |
| [SIDFactory II](https://blog.chordian.net/sf2) | Windows, macOS, Linux | ❌  | ✅  | ❌ | ❌ | ✅  | ✅  | ✅ | ❌ | Currently in separate branch |
| [Elektron ASID-XP](https://www.elektron.se/support-downloads/sidstation) | Windows | ✅   | ✅  | ✅  | ❌ | ❌ | ❌ | ❌ | ❌ | No longer developed |
| [Elektron ASID-X](https://www.elektron.se/support-downloads/sidstation) | macOS | ❌   | ✅  | ❌  | ❌ | ❌ | ❌ | ❌ | ❌ | No longer developed, 32-bit Mac OS X only |
| [Station 64](https://csdb.dk/release/?id=207214) | Commodore 64 | ❓  | ✅  | ❓ | ❌ | ❌  | ❌  | ❌ | ❌ | |
| [Sidplay 5](https://github.com/Alexco500/sidplay5) | macOS | ❌  | ✅  | ❌ | ❌ | ❌  | ❌  | ❌ | ❌ | |
| [Plogue chipsynth C64](https://www.plogue.com/products/chipsynth-c64.html) | Windows, macOS, Linux | ⚠️  | ✅  | ✅  | ❌ | ❌  | ❌  | ❌ | ❌ |  |

## Legend

- ✅ Supported  
- ⚠️ Partial support  
- ❌ Not supported  
- ❓ Unknown / not yet verified  

