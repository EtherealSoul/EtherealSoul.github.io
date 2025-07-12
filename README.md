# Midi Mind Manager
**Professional MIDI Processing & Transformation**

![Midi Mind Device](images/midi-mind-device.jpg)
*The Midi Mind hardware processor with MIDI TRS-A I/O and USB-C power*

## Overview

The **Midi Mind** is a compact, professional MIDI processor designed for live performance, studio recording, and creative experimentation. Featuring MIDI TRS-A input and output with USB-C power, it transforms MIDI messages in real-time using a powerful yet intuitive programming language.

**Key Features:**
- **Hardware-based processing** with ultra-low latency
- **Real-time MIDI transformation** for live performance
- **Intuitive programming language** for complex routing and effects
- **Professional connectivity** via MIDI TRS-A standard
- **USB-C powered** for reliable operation
- **Bluetooth configuration** via Midi Mind Manager app

## How It Works

![MIDI Signal Flow](images/midi-signal-flow.png)
*MIDI flows from your controller through Midi Mind's programmable processor to your sound modules*

### Hardware
The Midi Mind device sits between your MIDI controller and sound modules, processing every MIDI message according to your programmed rules:

**Input:** MIDI TRS-A from keyboards, controllers, DAWs, or other MIDI sources  
**Processing:** Real-time transformation using custom rule language  
**Output:** MIDI TRS-A to synthesizers, DAWs, effects processors, or other MIDI devices  
**Power:** USB-C for stable, reliable operation  
**Configuration:** Wireless via Bluetooth using the Midi Mind Manager app  

### Programming Language
Create powerful MIDI transformations with an expressive rule-based language:

```
// Route piano split - bass and treble to different channels
noteOn(ch, note, vel) & note < 60 -> noteOn(1, note, vel);     // Bass channel
noteOn(ch, note, vel) & note >= 60 -> noteOn(2, note, vel);    // Piano channel

// Add automatic harmony
noteOn(ch, note, vel) -> 
  noteOn(ch, note, vel),        // Original note
  noteOn(3, note + 4, vel),     // Add major third
  noteOn(4, note + 7, vel);     // Add perfect fifth
```

## Use Cases

### Live Performance
- **Keyboard splits** for bass/piano combinations
- **Dynamic layering** controlled by velocity or controllers
- **Effect routing** to different outputs
- **Real-time harmonization** and chord generation
- **Failsafe MIDI routing** between multiple devices

### Studio Recording
- **Velocity curves** for consistent dynamics
- **Channel mapping** for multi-timbral setups
- **MIDI effects processing** before reaching your DAW
- **Controller remapping** for different instruments
- **Automatic chord and melody generation**

### Creative/Experimental
- **Complex arpeggiation** with custom patterns
- **Algorithmic composition** using global variables and logic
- **Time-based effects** like echoes and delays
- **Interactive control systems** that respond to playing style
- **Live-coded MIDI manipulation** during performance

## Midi Mind Manager App

![App Interface](images/app-interface.png)
*The Midi Mind Manager provides a complete development environment for MIDI rules*

The **Midi Mind Manager** iOS and Android app is your control center for one or more Midi Mind devices:

### Rule Development
- **Syntax-highlighted editor** with intelligent error detection
- **Real-time testing** on connected hardware
- **Comprehensive examples** for every use case
- **File organization** with folders and tagging

### Device Management
- **Multi-device support** with custom naming
- **Wireless rule deployment** via Bluetooth
- **Firmware updates** for new features and improvements
- **Connection monitoring** with signal strength indication

### Testing & Monitoring
- **Live MIDI logging** to see exactly what's happening
- **Rule validation** with automated testing sequences
- **Performance monitoring** for complex rule sets
- **Troubleshooting tools** for connection and rule issues

## Getting Started

**[→ Quick Start Guide](MidiMindHelp.html#quick-start)** - Connect your device and write your first rules in 5 minutes

1. **Hardware Setup:** Connect MIDI input/output via TRS-A cables, power via USB-C
2. **Download App:** Get Midi Mind Manager from App Store or Google Play  
3. **Connect & Configure:** Pair via Bluetooth and follow the Quick Start Guide
4. **Deploy Rules:** Write transformations and deploy to your device

## Documentation & Support

### 📖 Complete Documentation
**[→ Interactive Help System](MidiMindHelp.html)**

Comprehensive documentation with search, examples, and step-by-step guides.

### 🚀 Quick Access
- **[Getting Started Guide](MidiMindHelp.html#quick-start)** - Your first rules in 5 minutes
- **[Route & Filter MIDI](MidiMindHelp.html#route-filter-midi)** - Channel routing and filtering examples  
- **[Transform Notes & Controllers](MidiMindHelp.html#transform-notes-controllers)** - Transpose, velocity, chord generation
- **[Language Reference](MidiMindHelp.html#language-reference)** - Complete syntax documentation
- **[Testing Your Rules](MidiMindHelp.html#testing-basics)** - How to validate your rules work
- **[Troubleshooting](MidiMindHelp.html#troubleshooting)** - Connection and rule problems

### 📧 Technical Support
For technical support and questions: **jared.smythe.innovation@gmail.com**

---

*Midi Mind transforms MIDI in real-time with professional reliability and creative flexibility. From simple channel routing to complex algorithmic processing, unlock new possibilities in your MIDI workflow.*
