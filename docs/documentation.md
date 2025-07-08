---
id: quick-start
title: Quick Start Guide
category: Getting Started
tags: [quick-start, getting-started, first-rules]
seeAlso: [route-filter-midi, transform-notes-controllers, time-based-effects, automatic-control, interactive-setups, testing-basics, device-connection, logging-tab, firmware-tab]
---

# Quick Start Guide

## First Connection
1. **Power on your Midi Mind device** with USB-C cable
2. **Open Device tab** → Scan for Devices → Connect
3. **Go to Rules tab** to write your first rule

## Language Power in 30 Seconds
The rule language is designed to be **simple but incredibly expressive**. Every rule follows this pattern:
```
condition -> action;
```

**What makes it powerful:**
• **Smart pattern matching** - `noteOn(ch, note, vel)` captures any note, `noteOn(1, 60, vel)` captures only middle C on channel 1
• **Flexible conditions** - Use `&` (and), `|` (or), comparisons like `note > 60`, even math like `vel * 2`
• **Multiple outputs** - One input can trigger many actions: `action1, action2, action3`
• **Global state** - Variables that remember values across all rules
• **Precise timing** - `delay()` function for echoes, arpeggios, complex timing
• **Every MIDI message** - Notes, controllers, pitch bend, system messages, even timing clocks

You can build anything from simple channel maps to complex harmonizers in just a few lines.

## Your First Rules

### Block Unwanted MIDI
```
// Block all drum channel messages
noteOn(10, note, vel) ->;
noteOff(10, note, vel) ->;
```

### Change MIDI Channels
```
// Change everything from channel 1 to channel 2
noteOn(1, note, vel) -> noteOn(2, note, vel);
noteOff(1, note, vel) -> noteOff(2, note, vel);
controlChange(1, cc, val) -> controlChange(2, cc, val);
```

### Duplicate Messages
```
// Send notes to multiple channels at once
noteOn(ch, note, vel) -> noteOn(2, note, vel), noteOn(3, note, vel);
noteOff(ch, note, vel) -> noteOff(2, note, vel), noteOff(3, note, vel);
```

### Smart Velocity Control
```
// Boost quiet notes, limit loud ones
noteOn(ch, note, vel) & vel < 40 -> noteOn(ch, note, vel + 30);
noteOn(ch, note, vel) & vel > 100 -> noteOn(ch, note, 100);
noteOff(ch, note, vel) -> noteOff(ch, note, vel);
```

### Conditional Routing
```
// Low notes to bass channel, high notes to lead channel
noteOn(ch, note, vel) & note < 60 -> noteOn(1, note, vel);  // Bass
noteOn(ch, note, vel) & note >= 60 -> noteOn(2, note, vel); // Lead
noteOff(ch, note, vel) & note < 60 -> noteOff(1, note, vel);
noteOff(ch, note, vel) & note >= 60 -> noteOff(2, note, vel);
```

### Controller Magic
```
// Map modulation wheel to control multiple parameters
controlChange(ch, 1, val) -> 
  controlChange(2, 11, val),      // Expression on channel 2
  controlChange(3, 7, val / 2),   // Half volume on channel 3
  controlChange(4, 10, val + 20); // Pan on channel 4
```

### Transpose with Global Variables
```
global transpose = 0;

// Control transpose with pitch bend
pitchBend(ch, bend) -> transpose = (bend - 8192) / 1000;

// Apply to all notes
noteOn(ch, note, vel) -> noteOn(ch, note + transpose, vel);
noteOff(ch, note, vel) -> noteOff(ch, note + transpose, vel);
```

### Simple Echo Effect
```
// Note plays immediately, then echoes on channel 3
noteOn(ch, note, vel) ->
  noteOn(ch, note, vel),        // Original
  delay(24),                    // Wait 1 beat
  noteOn(3, note, vel / 2);     // Echo (quieter)

noteOff(ch, note, vel) -> 
  noteOff(ch, note, vel),       // Turn off original
  delay(24),                    // Same delay
  noteOff(3, note, vel);        // Turn off echo
```

### Dynamic Harmonizer
```
global harmony = 0;

// Control harmony with modulation wheel
controlChange(ch, 1, val) -> harmony = val / 32;  // 0-3 range

// Add harmony based on setting
noteOn(ch, note, vel) & harmony == 1 ->
  noteOn(ch, note, vel),
  noteOn(3, note + 4, vel);  // Add third

noteOn(ch, note, vel) & harmony == 2 ->
  noteOn(ch, note, vel),
  noteOn(3, note + 4, vel),  // Third
  noteOn(4, note + 7, vel);  // Fifth

noteOff(ch, note, vel) & harmony == 1 ->
  noteOff(ch, note, vel),    // Turn off original
  noteOff(3, note + 4, vel); // Turn off third

noteOff(ch, note, vel) & harmony == 2 ->
  noteOff(ch, note, vel),    // Turn off original
  noteOff(3, note + 4, vel), // Turn off third
  noteOff(4, note + 7, vel); // Turn off fifth

// Pass through when harmony is off
noteOn(ch, note, vel) & harmony == 0 -> noteOn(ch, note, vel);
noteOff(ch, note, vel) & harmony == 0 -> noteOff(ch, note, vel);
```

## Test Your Rules
1. **Go to Test tab**
2. **Type**: `noteOn(1, 60, 100)`
3. **Click "Run Test"**
4. **See what happens**

## Watch Your Rules in Action
1. **Go to Logging tab** and start logging
2. **Play your MIDI controller**
3. **Watch green arrows (↓)** for MIDI coming in
4. **Watch red arrows (↑)** for what your rules output
5. **Perfect for learning** how your rules actually work

## Deploy to Device
1. **Rules tab** → "Try on Device" (temporary)
2. **Or "Save to Device"** (permanent)

## Keep Your Device Updated
**Firmware tab** shows your current version and available updates:
• **Update when available** for new features and bug fixes
• **Takes 2-5 minutes** - just click Update and wait
• **Improves performance** and adds new capabilities

## Next Steps
Once you've tried these basic examples:
• **[[route-filter-midi|Route and Filter MIDI]]** - Channel routing and message filtering
• **[[transform-notes-controllers|Transform Notes and Controllers]]** - Transpose, velocity curves, chord generation
• **[[time-based-effects|Add Time-Based Effects]]** - Echo, delays, arpeggios
• **[[automatic-control|Create Automatic Control]]** - LFO, beat-synced effects, auto-pan
• **[[interactive-setups|Build Interactive Setups]]** - Mode switching, dynamic controls
• **[[testing-basics|Testing Rules]]** - How to validate your rules work
• **[[device-connection|Device Connection]]** - Connection troubleshooting

## Rule Structure Basics
Every rule follows this pattern:
```
condition -> action;
```

**Examples:**
• `noteOn(10, note, vel) ->;` - Block (empty action)
• `noteOn(ch, note, vel) -> noteOn(ch, note, vel);` - Transform
• Multiple actions: `condition -> action1, action2;`

---
id: route-filter-midi
title: Route and Filter MIDI
category: Examples
tags: [routing, filtering, channels, keyboard-split]
seeAlso: [basic-tasks, transform-notes-controllers, language-reference, testing-basics, logging-tab]
---

# Route and Filter MIDI

## Block Specific Channels
```
// Block channel 10 (drums)
noteOn(10, note, vel) ->;
noteOff(10, note, vel) ->;
controlChange(10, cc, val) ->;
```
// Could prevent drum machine from triggering other sound modules
// Could isolate drum track in multi-instrument setups

## Simple Channel Change
```
// Route channel 1 to channel 5
noteOn(1, note, vel) -> noteOn(5, note, vel);
noteOff(1, note, vel) -> noteOff(5, note, vel);
controlChange(1, cc, val) -> controlChange(5, cc, val);
```
// Could route keyboard to different sound module channel
// Could separate performance parts in multi-timbral devices

## Multiple Channel Routing
```
// Send channel 1 to channels 2, 3, and 4
noteOn(1, note, vel) -> 
  noteOn(2, note, vel),
  noteOn(3, note, vel),
  noteOn(4, note, vel);
noteOff(1, note, vel) -> 
  noteOff(2, note, vel),
  noteOff(3, note, vel),
  noteOff(4, note, vel);
```
// Could layer multiple synthesizer sounds simultaneously
// Could trigger multiple guitar amp models at once

## Keyboard Split
```
// Split at middle C (60) - low notes to bass, high notes to piano
noteOn(ch, note, vel) & note < 60 -> noteOn(1, note, vel);
noteOff(ch, note, vel) & note < 60 -> noteOff(1, note, vel);
noteOn(ch, note, vel) & note >= 60 -> noteOn(2, note, vel);
noteOff(ch, note, vel) & note >= 60 -> noteOff(2, note, vel);

// Sustain pedal to both channels
controlChange(ch, 64, val) -> 
  controlChange(1, 64, val),
  controlChange(2, 64, val);
```
// Could split keyboard between bass synthesizer and piano module
// Could assign left hand to guitar bass channel, right hand to lead channel

## Conditional Channel Routing
```
// Route by velocity - quiet notes to acoustic channel, loud notes to electric
noteOn(ch, note, vel) & vel < 64 -> noteOn(1, note, vel);
noteOff(ch, note, vel) & vel < 64 -> noteOff(1, note, vel);
noteOn(ch, note, vel) & vel >= 64 -> noteOn(2, note, vel);
noteOff(ch, note, vel) & vel >= 64 -> noteOff(2, note, vel);
```
// Could switch between clean and distorted guitar sounds based on playing dynamics
// Could trigger different drum samples based on hit strength

