# Marine Workflow — Walk On, Plug In, Talk, Done

## The Full Journey

```
┌──────────────┐
│  WALK ON     │  ── Carry Deckboss case + ESP32 box
├──────────────┤
│  CONNECT     │  ── Power Deckboss. It boots, creates vessel room.
├──────────────┤
│  DISCOVER    │  ── Say "Describe the boat" → Deckboss scans for ESP32s
├──────────────┤
│  INSTALL     │  ── Place ESP32s at each control point (rudder, throttle, etc.)
├──────────────┤
│  CALIBRATE   │  ── Say "Calibrate steering" → Deckboss runs calibration
├──────────────┤
│  TEST        │  ── Say "Test all systems" → Deckboss exercises each
├──────────────┤
│  OPERATE     │  ── Say "Port 20°" → boat responds. Voice-first helm
├──────────────┤
│  DIAGNOSE    │  ── Deckboss monitors. Says "Rudder drift detected"
├──────────────┤
│  IMPROVE     │  ── Say "Response too slow" → Deckboss tunes parameters
└──────────────┘
```

## Step-by-Step

### 1. Walk On

You arrive at the boat carrying:
- **Deckboss case** (Jetson + PLATO server + voice mic in IP65 box)
- **ESP32 bag** (5+ pre-programmed ESP32s with cables)
- **Mounting kit** (zip ties, adhesive mounting pads, Deutsch connectors)

### 2. Connect

Plug Deckboss into 12V house battery. Wait 30 seconds for boot.

```
Deckboss: "Deckboss online. No ESP32s detected yet."
Captain: "Scan for devices"
Deckboss: "Scanning..." (listens for ensign tiles)
```

### 3. Discover

Walk around the boat. At each place you want to add automation, plug in an ESP32 and mount it. As each ESP32 boots, it publishes an ensign tile.

```
Deckboss: "Detected new device: ESP32-RUDDER"
Deckboss: "Detected new device: ESP32-THROTTLE"
Deckboss: "Detected new device: ESP32-ENGINE"
Deckboss: "3 devices online. Creating vessel room 'cocapn-vessel'"
Deckboss: "Vessel room created. 3 child rooms linked."
```

### 4. Install

For each ESP32, connect the mechanical system:

**Rudder:**
1. Mount ESP32 near rudder post
2. Connect servo to ESP32 (PWM output)
3. Connect position feedback potentiometer
4. Run manual override cable (mechanical, fail-safe)
5. Plug into 5V power bus

**Throttle:**
1. Mount ESP32 at engine controls
2. Connect servo to throttle linkage
3. Connect manual override cable
4. Connect tachometer sensor

### 5. Calibrate

```
Captain: "Calibrate steering"
Deckboss: "Rudder calibration started. Moving to hard port..."
         "Rudder: full port. Pulse: 1200μs. Feedback: 0°"
         "Moving to center..."
         "Rudder: center. Pulse: 1500μs. Feedback: 90°"
         "Moving to hard starboard..."
         "Rudder: full starboard. Pulse: 1800μs. Feedback: 180°"
         "Calibration complete."
         "Range: 0° - 180° (1200μs - 1800μs)"
         "Deadband: ±2° around center"
         "Calibration saved to NVS."
```

### 6. Test

```
Captain: "Test all systems"
Deckboss: "Running system test..."
          
          Steering: sweep port → center → starboard... ✅ (1.2s full travel)
          Throttle: idle → 50% → WOT... ✅ (0.8s full travel)
          Engine monitor: temp=42°C, oil=48psi, rpm=0... ✅
          
          Fail-safe check:
            Disconnect rudder servo... ✅ Manual cable engaged
            Kill throttle signal... ✅ Linkage dropped to idle
            Power cycle ESP32... ✅ Reconnects in 7s
          
          All systems nominal.
```

### 7. Operate

From the helm, everything is voice:

| Command | Response |
|---------|----------|
| "Port 10°" | Rudder turns port, confirms when at angle |
| "Starboard 20°" | Full starboard turn |
| "Throttle 1800" | Sets to 1800 RPM |
| "Idle" | Throttle drops to idle |
| "What's our fuel burn?" | "6.2 GPH at current speed" |
| "Status report" | Full system overview |
| "Hold course 045°" | Autopilot mode (if GPS connected) |
| "Emergency stop" | Throttle idle + rudder center + bilge pumps on |

### 8. Diagnose

Deckboss continuously monitors all systems. If something drifts, it speaks up:

```
Deckboss: "⚠️  Rudder anomaly: commanded 12° starboard, actual 9°."
         "Drift of 3° sustained for 8 seconds."
         "Likely: hydraulic fluid cold (engine just started)"
         "Action: increasing PID proportional gain by 15%"
Captain: "What changed?"
Deckboss: "PID Kp increased from 1.2 to 1.38."
         "Rudder error now: 1.2° — within tolerance."
         "New parameters saved."
```

### 9. Improve

```
Captain: "Throttle response is too slow"
Deckboss: "Current throttle servo speed: 0.8s from idle to WOT."
         "Options: Increase PWM frequency, increase servo voltage, or change servo."
         "Increasing PWM frequency from 50Hz to 100Hz..."
         "New throttle speed: 0.4s idle to WOT."
         "Test?"
Captain: "Test"
Deckboss: "Idle → 50%: 0.2s. 50% → WOT: 0.2s. Total: 0.4s. ✅"
         "New throttle parameters saved to ESP32 NVS."
         "This change persists across reboots."
```

## Maintenance Mode

When the boat is docked:

```
Captain: "Maintenance mode"
Deckboss: "Maintenance mode activated. Operation disabled."
         "Available commands: status, logs, configure, backup, restore"

Captain: "Backup config"
Deckboss: "Backing up all ESP32 configurations..."
         "Rudder: saved. Throttle: saved. Engine: saved."
         "Backup saved to /decksboss/backups/2026-05-08.tar.gz"
         "3 room configurations backed up to PLATO."
```

## Emergency Procedures

### Loss of Communication
If Deckboss loses connection to an ESP32:
1. **Immediate:** All ESP32s enter fail-safe (rudder stays, throttle idles)
2. **Deckboss says:** "Lost communication with ESP32-RUDDER. Fail-safe engaged."
3. **Action:** Captain uses manual override (mechanical cable)
4. **Recovery:** When ESP32 reconnects, Deckboss says "Rudder reconnected. Manual override disengaged — verify control."

### Single Point of Failure
If Deckboss core (Jetson) fails:
1. Each ESP32 continues running its last known good configuration
2. Captain operates with mechanical overrides
3. No voice interface — but all automated systems stay in their current state
4. Reboot Deckboss — systems sync on reconnect

### Complete Power Loss
1. All ESP32s fail to safe (rudder centers, throttle idles)
2. Mechanical overrides work without power
3. Engine still runs — throttle linkage is mechanical
4. Restore power → everything resumes within 30 seconds

## Example: First Day on a New Boat

```
Captain: "Describe the boat"
Deckboss: "0 devices detected. This boat has no PLATO nodes yet."
Captain: "Install 5 ESP32s"
Deckboss: "Ready. Install and power on each device."

[2 hours of installation]

Deckboss: "5 devices online."
          "1. ESP32-RUDDER — servo + feedback + CAN"
          "2. ESP32-THROTTLE — servo + tacho"
          "3. ESP32-ENGINE — coolant temp, oil pressure, RPM"
          "4. ESP32-BATTERY — voltage, current (house bank)"
          "5. ESP32-BILGE — water level, pump control"
          
          "Vessel room created: 'cocapn-vessel'"
          "Please calibrate each system."

Captain: "Calibrate steering"

[5 minutes of calibration]

Deckboss: "All systems calibrated and tested."

Captain: "Port 15°"
Deckboss: "Port 15° — moving. ETA 2 seconds."
         "At 15° port. ✅"

Captain: "I think I'm going to like this."
```
