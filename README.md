# ⚓ PLATO Vessel Technician — Deckboss

**Voice-first marine/industrial agent. Walk on a boat, plug in, talk, done.**

Deckboss is the PLATO vessel variant for technicians who work on boats, industrial equipment, and remote installations. It turns any ESP32-equipped system into a voice-responsive, fail-safe, self-diagnosing smart component.

```
┌─────────────────────────────────────────────┐
│         PLATO VESSEL TECHNICIAN             │
│                DECKBOSS                     │
│                                             │
│  Walk on boat →                             │
│  Plug in ESP32s →                           │
│  Tell Deckboss: "Describe the boat" →       │
│  Deckboss discovers all devices →           │
│  Creates a vessel room →                    │
│  Captain talks: "Port 20°" →                │
│  Throttle adjusts →                         │
│  Captain: "Response too slow" →             │
│  Deckboss tunes PID →                       │
│  All while boat is running                  │
└─────────────────────────────────────────────┘
```

## Why Deckboss Exists

Marine electronics are a nightmare of:
- Proprietary protocols (NMEA 2000, Raymarine SeaTalk, Simrad)
- Expensive proprietary hardware ($500+ for a basic display)
- Brittle wiring that corrodes in salt air
- Zero diagnostic self-awareness ("it just stopped working")

Deckboss solves this with:
- **Standard hardware:** ESP32 at $5 — if it fails, swap it. No vendor lock-in.
- **Voice-first interface:** Hands-free when the boat is rocking
- **Mechanical override on everything:** A servo fails? The cable still works.
- **Real-time diagnostics:** "Rudder angle reads 12° but GPS shows 8° heading change" → agent detects drift
- **Real-time code improvement:** "That response is too slow" → agent tunes PID parameters live

## Key Features

### 🔊 Voice-First Interface
Captain talks to Deckboss via a microphone on the bridge. No buttons, no menus, no touchscreen:
```
Captain: "Port 20°"
Deckboss: "Port 20° — rudder moving. ETA 3 seconds."
Captain: "Too slow"
Deckboss: "Increasing rudder servo speed. Reducing PID integral term."
Captain: "What's our fuel burn?"
Deckboss: "6.2 gallons per hour at 1800 RPM. Range: 280 NM."
```

### 🔧 Fail-Safe Design
Every automated system has a mechanical override:
- Servo disconnects → manual cable reverts
- Electronic throttle fails → mechanical linkage engages
- Power failure → systems fail to safe (rudder centers, throttle idles)
- Software failure → watchdog reset restores last known good config

### 📊 Diagnostics
Continuous monitoring detects drift, wear, and anomalies:
```
⚠️ DECKBOSS DIAGNOSTIC
  Rudder: cmd=12°, actual=10°, error=2° (threshold: ±1°)
  → Probable cause: hydraulic fluid viscosity change (cold water)
  → Recommended: Check hydraulic fluid temperature
  → Auto-correction: Increased servo PWM to compensate
  
🔧 RUDDER CALIBRATION REPORT (2026-05-08)
  Mechanical slop: 1.2° at center, 2.8° at full deflection
  → Recommend: Grease rudder shaft
  → Auto-compensation applied
```

### 🗺️ Mixed Fleet Discovery
Multiple ESP32s + existing boat electronics. Deckboss discovers all rooms and creates a **vessel room** that aggregates them:
```
room: "cocapn-vessel"
├── propulsion/
│   ├── throttle_controller/       (ESP32 room)
│   └── engine_monitor/            (ESP32 room)
├── steering/
│   ├── rudder_controller/         (ESP32 room)
│   └── autopilot/                 (ESP32 room)
├── navigation/
│   ├── gps_receiver/              (NMEA 0183 bridge → PLATO)
│   └── depth_sounder/             (NMEA 2000 bridge → PLATO)
├── electrical/
│   ├── battery_monitor/           (ESP32 room)
│   └── bilge_pump_controller/    (ESP32 room)
├── diagnostics/
│   ├── system_health/             (aggregated sensor fusion)
│   └── maintenance_log/          (agent-generated)
```

### 📄 Printable Diagrams
Deckboss generates PDF wiring diagrams specific to THIS boat:
```bash
# Voice command:
"Print the steering system diagram"
→ Generates: cocapn-vessel-steering-wiring.pdf
# Includes:
# - ESP32 pin connections for rudder servo
# - Relay wiring for autopilot bypass
# - Sensor positions with photos
# - Parts list for the steering system
```

### 🔧 Parts Sourcing
"This solenoid is 24V but we have 12V" → Deckboss recommends:
```
⚠️ VOLTAGE MISMATCH
  Installed: 24V solenoid (model: SMC-VQZ115)
  Available: 12V supply (boat's house battery)

Options:
  1. 24V solenoid + DC-DC boost converter ($12) — keeps current solenoid
  2. Replace with 12V solenoid ($18) — simpler wiring
  3. 24V solenoid + 12V/24V relay trigger ($6) — cheapest

Recommendation: Option 1 (keep existing solenoid, add boost converter)
  Boost converter: MT3608 ($2.50), set to 24V, 2A max
```

## Repository Contents

| File | Purpose |
|------|---------|
| `DECKBOSS-HARDWARE.md` | Recommended Jetson setup, carrier board, power |
| `MARINE-WORKFLOW.md` | Step-by-step: walk on boat → plug in → talk → done |
| `FAILSAFE-DESIGN.md` | How every automated system has a mechanical override |
| `VOICE-INTERFACE.md` | Complete voice command reference for helm POV |

## License

AGPL-3.0. See LICENSE file.