## Filter by Note Range
```
// Block very low and very high notes
noteOn(ch, note, vel) & note < 24 ->;
noteOff(ch, note, vel) & note < 24 ->;
noteOn(ch, note, vel) & note > 108 ->;
noteOff(ch, note, vel) & note > 108 ->;
```
// Could prevent accidental triggering of extreme piano notes
// Could limit guitar MIDI pickup to playable range

## Verify Your Rules Work
Use **[[logging-tab|Real-time Logging]]** to watch your rules in action:
• Green arrows show original MIDI input
• Red arrows show your transformed output
• Perfect for understanding exactly what your rules do

---
id: transform-notes-controllers
title: Transform Notes and Controllers
category: Examples
tags: [transform, transpose, velocity, controllers, chord-generation]
seeAlso: [route-filter-midi, time-based-effects, language-reference, testing-basics, logging-tab]
---

# Transform Notes and Controllers

## Transpose Notes
```
// Move everything up one octave
noteOn(ch, note, vel) -> noteOn(ch, note + 12, vel);
noteOff(ch, note, vel) -> noteOff(ch, note + 12, vel);
```
// Could match guitar MIDI pickup to song key
// Could transpose keyboard to match vocalist's range

## Octave Doubling
```
// Add octave above every note
noteOn(ch, note, vel) ->
  noteOn(ch, note, vel),           // Original
  noteOn(ch, note + 12, vel / 2);  // Octave (quieter)
noteOff(ch, note, vel) ->
  noteOff(ch, note, vel),          // Turn off original
  noteOff(ch, note + 12, vel);     // Turn off octave
```
// Could thicken piano sound with automatic octave doubling
// Could add shimmer effect to guitar MIDI pickups

## Simple Chord Generator
```
// Turn single notes into major triads
noteOn(ch, note, vel) ->
  noteOn(ch, note, vel),      // Root
  noteOn(ch, note + 4, vel),  // Third
  noteOn(ch, note + 7, vel);  // Fifth
noteOff(ch, note, vel) ->
  noteOff(ch, note, vel),     // Turn off root
  noteOff(ch, note + 4, vel), // Turn off third
  noteOff(ch, note + 7, vel); // Turn off fifth
```
// Could create instant harmony for solo keyboard performance
// Could generate guitar chord voicings from single-note input

## Velocity Curves
```
// Boost quiet notes, limit loud ones
noteOn(ch, note, vel) & vel < 40 -> noteOn(ch, note, vel + 30);
noteOff(ch, note, vel) & vel < 40 -> noteOff(ch, note, vel);
noteOn(ch, note, vel) & vel > 100 -> noteOn(ch, note, 100);
noteOff(ch, note, vel) & vel > 100 -> noteOff(ch, note, vel);

// Pass through normal velocities unchanged
noteOn(ch, note, vel) & vel >= 40 & vel <= 100 -> noteOn(ch, note, vel);
noteOff(ch, note, vel) & vel >= 40 & vel <= 100 -> noteOff(ch, note, vel);
```
// Could adjust touch sensitivity for different playing styles
// Could compensate for uneven velocity response in MIDI controllers

## Controller Remapping
```
// Map modulation wheel (1) to expression (11)
controlChange(ch, 1, val) -> controlChange(ch, 11, val);

// Invert pitch bend direction
pitchBend(ch, val) -> pitchBend(ch, 16383 - val);
```
// Could use mod wheel to control reverb depth on effects processor
// Could reverse pitch bend direction to match different instruments

## Multi-Parameter Controller Mapping
```
// Map one controller to multiple parameters with scaling
controlChange(ch, 1, val) ->
  controlChange(2, 7, val),           // Volume on channel 2 (direct)
  controlChange(3, 11, val / 2),      // Expression on channel 3 (half)
  controlChange(4, 10, 64 + val / 4); // Pan on channel 4 (center + quarter)
```
// Could use single knob to control multiple effect parameters simultaneously
// Could map breath controller to affect volume, filter, and vibrato together

## Verify Your Rules Work
Use **[[logging-tab|Real-time Logging]]** to watch your rules in action:
• Green arrows show original MIDI input
• Red arrows show your transformed output
• Perfect for understanding exactly what your rules do

---
id: time-based-effects
title: Add Time-Based Effects
category: Examples
tags: [time-based, echo, delay, arpeggiator, timing]
seeAlso: [transform-notes-controllers, automatic-control, language-reference, testing-basics, logging-tab]
---

# Add Time-Based Effects

OK, at this point, even though the numbers can be put directly in the rules, we are going to use global variables for some of them, because it will make it easier to describe what they do, and they will make it easier for you to customize if you copy and reuse this ruleset.

## Simple Echo Effect
```
global echo_channel = 3;    // Change to your echo output channel
global echo_delay = 24;     // Change delay time (24 = 1 beat)

// Play note immediately, then echo
noteOn(ch, note, vel) ->
  noteOn(ch, note, vel),          // Original
  delay(echo_delay),              // Wait
  noteOn(echo_channel, note, vel / 2);  // Echo (quieter)

noteOff(ch, note, vel) -> 
  noteOff(ch, note, vel),         // Turn off original immediately
  delay(echo_delay),              // Wait same amount
  noteOff(echo_channel, note, vel);    // Turn off echo
```
// Could add delay repeats to guitar MIDI pickups
// Could create echo effects on any MIDI instrument
// Could send delayed notes to separate reverb channels

## Multi-Tap Echo
```
global echo_channel = 3;    // Change to your echo output channel

// Three echoes with decreasing volume
noteOn(ch, note, vel) ->
  noteOn(ch, note, vel),          // Original
  delay(12),                      // 1/8 beat
  noteOn(echo_channel, note, vel / 2),     // First echo
  delay(12),                      // Another 1/8 beat
  noteOn(echo_channel, note, vel / 4),     // Second echo
  delay(12),                      // Another 1/8 beat
  noteOn(echo_channel, note, vel / 8);     // Third echo

noteOff(ch, note, vel) -> 
  noteOff(ch, note, vel),         // Turn off original
  delay(12),                      // Wait for first echo
  noteOff(echo_channel, note, vel),       // Turn off first echo
  delay(12),                      // Wait for second echo  
  noteOff(echo_channel, note, vel),       // Turn off second echo
  delay(12),                      // Wait for third echo
  noteOff(echo_channel, note, vel);       // Turn off third echo
```
// Could create complex delay patterns for guitar processors
// Could generate rhythmic echo effects on synthesizers

## Basic Arpeggiator
```
global arp_channel = 2;     // Change to your arpeggio output channel
global arp_speed = 12;      // Change arpeggio speed (12 = 1/8 note)

// Up arpeggio: root, third, fifth, octave
noteOn(ch, note, vel) ->
  noteOn(arp_channel, note, vel),
  delay(arp_speed),
  noteOn(arp_channel, note + 4, vel),
  delay(arp_speed),
  noteOn(arp_channel, note + 7, vel),
  delay(arp_speed),
  noteOn(arp_channel, note + 12, vel);

// Turn off all arpeggio notes when key released
noteOff(ch, note, vel) -> allNotesOff(arp_channel);
```
// Could turn held chords into automatic arpeggiated patterns
// Could create guitar fingerpicking patterns from simple chord input

## Verify Your Rules Work
Use **[[logging-tab|Real-time Logging]]** to watch your rules in action:
• Green arrows show original MIDI input
• Red arrows show your transformed output
• Perfect for understanding exactly what your rules do

---
id: testing-basics
title: Testing Your Rules
category: Testing
tags: [testing, validation, single-tests, test-sequences]
seeAlso: [quick-start, route-filter-midi, testing-advanced, logging-tab]
---

# Testing Your Rules

## Single Tests

### Quick Testing
1. **Test tab** → Single Test mode
2. **Type MIDI message**: `noteOn(1, 60, 100)`
3. **Click "Run Test"**
4. **See results**

### Common Test Messages
```
noteOn(1, 60, 100)        // Middle C at medium velocity
noteOff(1, 60, 0)         // Note off
controlChange(1, 64, 127) // Sustain pedal on
controlChange(1, 1, 64)   // Modulation wheel center
programChange(1, 5)       // Program change
```

### Understanding Results
```
Input: noteOn(1, 60, 100)
Output:
- noteOn(2, 72, 50)
```

• **Input** - What you sent
• **Output** - What your rules produced
• **No output** - Rules filtered the message

## Test Sequences

### Basic Test Sequence
```
noteOn(1, 60, 100)    // Send a note
wait(0.5)             // Wait half second
noteOff(1, 60, 0)     // Release note
wait(0.2)             // Brief pause
controlChange(1, 64, 127)  // Sustain on
```

### Test Sequence Commands
```
noteOn(channel, note, velocity)
noteOff(channel, note, velocity) 
controlChange(channel, controller, value)
wait(seconds)         // Pause for specified time
start()              // MIDI start
stop()               // MIDI stop
```

## Testing with Comments

### Adding Context to Tests
```
// Test keyboard split functionality
noteOn(1, 59, 100)    // Should go to bass channel
noteOn(1, 60, 100)    // Should go to piano channel
wait(0.5)

/* Test sustain pedal routing
   Sustain should affect both bass and piano channels */
controlChange(1, 64, 127)  // Sustain on
noteOn(1, 48, 100)         // Bass note with sustain
noteOn(1, 72, 100)         // Piano note with sustain
```

**Comments appear in test results** for easy understanding of what each part tests.

