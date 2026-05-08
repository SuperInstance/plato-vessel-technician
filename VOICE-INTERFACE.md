# Voice Interface — Helm POV

## Setup

A USB microphone (cardioid pattern, noise-canceling) and a small marine speaker connected to the Deckboss core. Positioned at the helm where the captain stands.

No wake words needed — Deckboss is always listening but only responds when the captain's intent is clear. A "say again" confirmation loop prevents misinterpretation.

## Navigation Commands

| You say | Deckboss does | Feedback |
|---------|--------------|----------|
| "Port [N]°" | Turns rudder port by N degrees | "Port 15° — moving. ETA 2 seconds." |
| "Starboard [N]°" | Turns rudder starboard | "Starboard 10° — moving." |
| "Hard port" | Full port lock | "Hard port — rudder at stop." |
| "Hard starboard" | Full starboard lock | "Hard starboard — rudder at stop." |
| "Center rudder" | Returns rudder to zero | "Rudder centered." |
| "Steady as she goes" | Locks current heading | "Steady as she goes. Heading 045° locked." |
| "Hold course [N]°" | Autopilot to specific heading | "Holding 045°. Current deviation: 1° port." |
| "Come left/right to [N]°" | Change heading to specific | "Coming left to 020°." |
| "Jog left/right [N]" | Micro-adjustments | "Jogging right 2°. ✅" |

## Propulsion Commands

| You say | Deckboss does | Feedback |
|---------|--------------|----------|
| "Throttle [RPM]" | Sets engine RPM | "Throttle at 1800 RPM." |
| "Ahead [%]" | Sets throttle % forward | "Ahead 50%." |
| "Idle" | Drops to idle | "Throttle at idle." |
| "All stop" | Throttle idle + rudder center | "All stop. Throttle idle, rudder center." |
| "Emergency stop" | Same + bilge pumps on | "EMERGENCY STOP. Bilge pumps activated." |
| "Increase RPM by [N]" | Bump throttle up | "Throttle increased by 200 RPM. At 1800 RPM." |
| "Decrease RPM by [N]" | Bump throttle down | "Throttle decreased by 100 RPM. At 1700 RPM." |

## Information Queries

| You say | Deckboss does |
|---------|--------------|
| "Status report" | Full system overview: all ESP32s, alerts, uptime |
| "What's our speed?" | Speed through water (if paddle wheel) or SOG (if GPS) |
| "What's our fuel burn?" | GPH at current RPM, range in NM |
| "What's the engine temp?" | Coolant and exhaust temperature |
| "How's the battery?" | Voltage, current, state of charge, remaining time |
| "Any alerts?" | Lists active alerts sorted by severity |
| "Show me the steering" | Rudder angle, servo PWM, manual override status |
| "What changed since..." | Tile diff from a past PLATO room |

## Diagnostic Commands

| You say | Deckboss does |
|---------|--------------|
| "Diagnose [system]" | Runs diagnostic on specific system |
| "Run test [system]" | Exercises the system end-to-end |
| "Calibrate [system]" | Runs calibration sequence |
| "What's the error on [sensor]?" | Current error between command and feedback |
| "Show me the last 10 minutes of [system]" | PLATO tile history for that domain |
| "Compare today and yesterday" | Runs diff on tiles between time windows |
| "Why did the rudder drift?" | Pulls diagnostics: last calibration, error history, temperature |

## Configuration Commands

| You say | Deckboss does |
|---------|--------------|
| "Too slow" | Increase servo speed by tuning PWM or voltage |
| "Too fast" | Decrease servo speed or dampen response |
| "Not responsive enough" | Increase PID gains, add deadband compensation |
| "Too twitchy" | Decrease PID gains, add filtering |
| "Make it smoother" | Add acceleration ramps to servo movement |
| "Save this config" | Save current parameters to NVS on each ESP32 |
| "Restore factory" | Reset all parameters to defaults |
| "Backup" | Dump all ESP32 configs to Deckboss storage |
| "Restore from [date]" | Restore a specific backup |

## Voice Feedback Conventions

Deckboss always acknowledges commands. Three levels of feedback:

### Level 1: Simple Confirmation
For routine commands that match expectations:
```
"Port 15°"
→ "Port 15°."
→ (1 second later) "At 15° port."
```

### Level 2: Confirmation + Detail
For commands that have a measurable outcome:
```
"What's our fuel burn?"
→ "6.2 gallons per hour at 1800 RPM."
→ "Range at current burn: 280 NM."
```

### Level 3: Confirmation + Alert
For commands that trigger a diagnostic finding:
```
"Test steering"
→ "Rudder: cmd=12°, actual=10°. Drift 2°."
→ "Probable cause: low hydraulic fluid."
→ "Checking reservoir..." → "Reservoir is 0.5L low."
```

## Error Handling

| Scenario | Deckboss says | Fallback |
|----------|--------------|----------|
| Unrecognized command | "Say again?" | No action taken |
| Ambiguous command | "Did you mean port or starboard?" | No action until clarified |
| Command while system is in override | "Cannot command — manual override active. Disengage to enable." | No override override |
| Dangerous command | "Cannot — that exceeds safe limits. Maximum: 35° rudder at current speed." | Action blocked |
| Multiple commands at once | "Processing one at a time. Steering first..." | Queues the rest |
| ESP32 offline | "ESP32-RUDDER is offline. Fail-safe engaged. Try again in 30 seconds." | Manual override |
| Network noise | "Poor audio quality on that command. Say again?" | None |

## Voice Profiles

Deckboss can recognize multiple speakers but has one primary profile:

```
Captain: Casey Digennaro
  ─ Recognized voice. Full command set.
  ─ Commands processed without confirmation for routine actions.
  ─ Anomalous commands trigger "Verify: port 20°?"

Crew/guest: Unrecognized voice
  ─ Read-only queries only: "What's our speed?"
  ─ No control commands: "Port 20°" → "Not authorized for control."
  ─ Configurable: "Add crew: name" to authorize

Override: Physical MANUAL button
  ─ Always works regardless of voice recognition state
  ─ Captain doesn't need to say anything — press the button
```

## Example Dialogue

```
Captain: "Deckboss, status"
Deckboss: "All systems nominal. 5 ESP32s online."
          "Steering: 2° port drift (compensating)"
          "Throttle: 1800 RPM"
          "Engine: 82°C, oil 48psi"
          "House bank: 12.4V, 85% SOC"
          "No alerts."

Captain: "Port 20°"
Deckboss: "Port 20° — moving."
          (2 seconds)
          "At 20° port. Rudder response: 1.8s."

Captain: "That's good. Save this config."
Deckboss: "Steering parameters saved to ESP32-RUDDER. ✅"
```