## Real-Time Testing with Logging
1. **Logging tab** → Turn on logging
2. **Play your MIDI controller**
3. **Watch input messages** (green arrows)
4. **Verify output messages** (red arrows)
5. **Compare to expectations**

This is especially helpful when:
• Rules aren't working as expected
• Learning what your controller actually sends
• Understanding complex rule interactions

## Testing Different Scenarios

### Test Edge Cases
```
// Test boundary conditions
noteOn(1, 0, 100)     // Lowest note
noteOn(1, 127, 100)   // Highest note
noteOn(1, 60, 0)      // Zero velocity
noteOn(1, 60, 127)    // Maximum velocity
```

### Test Wrong Inputs
```
// These should produce no output if filtering channel 10
noteOn(10, 36, 100)   // Should be blocked
noteOn(10, 38, 100)   // Should be blocked
noteOn(1, 60, 100)    // Should pass through (different channel)
```

### Test State Changes
```
// Test global variable changes
controlChange(1, 1, 0)     // Set transpose to minimum
noteOn(1, 60, 100)         // Test with low transpose
controlChange(1, 1, 127)   // Set transpose to maximum  
noteOn(1, 60, 100)         // Test with high transpose
```

## Common Test Patterns

### Test Filter Rules
```
// Rule: noteOn(10, note, vel) ->;
noteOn(10, 36, 100)   // Should produce no output
noteOn(1, 60, 100)    // Should pass through unchanged
```

### Test Channel Mapping
```
// Rule: noteOn(1, note, vel) -> noteOn(2, note, vel);
noteOn(1, 60, 100)    // Should output on channel 2
noteOn(3, 60, 100)    // Should pass through on channel 3
```

### Test Transformations
```
// Rule: noteOn(ch, note, vel) -> noteOn(ch, note + 12, vel);
noteOn(1, 60, 100)    // Should output noteOn(1, 72, 100)
noteOn(1, 48, 100)    // Should output noteOn(1, 60, 100)
```

## Background Testing

Long test sequences continue running even if you switch to other tabs. Return to Test tab anytime to see accumulated results.

## Troubleshooting Tests

### No Output from Test
• Check device connection
• Verify rule syntax (missing semicolon?)
• Check if conditions match your test input
• Rule might be filtering (empty action)

### Wrong Output
• Multiple rules might be matching
• Check arithmetic in expressions
• Verify variable names are correct
• Check operator precedence

### Test Won't Run
• Device must be connected
• Check test syntax
• Verify MIDI message format

---
id: automatic-control
title: Create Automatic Control
category: Examples
tags: [automatic, lfo, tremolo, auto-pan, beat-sync, program-changes]
seeAlso: [time-based-effects, interactive-setups, language-reference, testing-basics, logging-tab]
---

# Create Automatic Control

## Automatic Control Oscillator (LFO)
```
global filter_cc = 74;       // Change to your device's filter cutoff CC number
global output_channel = 2;   // Change to your target channel
global lfo_speed = 8;        // Change LFO speed (higher = faster)
global lfo_counter = 0;

// Create oscillating control values with timing clock
timingClock() -> lfo_counter = (lfo_counter + 1) % (24 * lfo_speed);

// Triangle wave LFO for filter cutoff
timingClock() ->
  controlChange(output_channel, filter_cc, lfo_counter < (12 * lfo_speed) ? 
    (lfo_counter * 127) / (12 * lfo_speed) :              // Rising
    127 - ((lfo_counter - 12 * lfo_speed) * 127) / (12 * lfo_speed)); // Falling

// Control LFO speed with modulation wheel
controlChange(ch, 1, val) -> lfo_speed = 2 + (val / 16);  // Speed 2-10

// Pass through notes to filtered channel
noteOn(ch, note, vel) -> noteOn(output_channel, note, vel);
noteOff(ch, note, vel) -> noteOff(output_channel, note, vel);
```
// Could control filter cutoff on synthesizers
// Could adjust distortion gain on guitar processors
// Could modify chorus speed on multi-effects units

## Beat-Synced Program Changes
```
global target_channel = 5;   // Change to your target device channel
global patch_count = 8;      // Change to number of patches to cycle through
global patch_beat = 0;
global current_patch = 0;

// Count beats for patch changes
timingClock() -> patch_beat = (patch_beat + 1) % 192;  // 8 beats * 24 ticks

// Change patch every 2 beats
timingClock() & patch_beat % 48 == 0 ->
  current_patch = (current_patch + 1) % patch_count,
  programChange(target_channel, current_patch);

// Route notes to auto-changing channel
noteOn(ch, note, vel) -> noteOn(target_channel, note, vel);
noteOff(ch, note, vel) -> noteOff(target_channel, note, vel);
```
// Could trigger different amp models every 2 bars
// Could switch between reverb presets rhythmically
// Could cycle through synth patches during performance

## Tremolo Effect (Volume Oscillation)
```
global tremolo_channel = 6;  // Change to your tremolo output channel
global tremolo_depth = 64;   // Change tremolo strength (0-127)
global tremolo_counter = 0;

// Fast tremolo oscillation
timingClock() -> tremolo_counter = (tremolo_counter + 1) % 48;  // 2-beat cycle

// Create tremolo volume changes
timingClock() ->
  controlChange(tremolo_channel, 7, 64 + (tremolo_depth * 
    (tremolo_counter < 24 ? tremolo_counter - 12 : 12 - tremolo_counter)) / 24);

// Control tremolo depth with breath controller
controlChange(ch, 2, val) -> tremolo_depth = val / 2;  // 0-63 range

// Send notes to tremolo channel
noteOn(ch, note, vel) -> noteOn(tremolo_channel, note, vel);
noteOff(ch, note, vel) -> noteOff(tremolo_channel, note, vel);
```
// Could add automatic volume pulsing to any MIDI device
// Could create tremolo effects on guitar amp simulators
// Could add rhythmic volume changes to synthesizer patches

## Rhythmic Auto-Pan Effect
```
global pan_channel = 3;      // Change to your panning output channel
global pan_beat = 0;

// Count quarter note beats
timingClock() -> pan_beat = (pan_beat + 1) % 96;  // 4 beats * 24 ticks

// Create rhythmic panning every 8th note
timingClock() & pan_beat % 12 == 0 ->
  controlChange(pan_channel, 10, pan_beat % 24 == 0 ? 0 : 127);  // Left/right every 8th note

// Pass through notes to channel being panned
noteOn(ch, note, vel) -> noteOn(pan_channel, note, vel);
noteOff(ch, note, vel) -> noteOff(pan_channel, note, vel);
```
// Could control stereo position in mixing boards
// Could trigger left/right speaker switching
// Could adjust balance controls on dual-amp setups

## Verify Your Rules Work
Use **[[logging-tab|Real-time Logging]]** to watch your rules in action:
• Green arrows show original MIDI input
• Red arrows show your transformed output
• Perfect for understanding exactly what your rules do

---
id: interactive-setups
title: Build Interactive Setups
category: Examples
tags: [interactive, mode-switching, variable-split, dynamic-harmonizer, sustain-effects]
seeAlso: [automatic-control, transform-notes-controllers, language-reference, testing-basics, logging-tab]
---

# Build Interactive Setups

## Mode Switching
```
global mode = 0;

// Switch modes with program change  
programChange(ch, prog) -> mode = prog % 3;

// Mode 0: Pass through
noteOn(ch, note, vel) & mode == 0 -> noteOn(ch, note, vel);
noteOff(ch, note, vel) & mode == 0 -> noteOff(ch, note, vel);

// Mode 1: Octave doubler
noteOn(ch, note, vel) & mode == 1 -> 
  noteOn(ch, note, vel),
  noteOn(3, note + 12, vel / 2);
noteOff(ch, note, vel) & mode == 1 -> 
  noteOff(ch, note, vel),
  noteOff(3, note + 12, vel);

// Mode 2: Major chord harmonizer  
noteOn(ch, note, vel) & mode == 2 ->
  noteOn(ch, note, vel),      // Original
  noteOn(2, note + 4, vel),   // Third
  noteOn(3, note + 7, vel);   // Fifth
noteOff(ch, note, vel) & mode == 2 ->
  noteOff(ch, note, vel),     // Turn off original
  noteOff(2, note + 4, vel),  // Turn off third
  noteOff(3, note + 7, vel);  // Turn off fifth
```
// Could toggle between different effect configurations with program changes
// Could switch guitar amp models with footswitch program changes
// Could change keyboard performance modes during live sets

## Variable Keyboard Split
```
global split_point = 60;     // Change default split point (middle C)

// Adjust split point with program changes
programChange(ch, prog) -> split_point = 36 + prog;

// Dynamic split based on variable
noteOn(ch, note, vel) & note < split_point -> noteOn(1, note, vel);
noteOff(ch, note, vel) & note < split_point -> noteOff(1, note, vel);
noteOn(ch, note, vel) & note >= split_point -> noteOn(2, note, vel);
noteOff(ch, note, vel) & note >= split_point -> noteOff(2, note, vel);

// Sustain pedal to both channels
controlChange(ch, 64, val) -> 
  controlChange(1, 64, val),
  controlChange(2, 64, val);
```
// Could adjust split point between bass and piano sounds during performance
// Could change guitar pickup blending point with program changes

## Dynamic Harmonizer
```
global harmony_channel = 3;  // Change to your harmony output channel
global harmony = 0;

// Control harmony with modulation wheel
controlChange(ch, 1, val) -> harmony = val / 32;  // 0-3 range

// Add harmony based on setting
noteOn(ch, note, vel) & harmony == 1 ->
  noteOn(ch, note, vel),
  noteOn(harmony_channel, note + 4, vel);  // Add third

noteOn(ch, note, vel) & harmony == 2 ->
  noteOn(ch, note, vel),
  noteOn(harmony_channel, note + 4, vel),  // Third
  noteOn(harmony_channel + 1, note + 7, vel);  // Fifth

noteOff(ch, note, vel) & harmony == 1 ->
  noteOff(ch, note, vel),
  noteOff(harmony_channel, note + 4, vel);

noteOff(ch, note, vel) & harmony == 2 ->
  noteOff(ch, note, vel),
  noteOff(harmony_channel, note + 4, vel),
  noteOff(harmony_channel + 1, note + 7, vel);

// Pass through when harmony is off
noteOn(ch, note, vel) & harmony == 0 -> noteOn(ch, note, vel);
noteOff(ch, note, vel) & harmony == 0 -> noteOff(ch, note, vel);
```
// Could use breath controller to add/remove harmony voices
// Could control harmony amount with expression pedal
// Could adjust vocal harmony intensity with mod wheel

## Sustain Pedal Effects
```
global sustain = 0;
global sustain_channel = 3;  // Change to your sustain effect channel

// Track sustain state
controlChange(ch, 64, val) -> 
  sustain = val > 63 ? 1 : 0,
  controlChange(ch, 64, val);

// Different behavior when sustain is pressed
noteOn(ch, note, vel) & sustain == 1 -> 
  noteOn(ch, note, vel),            // Original note
  noteOn(sustain_channel, note + 12, vel / 3);    // Add quiet octave when sustained

noteOff(ch, note, vel) & sustain == 1 -> 
  noteOff(ch, note, vel),           // Turn off original
  noteOff(sustain_channel, note + 12, vel);       // Turn off octave

noteOn(ch, note, vel) & sustain == 0 -> 
  noteOn(ch, note, vel);            // Just pass through when not sustained

noteOff(ch, note, vel) & sustain == 0 -> 
  noteOff(ch, note, vel);
```
// Could add shimmer effects only when sustain pedal is pressed
// Could trigger ambient textures with sustain pedal
// Could enable special guitar effects during sustained sections

## Verify Your Rules Work
Use **[[logging-tab|Real-time Logging]]** to watch your rules in action:
• Green arrows show original MIDI input
• Red arrows show your transformed output
• Perfect for understanding exactly what your rules do

---
id: language-reference
title: Language Reference
category: Reference
tags: [reference, syntax, language, rules]
seeAlso: [route-filter-midi, transform-notes-controllers, time-based-effects, testing-basics]
---

# Language Reference

## Rule Structure
```
condition -> action;
```

Every rule must end with a semicolon.

## Pattern Matching
```
noteOn(channel, note, velocity)    // Bind all variables
noteOn(1, note, velocity)          // Match specific channel
noteOn(1, 60, velocity)            // Match specific note
noteOn(1, 60, 100)                 // Match exact message
```

## Actions

### Empty Actions (Filtering)
```
condition ->;  // Matches input but produces no output = filtering
```

### Single Actions
```
condition -> noteOn(ch, note, vel);
```

### Multiple Actions
```
condition -> action1, action2, action3;
```

## Conditions

### Logical Operators
```
& // AND - both conditions must be true
| // OR - either condition can be true
```

### Comparisons
```
== != < <= > >= // Standard comparisons
```

### Conditional Expression
```
condition ? value_if_true : value_if_false
```

### Examples
```
// AND condition
noteOn(ch, note, vel) & vel > 64 -> noteOn(ch, note, vel);

// OR condition  
(noteOn(1, note, vel) | noteOn(2, note, vel)) -> noteOn(3, note, vel);

// Conditional value
noteOn(ch, note, vel) -> noteOn(ch, note, vel > 100 ? 100 : vel * 2);
```

## Arithmetic
```
+ - * / %  // Addition, subtraction, multiplication, division, modulo
() // Grouping
```

**Examples:**
```
noteOn(ch, note, vel) -> noteOn(ch, note + 12, vel / 2);  // Octave up, half volume
controlChange(ch, cc, val) -> controlChange(ch, cc + 10, val % 128);
```

## MIDI Messages

### Note Messages
```
noteOn(channel, note, velocity)
noteOff(channel, note, velocity)
```

### Controllers
```
controlChange(channel, controller, value)
programChange(channel, program)
pitchBend(channel, value)
aftertouch(channel, pressure)
```

### System Messages
```
start()
stop()
continue()
timingClock()
systemReset()
allNotesOff(channel)
allSoundOff(channel)
```

## Global Variables
```
global variableName = initialValue;

// Examples
global transpose = 0;
global mode = 1;
global counter = 0;
```

**Usage:**
```
// Read global variable
noteOn(ch, note, vel) -> noteOn(ch, note + transpose, vel);

// Modify global variable
controlChange(ch, 1, val) -> transpose = val - 64;
```

## Delay Function
```
delay(ticks)  // 24 ticks = 1 beat
```

**Examples:**
```
// Simple delay
noteOn(ch, note, vel) ->
  noteOn(ch, note, vel),
  delay(24),
  noteOn(3, note, vel);

// Multiple delays
noteOn(ch, note, vel) ->
  noteOn(ch, note, vel),     // Immediate
  delay(12),                 // 1/8 beat later
  noteOn(2, note + 4, vel),  // Third
  delay(12),                 // Another 1/8 beat
  noteOn(3, note + 7, vel);  // Fifth
```

## How Rule Matching Works

**Key principle:** When any rule matches an input message, the original message is **consumed** and only the rule outputs are sent.

**Examples:**
```
// Input: noteOn(1, 60, 100)
noteOn(ch, note, vel) -> noteOn(2, note, vel);
// Result: noteOn(2, 60, 100) (original consumed, replacement sent)

// Input: noteOn(1, 60, 100)  
noteOn(10, note, vel) -> noteOn(2, note, vel);
// Result: noteOn(1, 60, 100) (no rule matched, passes through unchanged)

// Input: noteOn(1, 60, 100)
noteOn(ch, note, vel) ->;
// Result: (nothing) (rule matched and consumed, empty action = filtered)
```

## Common Patterns

### Pass Through Unchanged
```
noteOn(ch, note, vel) -> noteOn(ch, note, vel);
noteOff(ch, note, vel) -> noteOff(ch, note, vel);
```

### Block Everything from Channel
```
noteOn(10, note, vel) ->;
noteOff(10, note, vel) ->;
controlChange(10, cc, val) ->;
```

### Copy to Multiple Channels
```
noteOn(ch, note, vel) -> 
  noteOn(2, note, vel),
  noteOn(3, note, vel),
  noteOn(4, note, vel);
noteOff(ch, note, vel) -> 
  noteOff(2, note, vel),
  noteOff(3, note, vel),
  noteOff(4, note, vel);
```

### Route by Note Range
```
noteOn(ch, note, vel) & note < 60 -> noteOn(2, note, vel);  // Low notes
noteOff(ch, note, vel) & note < 60 -> noteOff(2, note, vel);
noteOn(ch, note, vel) & note >= 60 -> noteOn(3, note, vel); // High notes
noteOff(ch, note, vel) & note >= 60 -> noteOff(3, note, vel);
```

### Transform and Keep Original
```
noteOn(ch, note, vel) ->
  noteOn(ch, note, vel),           // Keep original
  noteOn(2, note + 12, vel / 2);   // Add transformed copy
noteOff(ch, note, vel) ->
  noteOff(ch, note, vel),
  noteOff(2, note + 12, vel);
```

---
id: testing-advanced
title: Advanced Testing
category: Testing
tags: [testing, advanced, complex-scenarios, regression, load-testing]
seeAlso: [testing-basics, automatic-control, interactive-setups, troubleshooting, logging-tab]
---

# Advanced Testing

## Complex Test Scenarios

### Complete Keyboard Split Test
```
/* Comprehensive keyboard split validation
   Tests split point, sustain routing, and chord handling */

// Test exact split boundary at middle C
noteOn(1, 59, 100)    // Should route to bass (channel 1)
noteOn(1, 60, 100)    // Should route to piano (channel 2)
noteOn(1, 61, 100)    // Should route to piano (channel 2)
wait(0.5)
noteOff(1, 59, 0)
noteOff(1, 60, 0)
noteOff(1, 61, 0)
wait(0.3)

/* Test sustain pedal routing to both channels */
controlChange(1, 64, 127)  // Sustain on
noteOn(1, 48, 100)         // Bass note with sustain  
noteOn(1, 72, 100)         // Piano note with sustain
wait(0.5)
noteOff(1, 48, 0)          // Notes should sustain
noteOff(1, 72, 0)
wait(0.5)
controlChange(1, 64, 0)    // Sustain off - notes should stop
```

### Performance Simulation Test
```
/* Simulate live performance scenario
   Tests mode switching, controller changes, complex timing */

// Start with basic pass-through mode
programChange(1, 0)       // Set mode 0
noteOn(1, 60, 80)         // Should pass through unchanged
wait(0.5)
noteOff(1, 60, 0)

// Switch to harmonizer mode
programChange(1, 2)       // Set mode 2  
noteOn(1, 60, 80)         // Should produce harmony (C-E-G)
wait(1.0)
noteOff(1, 60, 0)

// Test controller interaction during performance
controlChange(1, 1, 64)   // Modulation wheel center
noteOn(1, 67, 100)        // G major chord
noteOn(1, 71, 100)
noteOn(1, 74, 100)
wait(0.5)
controlChange(1, 1, 127)  // Modulation wheel up
wait(0.5)
controlChange(1, 1, 0)    // Modulation wheel down
wait(0.5)
// Cleanup
noteOff(1, 67, 0)
noteOff(1, 71, 0)
noteOff(1, 74, 0)
```

### Timing and Delay Validation
```
/* Test precise timing of delay() function
   Validates echo effects and arpeggio timing */

// Test basic echo with 1-beat delay
noteOn(1, 60, 100)        // Should trigger immediate + delayed notes
wait(1.5)                 // Wait for delayed note
/* Expected: immediate C, then 1-beat later, echo C */

// Test arpeggio timing
noteOn(1, 60, 100)        // C major arpeggio input
wait(2.0)                 // Wait for full arpeggio sequence
/* Expected sequence:
   1. C (immediate)
   2. E (after 1/8 beat) 
   3. G (after 1/4 beat)
   4. C octave (after 3/8 beat) */
noteOff(1, 60, 0)
```

## Testing Global Variables

### State Persistence Test
```
/* Test global variable persistence across different message types */

// Initialize transpose
controlChange(1, 1, 64)   // Center position = 0 transpose
noteOn(1, 60, 100)        // Should output note 60 (no change)
wait(0.3)
noteOff(1, 60, 0)

// Change transpose setting
controlChange(1, 1, 80)   // Should set transpose to +2
noteOn(1, 60, 100)        // Should output note 62 (60 + 2)
wait(0.3)
noteOff(1, 60, 0)

/* Verify state persists across different message types */
controlChange(1, 7, 100)  // Volume change - should not affect transpose
noteOn(1, 60, 100)        // Should still output note 62
wait(0.3)
noteOff(1, 60, 0)
```

### Counter and Statistics Testing
```
/* Test rules that track statistics and counters */

// Reset counters
start()                   // Should reset global counters
wait(0.1)

// Generate series of notes with different velocities
noteOn(1, 60, 40)         // Quiet note
wait(0.2)
noteOff(1, 60, 0)
noteOn(1, 60, 80)         // Medium note
wait(0.2)
noteOff(1, 60, 0)
noteOn(1, 60, 120)        // Loud note
wait(0.2)
noteOff(1, 60, 0)

/* Rules should be tracking:
   - Total note count
   - Average velocity
   - Playing style adaptation */
```

## Multi-Device Testing

### Device Role Testing
```
/* Test device-specific behavior based on assigned roles */

// Set device to lead role
programChange(16, 1)      // Device role = 1 (lead)
noteOn(1, 60, 100)        // Should get lead processing
wait(0.5)
noteOff(1, 60, 0)

// Switch to bass role
programChange(16, 2)      // Device role = 2 (bass)
noteOn(1, 48, 100)        // Should get bass processing
wait(0.5) 
noteOff(1, 48, 0)

// Test invalid role
programChange(16, 99)     // Invalid role
noteOn(1, 60, 100)        // Should use default behavior
```

## Error Condition Testing

### Boundary Value Testing
```
/* Test extreme values and edge cases */

// Test note range boundaries
noteOn(1, 0, 100)         // Lowest possible note
noteOn(1, 127, 100)       // Highest possible note
noteOn(1, 60, 0)          // Zero velocity (should be note-off)
noteOn(1, 60, 127)        // Maximum velocity

// Test arithmetic overflow scenarios
global transpose = 0;
controlChange(1, 1, 127)  // Maximum modulation
noteOn(1, 120, 127)       // High note + high transpose
// Should handle gracefully without producing invalid MIDI
```

### Rule Conflict Testing
```
/* Test scenarios where multiple rules might conflict */

// Send messages that could match multiple rules
noteOn(1, 60, 100)        // Might match several pattern rules
controlChange(1, 64, 127) // Sustain - might affect multiple channels
noteOn(1, 59, 100)        // Right at split boundary conditions
```

## Load Testing

### High Message Rate Testing
```
/* Test system under high MIDI message load */

// Rapid note sequence
20 {
  noteOn(1, 60, 100)
  wait(0.05)              // 50ms between notes = 20 notes/second
  noteOff(1, 60, 0)
  wait(0.05)
}

// Continuous controller sweep
128 {
  controlChange(1, 1, val)  // val increments 0-127
  wait(0.01)              // 10ms between changes
}
```

## Test Organization

### Test Suites by Feature
Create comprehensive test suites for each major feature:

• **Filter Tests** - All filtering scenarios
• **Channel Mapping Tests** - All routing scenarios  
• **Transform Tests** - All transformation scenarios
• **Timing Tests** - All delay and timing scenarios
• **State Tests** - All global variable scenarios

### Regression Testing
Keep test sequences that validate fixes for previously found bugs. Run these whenever making changes to ensure problems don't reoccur.

## Test Result Analysis

### Reading Complex Results
With comments preserved in results, you can:
• **Quickly identify failing sections** - Comments show what each part tests
• **Understand context** - Know why a test was written
• **Debug efficiently** - Comments help locate specific issues
• **Document behavior** - Results serve as behavior specification

Advanced testing ensures your MIDI rules work correctly in all scenarios, from simple use cases to complex live performance situations.

---
id: firmware-tab
title: Firmware Updates
category: Device Management
tags: [firmware, updates, ota, device-management, security]
seeAlso: [device-connection, device-management, troubleshooting]
---

# Firmware Updates

## What This Does

The Firmware tab lets you update your Midi Mind device with the latest features and improvements. Think of it like updating an app on your phone - it keeps your device working its best.

## Before You Start

• **Connect your device** - Green light means you're ready
• **Keep devices close** - Stay within arm's reach during update
• **Use USB-C power** - Don't rely on battery during updates
• **Save your rules** - Backup important setups first

## What You See

### Your Current Version
Shows what version is currently on your device. If you see a "Latest" badge, you're all set!

### Available Updates
A list of versions you can install. The newest one is marked "Latest" and is usually what you want.

### Update Button
Changes based on what you're doing:
• **Update** - Installing a newer version
• **Revert** - Going back to an older version
• **Reflash** - Reinstalling the same version (if something went wrong)

## How to Update

1. **Pick a version** from the list (usually the "Latest" one)
2. **Read what's new** to see what the update does
3. **Click the Update button**
4. **Wait** - Takes about 2-5 minutes
5. **Done!** Your device restarts with the new version

## During the Update

You'll see a progress bar and time remaining. The most important thing: **don't unplug anything** once it starts uploading to your device.

The update happens in stages:
• **Getting ready** - Device prepares for update
• **Downloading** - Gets the new version from the internet  
• **Uploading** - Sends it to your device (this is the longest part)
• **Finishing** - Device installs and restarts

## If Something Goes Wrong

### Update Won't Start
• Check that your device shows green (connected)
• Make sure you have internet
• Try reconnecting your device

### Update Stops or Fails
• Move devices closer together
• Check your USB-C connection
• Restart everything and try again

### Device Won't Work After Update
• Unplug USB-C and plug back in
• Reconnect Bluetooth
• If still broken, contact support

## Why Update?

Updates bring:
• **New features** - More ways to transform your MIDI
• **Bug fixes** - Solutions to problems you might not even notice
• **Better performance** - Your device works smoother

## Stay Safe

The app handles all the technical stuff automatically. Your device can recover from most problems, and you can always go back to an older version if needed.

---
id: logging-tab
title: Real-time Logging
category: Monitoring
tags: [logging, monitoring, midi, real-time, debugging]
seeAlso: [testing-basics, route-filter-midi, troubleshooting, device-connection]
---

# Real-time Logging

## What This Does

The Logging tab shows you MIDI messages flowing through your device in real-time. It's like watching your rules work - you can see what goes in and what comes out.

## Getting Started

1. **Connect your device** (green light means you're ready)
2. **Open the Logging tab** - it starts automatically
3. **Look for "Logging Active"** - means it's working
4. **Play some MIDI** - watch messages appear!

## What You See

### Message Flow
Messages appear as they happen:
• **Green arrows (↓)** - MIDI coming INTO your device
• **Red arrows (↑)** - MIDI going OUT of your device
• **Message text** - What the MIDI message says (like "noteOn C4 velocity 100")

### Counters at Top
Shows how many messages have passed through:
• **Input count** - Messages received
• **Output count** - Messages sent

## Why This Helps

### See Your Rules Working
• **Compare input vs output** - See how your rules change things
• **Spot problems** - If something's wrong, you'll see it here
• **Understand your setup** - Watch what your keyboard/controller actually sends

### Debug Issues
• **No sound?** - Check if input messages appear
• **Wrong notes?** - See if output messages look right
• **Controller not working?** - Watch for controller messages

## Controlling What You See

### Too Many Messages?
Use the filter options:
• **"Performance" preset** - Hides timing clocks (reduces clutter for live use)
• **"Basic" preset** - Shows just notes and important controllers
• **"Full" preset** - Shows everything

### Filter Controls
• **Include/Exclude toggle** - Choose whether to show or hide selected types
• **Filter chips** - Quick buttons for common message types
• **Advanced button** - More detailed filtering options

## Useful Buttons

• **Clear** - Remove all messages from display
• **Export** - Copy messages to clipboard
• **Auto-scroll** - Automatically show newest messages (can turn off)
• **Timestamps** - Show/hide exact timing (usually not needed)

## When to Use This

### Writing Rules
1. Write a rule
2. Turn on logging
3. Test your rule
4. See if it does what you expected

### Live Performance
• Use "Performance" filter to reduce clutter
• Watch for problems during your set
• See if controllers are working

### Learning
• Play your keyboard and see what messages it sends
• Try different controllers and knobs
• Understand how MIDI works

## Common Message Types

• **noteOn** - Key press
• **noteOff** - Key release  
• **controlChange** - Knobs, sliders, pedals
• **programChange** - Patch/sound changes
• **pitchBend** - Pitch wheel

## Tips

• **Start simple** - Use Basic filter first, then try Full if you need more
• **Clear regularly** - Hit Clear button to remove old messages
• **Turn off when not needed** - Saves your device some work
• **Use with Test tab** - Send test messages and watch them in logging

The Logging tab is your window into what your device is actually doing. When something's not working right, this is usually the first place to look!

---
id: device-connection
title: Device Connection
category: Device Management
tags: [device, connection, bluetooth, troubleshooting]
seeAlso: [quick-start, troubleshooting, device-management, firmware-tab]
---

# Device Connection

## Initial Setup

### Before Connecting
1. **Power on your Midi Mind** with USB-C cable
2. **Enable Bluetooth** on your Android device
3. **Grant app permissions** (Bluetooth, location as required by Android)
4. **Put Midi Mind in pairing mode** (check device manual for button sequence)

### Connection Steps
1. **Device tab** → "Scan for Devices"
2. **Wait 10-30 seconds** for your device to appear
3. **Tap "Connect"** next to your Midi Mind
4. **Wait for green status** indicator

## Connection Status

### Status Indicators
• **Green dot** - Connected and ready
• **Red dot** - Not connected  
• **Signal bars** - Connection quality (green = good, red = poor)
• **Device name** - Shows at top when connected

### Custom Device Names
1. **Type descriptive name** like "Studio Rig" or "Live Setup"
2. **Tap "Set Name"** to apply
3. **Use "Reset"** to return to original name

**Why custom names help:**
• Distinguish multiple devices
• Match names to usage context
• Organize setups clearly

## Testing Connection

### Quick Test
1. **Test tab** → Single Test mode
2. **Type**: `noteOn(1, 60, 100)`
3. **Click "Run Test"**
4. **Look for results** indicating MIDI processing

### MIDI Hardware Test
1. **Connect MIDI keyboard** to Midi Mind input (MIDI TRS A cable)
2. **Connect Midi Mind output** to sound module/DAW (MIDI TRS A cable)
3. **Deploy simple rule** like channel mapping
4. **Play keyboard** - verify MIDI processing works

## Multiple Devices

### Switching Between Devices
1. **Disconnect** from current device
2. **Scan again** to see available devices
3. **Connect** to desired device
4. **Deploy appropriate rules** for that setup

### Managing Several Devices
• **Custom names** for each device purpose
• **Organize rule sets** by device or context
• **Quick switching** between setups

## Connection Quality

### Maintaining Good Connection
• **Keep devices within 10 feet** during use
• **Use stable USB-C power supply** - unstable power affects Bluetooth
• **Minimize interference** from WiFi routers, other Bluetooth devices
• **Monitor signal strength** - green bars indicate good quality

### Improving Poor Connections
• **Move devices closer** together
• **Check power supply** quality and connections
• **Remove interference** sources
• **Restart Bluetooth** on Android device if needed

## Troubleshooting

### Device Doesn't Appear
• **Check pairing mode** - ensure Midi Mind is ready
• **Move closer** - devices need proximity for initial pairing
• **Restart scan** - tap "Scan for Devices" again
• **Power cycle** - disconnect and reconnect USB-C power
• **Check Android Bluetooth** - ensure it's enabled

### Connection Fails
• **Retry connection** - tap "Connect" again
• **Restart Bluetooth** - toggle Android Bluetooth off/on
• **Check power** - ensure stable USB-C power supply
• **Clear interference** - move away from WiFi routers
• **Restart app** if connection repeatedly fails

### Connection Drops During Use
• **Check distance** - move devices closer
• **Verify power supply** - unstable USB-C power causes drops
• **Check interference** - other Bluetooth devices nearby
• **Monitor signal strength** - red bars indicate problems
• **Reconnect manually** if auto-reconnect fails

### Can't Deploy Rules
• **Verify connection** - green status indicator required
• **Check rule syntax** - syntax errors prevent deployment
• **Try simpler rule** - test with basic rule first
• **Restart connection** if deployment repeatedly fails

## Quick Deployment

### Deploy Saved Rules
1. **Device tab** → "Deploy Rule Set"
2. **Browse or search** saved rule sets
3. **Tap rule set** to deploy immediately
4. **Rules become active** on connected device

### Rule Storage Options
• **"Try on Device"** - Temporary (lost on power cycle)
• **"Save to Device"** - Permanent storage
• **"Load from Device"** - Retrieve current device rules

Good connection management ensures reliable MIDI processing during practice, recording, and live performance.

---
id: device-management
title: Device Management
category: Device Management
tags: [device, management, multiple-devices, organization]
seeAlso: [device-connection, file-organization, troubleshooting, firmware-tab]
---

# Device Management

## Working with Multiple Devices

### Device Organization
• **Custom names** - "Studio Rig", "Live Setup", "Practice Room"
• **Purpose-based naming** - Match names to usage context
• **Location-based naming** - "Home Studio", "Rehearsal Space"

### Switching Between Devices
1. **Disconnect** from current device if needed
2. **Scan for devices** to see all available Midi Mind units
3. **Connect** to desired device
4. **Deploy appropriate rules** for that context

### Rule Organization by Device
```
Live Performance/
  ├── Stage Piano Setup (Device: "Live Rig")
  └── Keyboard Split (Device: "Live Rig")
Studio/
  ├── Recording Enhancement (Device: "Studio Mind")
  └── Mix Processing (Device: "Studio Mind")
Practice/
  ├── Scale Helper (Device: "Practice Unit")
  └── Timing Exercises (Device: "Practice Unit")
```

## Device-Specific Workflows

### Live Performance Setup
• **Connect to "Live Rig"** device
• **Deploy performance rules** - keyboard splits, layers
• **Test thoroughly** before performance
• **Keep backup rules** saved to device

### Studio Recording Setup
• **Connect to "Studio Mind"** device  
• **Deploy recording rules** - velocity curves, channel routing
• **Save session-specific rules** to device
• **Document rules** for future sessions

### Practice and Development
• **Use "Practice Unit"** for experimentation
• **Try experimental rules** without affecting performance setups
• **Develop new rules** safely
• **Test complex scenarios** without risk

## Device Storage Management

### Rule Storage Options

#### Temporary Deployment
**"Try on Device"**
• Rules active immediately
• Lost when device powers off
• Good for testing and development
• No permanent storage used

#### Permanent Storage
**"Save to Device"**
• Rules stored in device memory
• Survive power cycling
• Device uses as default rules
• Limited storage space

#### Rule Retrieval
**"Load from Device"**
• Get currently active rules from device
• Load into Rules tab for editing
• Backup device rules to app
• Verify what's actually running

### Managing Device Memory
• **Device storage is limited** - save only essential rules
• **Use app storage** for rule development and variations
• **Keep device rules simple** and well-tested
• **Document what's stored** on each device

## Device Health and Updates

### Firmware Management
• **Firmware tab** → Check current version regularly  
• **Update when available** → New features and bug fixes
• **Before important use** → Ensure device is current
• **After connection problems** → Updates may resolve issues

### Regular Maintenance
• **Check for updates** monthly or before important sessions
• **Update device firmware** to maintain compatibility
• **Keep device software current** for best performance

## Quick Deployment

### Direct Deployment from Device Tab
1. **Device tab** → "Deploy Rule Set"
2. **Search or browse** saved rules
3. **Filter by folder** if organized
4. **Tap to deploy** immediately

### Benefits of Quick Deployment
• **No tab switching** required
• **Fast setup changes** during sessions
• **Easy A/B testing** of different rule sets
• **Quick recovery** from rule problems

## Connection Monitoring

### Real-Time Status
• **Connection indicator** - green/red status
• **Signal strength** - bars showing quality
• **Device name display** - shows connected device
• **Connection duration** - how long connected

### Automatic Recovery
• **Connection monitoring** - detects drops
• **Auto-reconnect attempts** - tries to restore connection
• **Status notifications** - alerts about connection issues
• **Manual reconnect** - button to force reconnection

## Device Identification

### Hardware Identification
• **Bluetooth device name** - factory assigned
• **Custom app names** - your assigned names
• **Connection history** - previously connected devices
• **Visual identification** - match names to physical devices

### Avoiding Mix-ups
• **Use descriptive names** - clear purpose identification
• **Physical labels** - label actual devices
• **Test rules** before important use
• **Document device assignments** for complex setups

## Backup and Recovery

### Rule Backup Strategy
• **Save working rules** to app storage
• **Export important setups** for external backup
• **Document device configurations**
• **Keep copies** of device-stored rules in app

### Recovery Procedures
• **Load from device** - retrieve rules from device
• **Deploy from app** - restore from app storage
• **Reset device** if rules corrupted
• **Reconnect and redeploy** for connection issues

## Best Practices

### Organization
• **Consistent naming** across devices and rules
• **Folder structure** matches device usage
• **Regular backups** of important configurations
• **Documentation** of device assignments

### Performance
• **Test before use** - always validate rules work
• **Keep backups** of working configurations
• **Simple device rules** - complex rules in app storage
• **Monitor connections** during critical use

### Maintenance
• **Regular connection testing** - verify device health
• **Update custom names** as usage changes
• **Clean up old rules** from device storage
• **Document changes** for future reference

Effective device management ensures reliable MIDI processing across different contexts and setups.

---
id: file-organization
title: File Organization
category: Organization
tags: [organization, files, folders, naming, backup]
seeAlso: [device-management, testing-basics, route-filter-midi]
---

# File Organization

## Folder Structure

### By Usage Context
```
Live Performance/
  ├── Jazz Gigs/
  │   ├── Piano Trio Split
  │   └── Solo Piano Setup
  ├── Rock Shows/
  │   ├── Keyboard Layers
  │   └── Bass + Keys Split
  └── Open Mic/
      └── Simple Piano Setup

Studio Work/
  ├── Recording/
  │   ├── Velocity Curves
  │   └── Channel Routing
  └── Mixing/
      ├── MIDI Effects
      └── Controller Mapping

Practice/
  ├── Technique/
  │   ├── Scale Practice
  │   └── Timing Exercises
  └── Learning/
      ├── Song Arrangements
      └── Chord Practice
```

### By Instrument/Setup
```
Keyboards/
  ├── 88-Key Weighted/
  │   ├── Piano Split
  │   └── Organ Layers
  └── 61-Key Synth/
      ├── Lead Patches
      └── Bass Sounds

Controllers/
  ├── Modulation Mapping
  ├── Expression Setup
  └── Pedal Routing

Multi-Device/
  ├── Dual Keyboard Setup
  └── Keyboard + Drums
```

## File Naming

### Descriptive Names
**Good examples:**
• "Stage Piano Split at Middle C"
• "Studio Velocity Curve - Gentle"
• "Jazz Trio - Bass Channel Routing"
• "Practice - Scale Transposer"

**Avoid:**
• "Rules1", "Test", "New Setup"
• Generic names without context
• Numbers without description

### Naming Conventions
• **Include purpose** - what the rules accomplish
• **Add context** - when/where used
• **Version if needed** - "Setup v2" for iterations
• **Keep length reasonable** - descriptive but not excessive

## Save Dialog Best Practices

### Required Information
**Name:** Clear, descriptive identification
**Description:** What the rules do and when to use them
**Folder:** Logical grouping for easy finding
**Tags:** Keywords for search and filtering

### Example Save Dialog
```
Name: "Live Keyboard Split"
Description: "Split at middle C, bass to channel 1, piano to channel 2, sustain to both"
Folder: "Live Performance/Jazz Gigs"
Tags: "keyboard, split, live, jazz"
```

## Test File Organization

### Linking Tests to Rules
• **Associate during save** - link test to specific rule set
• **Visual indicators** - see which tests go with which rules
• **Project coherence** - keep related files together
• **Easy validation** - find tests for specific rule sets

### Test File Structure
```
Live Performance/
  Rules: Stage Piano Split
  Tests: Split Point Test, Sustain Test, Performance Scenario
  
  Rules: Keyboard Layers  
  Tests: Layer Balance Test, Controller Response Test

Studio Work/
  Rules: Recording Enhancement
  Tests: Velocity Curve Test, Level Test, Processing Test
```

## Search and Discovery

### Search Features
• **Text search** - find by name, description, or content
• **Folder filtering** - browse specific categories
• **Tag filtering** - find by keywords
• **Rule associations** - find tests for specific rule sets

### Effective Tagging
**Use consistent tags:**
• **Context:** live, studio, practice
• **Function:** split, layer, filter, transpose
• **Instrument:** piano, keyboard, bass, drums
• **Style:** jazz, rock, classical

**Example tag combinations:**
• "live, keyboard, split, jazz"
• "studio, velocity, recording, piano"
• "practice, transpose, scales"

## Maintenance

### Regular Cleanup
• **Remove old versions** - delete outdated rule sets
• **Update descriptions** - keep information current
• **Consolidate similar** - merge duplicate or similar setups
• **Clear test history** - remove old single tests periodically

### Version Management
• **Save working versions** - keep copies of rules that work well
• **Version naming** - "Jazz Setup v2", "Studio Mix Final"
• **Archive old versions** - move to "Archive" folder instead of deleting
• **Document changes** - note what changed between versions

## Backup Strategy

### Local Backup
• **Export important setups** - save copies outside app
• **Regular exports** - weekly backup of critical rules
• **Document exports** - note what was backed up when

### Organization for Backup
• **Critical folder** - rules you can't afford to lose
• **Working folder** - current development rules
• **Archive folder** - old but potentially useful rules

## Sharing and Collaboration

### Export for Sharing
• **Include descriptions** - help others understand purpose
• **Add metadata** - tags and folder information
• **Document requirements** - what MIDI setup is needed
• **Test thoroughly** - ensure rules work before sharing

### Import Organization
• **Review before importing** - understand what rules do
• **Organize appropriately** - put in correct folders
• **Test immediately** - verify rules work in your setup
• **Rename if needed** - adjust names for your naming convention

## Quick Access

### Favorites/Recent
• **Deploy Rule Set** from Device tab shows recent and organized rules
• **Search functionality** finds rules quickly
• **Folder chips** filter by category
• **Star important rules** for quick access

### Workflow Optimization
• **Name by usage frequency** - put commonly used terms first
• **Group related rules** in same folders
• **Use consistent structure** across all folders
• **Keep descriptions current** for accurate search results

Good file organization saves time during setup, reduces confusion during performance, and makes rule development more efficient.

---
id: troubleshooting
title: Troubleshooting
category: Help
tags: [troubleshooting, problems, debugging, help]
seeAlso: [device-connection, testing-basics, route-filter-midi, language-reference, firmware-tab, logging-tab]
---

# Troubleshooting

## First Steps for Any Problem

### Check Real-Time MIDI Flow
• **Logging tab** → Start logging
• **Play your controller** → See if input messages appear
• **No input?** Check MIDI connections
• **Wrong input?** Controller might be sending different data than expected
• **No output?** Rules might be filtering or have syntax errors

### Update Device Firmware
• **Firmware tab** → Check for updates
• **Updates may fix** connection and processing issues
• **Update if available** → Usually resolves many problems

## Rule Problems

### Rule Not Working
**Check device connection:**
• Device tab - is device connected (green status)?
• Signal strength - are bars green?
• Try reconnecting if status is red

**Test rule logic:**
• Test tab - does single test show expected output?
• Check rule syntax - missing semicolon?
• Verify conditions match your input messages

**Common syntax errors:**
```
// Wrong - missing semicolon
noteOn(ch, note, vel) -> noteOn(ch, note, vel)

// Right - has semicolon
noteOn(ch, note, vel) -> noteOn(ch, note, vel);

// Wrong - mismatched parentheses
noteOn(ch, note, vel) & (note < 60 -> noteOn(ch, note, vel);

// Right - balanced parentheses  
noteOn(ch, note, vel) & (note < 60) -> noteOn(ch, note, vel);
```

### No Output from Rule
**Rule matched but empty action:**
```
noteOn(10, note, vel) ->;  // This filters (blocks) the message
```

**Condition doesn't match:**
```
// Rule expects channel 1, but you're sending channel 2
noteOn(1, note, vel) -> noteOn(ch, note, vel);  // Won't match channel 2 input
```

**Variable name errors:**
```
// Wrong - misspelled variable
noteOn(ch, note, vel) -> noteOn(ch, not, vel);

// Right - consistent variable names
noteOn(ch, note, vel) -> noteOn(ch, note, vel);
```

### Getting Wrong Output
**Multiple rules matching:**
```
// Both rules can match the same input
noteOn(ch, note, vel) -> noteOn(ch, note, vel);      // Rule 1
noteOn(ch, note, vel) -> noteOn(3, note + 12, vel); // Rule 2
// Result: Both outputs are generated
```

**Arithmetic errors:**
```
// Wrong - might produce invalid note numbers
noteOn(ch, note, vel) -> noteOn(ch, note + 20, vel);  // Note 120 + 20 = 140 (invalid)

// Right - limit output range
noteOn(ch, note, vel) -> noteOn(ch, note + 20 > 127 ? 127 : note + 20, vel);
```

**Operator precedence:**
```
// Wrong - multiplication happens first
noteOn(ch, note, vel) -> noteOn(ch, note, vel + 10 * 2);  // vel + 20

// Right - parentheses control order
noteOn(ch, note, vel) -> noteOn(ch, note, (vel + 10) * 2);  // (vel + 10) * 2
```

## Device Issues

### Can't Connect to Device
**Device preparation:**
• Power on Midi Mind with stable USB-C supply
• Put device in pairing mode (check manual)
• Keep devices within 3 feet for initial pairing

**Android Bluetooth:**
• Enable Bluetooth in Android settings
• Grant app permissions (Bluetooth, location)
• Try toggling Bluetooth off/on

**App connection:**
• Device tab → "Scan for Devices"
• Wait 10-30 seconds for device to appear
• Tap "Connect" next to your device
• Check for green status indicator

### Connection Drops
**Distance and interference:**
• Keep devices within 10 feet during use
• Move away from WiFi routers, other Bluetooth devices
• Check signal strength bars - green is good, red is poor

**Power supply:**
• Use stable USB-C power supply
• Poor power causes Bluetooth instability
• Try different USB-C cable or power source if connection drops frequently

**Bluetooth recovery:**
• Monitor connection status in Device tab
• Use "Reconnect" if auto-recovery fails
• Restart app if connection problems persist

### Can't Deploy Rules
**Connection required:**
• Green status indicator must be showing
• Try "Scan for Devices" → "Connect" if red

**Rule syntax:**
• Check Rules tab for syntax errors (red underlines)
• Fix syntax errors before deploying
• Test with simple rule first: `noteOn(ch, note, vel) -> noteOn(ch, note, vel);`

**Device storage:**
• Device memory is limited
• Try "Load from Device" to see current storage
• Clear old rules if storage full

## MIDI Hardware Issues

### No MIDI Processing
**MIDI connections:**
• MIDI source → Midi Mind input (MIDI TRS A cable)
• Midi Mind output → MIDI destination (MIDI TRS A cable)
• Check cable connections are secure

**MIDI routing:**
• Set MIDI source to send on channels your rules expect
• Set MIDI destination to receive on channels your rules output
• Test with simple channel mapping rule first

**Rule deployment:**
• Rules must be deployed to device ("Try on Device" or "Save to Device")
• Check Device tab for deployment status
• Try simple rule to verify basic processing works

### Wrong MIDI Output
**Channel mismatch:**
• Check which channels your rules output to
• Verify MIDI destination is listening on correct channels
• Use Test tab to see exactly what your rules produce

**MIDI message types:**
• Some devices only respond to specific MIDI messages
• Check if destination needs note-on/note-off, controllers, etc.
• Use Logging tab to verify message flow

## Testing Issues

### Tests Won't Run
**Device connection:**
• Test tab requires connected device
• Check Device tab for green connection status
• Connect device before running tests

**Test syntax:**
• Single tests: exact MIDI message format required
• Example: `noteOn(1, 60, 100)` not `note on 1 60 100`
• Test sequences: proper command format required

### Test Results Confusing
**Understanding results:**
• **Input:** what you sent to test
• **Output:** what your rules produced
• **No output:** rules filtered the message (empty action)

**Use comments in test sequences:**
```
// This comment explains what this test does
noteOn(1, 60, 100)    // This comment appears in results
```

## Firmware Issues

### Update Won't Start
• **Check connection** - Device must show green (connected)
• **Check internet** - Make sure you have internet access
• **Restart app** - Close and reopen if needed

### Update Stops or Fails
• **Keep devices close** - Move them closer together
• **Check USB-C** - Make sure power cable is secure
• **Try again** - Most problems fix themselves on retry

### Device Won't Work After Update
• **Unplug and replug** - Disconnect USB-C, wait 5 seconds, reconnect
• **Reconnect Bluetooth** - Go to Device tab and reconnect
• **Contact support** - If device still doesn't work

## Logging Issues

### No Messages Showing
• **Check connection** - Device must show green (connected)
• **Click Start** - Make sure logging is turned on
• **Try "All Messages"** - Remove any filters temporarily

### Too Many Messages
• **Use "Performance" filter** - Reduces clutter
• **Click "Clear"** - Remove old messages
• **Turn off when not needed** - Use Stop button

## Performance Issues

### Processing Delays
**Complex rules:**
• Simplify rules if processing seems slow
• Avoid unnecessary calculations in rules
• Use simpler conditions when possible

**Device overload:**
• Too many simultaneous MIDI messages can overload device
• Reduce MIDI message rate if problems occur
• Use filtering to reduce unnecessary processing

**Logging impact:**
• Disable logging when not needed
• Use selective filtering to reduce overhead
• Turn off timestamps for better performance

### Global Variable Problems
**Initialization:**
```
// Wrong - using uninitialized variable
noteOn(ch, note, vel) -> noteOn(ch, note + transpose, vel);

// Right - initialize first
global transpose = 0;
noteOn(ch, note, vel) -> noteOn(ch, note + transpose, vel);
```

**Value ranges:**
```
// Wrong - can produce invalid values
global transpose = 0;
controlChange(ch, 1, val) -> transpose = val;  // val can be 0-127

// Right - scale to reasonable range
global transpose = 0;
controlChange(ch, 1, val) -> transpose = (val - 64) / 8;  // -8 to +8
```

## Getting Help

### Systematic Troubleshooting
1. **Isolate the problem** - test individual components
2. **Start simple** - use basic rules to verify connection
3. **Check each step** - connection, rules, deployment, MIDI routing
4. **Use Test tab** - verify rule behavior before live use
5. **Use Logging tab** - monitor actual MIDI message flow

### When to Restart
• **App restart** - for persistent connection issues
• **Device power cycle** - for device behavior problems
• **Android Bluetooth restart** - for Bluetooth connectivity issues

### Documentation
• **Save working rules** - keep copies of setups that work
• **Document problems** - note what didn't work and why
• **Test thoroughly** - validate rules before important use
• **Export logs** - save relevant log data for analysis

Most problems are solved by checking connections, verifying rule syntax, and testing with simple examples before moving to complex scenarios.

---
id: app-tabs
title: App Tabs Overview
category: App Structure
tags: [app, tabs, workflow, overview]
seeAlso: [quick-start, device-connection, route-filter-midi, testing-basics, firmware-tab, logging-tab]
---

# App Tabs Overview

## Tab Functions

### Device Tab
**Purpose:** Connect to Midi Mind devices and deploy rules

**Key functions:**
• Scan for and connect to Midi Mind devices
• Monitor connection status and signal strength
• Set custom device names for organization
• Quick deployment of saved rule sets
• Switch between multiple devices

**When to use:** Start here to establish device connection before writing rules

### Rules Tab
**Purpose:** Write and edit MIDI transformation rules

**Key functions:**
• Multi-tab editor for different rule sets
• Syntax highlighting for MIDI rule language
• Save and organize rule files in folders
• Deploy rules to connected devices (temporary or permanent)
• Load existing rules from device or storage

**When to use:** After connecting device, write your MIDI transformation logic here

### Test Tab
**Purpose:** Validate rules work correctly with connected device

**Key functions:**
• Single tests - quick validation with individual MIDI messages
• Test sequences - comprehensive testing with timing and comments
• Real-time execution on connected hardware
• Comment preservation in test results for better readability
• Background execution - tests continue while using other tabs

**When to use:** After writing rules, validate they work as expected before live use

### Firmware Tab
**Purpose:** Keep your device current and healthy

**Key functions:**
• See what version your device has
• Download and install updates
• Keep your device current with bug fixes and improvements

**When to use:** Initial setup, troubleshooting, monthly maintenance checks

### Logging Tab
**Purpose:** Watch MIDI messages in real-time - your window into what's actually happening

**Key functions:**
• See what MIDI is going into your device (green arrows)
• See what MIDI is coming out (red arrows)
• Filter messages to reduce clutter
• Spot problems with your rules or setup

**When to use:** Debug rules, learn MIDI, verify connections, troubleshoot any issues

### Help Tab
**Purpose:** Access documentation and examples

**Key functions:**
• Quick start guide and tutorials
• Complete language reference
• Rule examples organized by topic (Route/Filter, Transform, Time-Based Effects, etc.)
• Troubleshooting guides
• Best practices for organization

**When to use:** When learning the system, looking up specific syntax, or exploring topic-based examples

### Settings Tab
**Purpose:** Customize app behavior and appearance

**Key functions:**
• Theme selection (light/dark/auto)
• Editor preferences (font size, syntax highlighting)
• Bluetooth and MIDI settings
• File management options
• Performance and resource settings

**When to use:** Set up app preferences to match your workflow

## Typical Workflow

### First Time Setup
1. **Settings tab** - Configure app preferences
2. **Device tab** - Connect to your Midi Mind
3. **Help tab** - Read Quick Start Guide and explore topic-based examples
4. **Rules tab** - Write your first simple rule
5. **Test tab** - Validate the rule works
6. **Firmware tab** - Check for updates

### Regular Usage
1. **Device tab** - Connect and verify connection
2. **Rules tab** - Write or load rules
3. **Test tab** - Validate rules work correctly
4. **Logging tab** - Monitor live MIDI flow (optional)
5. **Device tab** - Deploy rules for live use

### Development Workflow
1. **Rules tab** - Write/modify rules
2. **Test tab** - Test changes
3. **Logging tab** - Monitor actual MIDI processing
4. **Rules tab** - Refine based on test results
5. **Test tab** - Final validation
6. **Device tab** - Deploy when satisfied

### Maintenance Workflow
1. **Firmware tab** - Check for device updates
2. **Device tab** - Manage device connections
3. **Logging tab** - Monitor system performance
4. **Settings tab** - Adjust app configuration as needed

## Tab Integration

### Device Connection Context
• **All tabs** show device connection status
• **Rules, Test, Firmware, and Logging tabs** require device connection
• **Device tab** provides quick rule deployment
• **Connection status** visible throughout app

### File Management
• **Rules tab** - Create and edit rule files
• **Test tab** - Create and associate test files
• **Device tab** - Quick access to saved rule files
• **Help tab** - File organization guidance

### Real-Time Feedback
• **Device tab** - Connection quality monitoring
• **Rules tab** - Syntax error detection
• **Test tab** - Real-time rule execution results
• **Firmware tab** - Update progress and device version
• **Logging tab** - Live MIDI message monitoring
• **All tabs** - Device status indicators

## Advanced Features

### Cross-Tab Operations
• **Deploy from Device tab** - Quick deployment without tab switching
• **Background testing** - Tests continue while working in other tabs
• **Live logging** - Monitor MIDI flow while developing rules
• **Firmware updates** - Can continue working while firmware downloads

### State Preservation
• **Rules persist** across tab switches
• **Test sequences continue** in background
• **Logging maintains history** when tab not active
• **Device connection monitored** continuously

### Workflow Optimization
• **Recent files accessible** from Device tab
• **Quick actions available** in all relevant tabs
• **Status indicators provide** immediate feedback
• **Contextual help available** throughout interface

The tab structure provides a logical workflow from connection through development to validation and monitoring, with each tab serving a specific purpose in the MIDI rule development and device management process.
