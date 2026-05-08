# Fail-Safe Design — Mechanical Override for Every Automated System

## Philosophy

Every automated system on a boat must have a manual backup that works without electricity, without software, and without thought. The override should be:

1. **Obvious** — a crew member who has never seen the system before should find and operate the override in under 5 seconds
2. **Independent** — no dependency on the automated system's power or control
3. **Unambiguous** — pulling the override cable produces a mechanically certain result
4. **Testable** — testable at the dock before every trip
5. **Overrideable** — even the override mechanism has a backup (pry bar)

## Steering (Rudder)

### Automated Path
```
Deckboss voice command →[digital]→ ESP32 →[PWM]→ Servo →[push/pull]→ Rudder quadrant
```

### Fail-Safe 1: Electrical Disconnect
```
Pull a pin → Servo arm disconnects → Rudder quadrant only connected to manual steering cable
```
- The servo arm has a quick-release pin (R-clip, colored red)
- Pull the pin → servo arm swings free → manual helm cable is the only connection
- **Test:** Pull pin at dock, confirm manual steering works. Replace pin.

### Fail-Safe 2: Servo Seizure
```
If servo jams: Break-away linkage at 50lb force → Servo disconnects → Manual cable takes over
```
- The linkage between servo and quadrant has a break-away point (shear pin or magnetic coupling)
- Design force: 50 lbs (exceeds normal steering load, breaks before servo damage)
- **Spare shear pins:** Stored in a tube next to the rudder post

### Fail-Safe 3: Complete ESP32 Failure
```
No PWM output → Servo input floats → 10kΩ pull-down resistor → Servo goes to "center" position
```
- When ESP32 crashes, servo PWM input goes to 0V
- 10kΩ pull-down on the signal line ensures servo centers
- Centered rudder = straight line = safe (not turning)

### Fail-Safe 4: Power Loss
```
No power to servo → Servo has no holding torque → Springs center the rudder
```
- Torsion springs on the rudder quadrant return it to center
- Spring force: enough to center against water flow, not enough to fight manual steering
- **Note:** Springs are always engaged — they provide centering force even during normal operation

## Throttle

### Automated Path
```
Deckboss voice → ESP32 →[PWM]→ Servo →[push/pull]→ Throttle lever → Engine governor
```

### Fail-Safe 1: Cable Override
```
Pull override cable → Servo connects directly to lever → Manual control
```
- The servo connects to the throttle lever via a Bowden cable
- A second Bowden cable runs alongside it (the override)
- Pull the "MANUAL" tag → override cable engages → servo cable disengages
- **Color:** Override cable is bright yellow. Servo cable is black.

### Fail-Safe 2: Spring Return to Idle
```
Loss of signal → Servo spring → Throttle lever spring → Idle position
```
- Dual spring mechanism: servo has a return spring, throttle lever has a stronger spring
- On signal loss, both springs act together
- Engine returns to idle (safe position — no power, but steerable)

### Fail-Safe 3: Physical Stop
```
Servo over-travel → Physical stop at WOT → Cannot exceed max RPM
```
- A mechanical stop on the throttle linkage prevents exceeding safe RPM
- Adjusted during installation based on engine specs
- **Human override:** Stop can be moved with tools, but only when engine is off

## Engine Monitor

### Automated Path
```
ESP32 reads sensors → Publishes PLATO tiles → Deckboss monitors
```

### Fail-Safe Strategy
```
The engine monitor is read-only. It cannot cause damage by failing.
If it fails: Captain glances at mechanical gauges on the dashboard.
```

**No engine system is controlled by the monitor ESP32** — it only reads. Fail-safe is inherent.

## Battery Monitor

### Automated Path
```
ESP32 reads voltage/current → PLATO tiles → Deckboss alerts on low battery
```

### Fail-Safe Strategy
```
The battery monitor is read-only. It cannot cause damage by failing.
Alerts are advisory — the captain still checks the battery gauge.
Same inherent safety as engine monitor.
```

## Bilge Pump

### Automated Path
```
ESP32 reads water level → If detected, MOSFET activates pump → Repeat until clear
```

### Fail-Safe 1: Bilge Pump Float Switch (Independent)
```
Water rises beyond ESP32 level → Mechanical float switch → Directly connects pump to 12V
```
- The bilge pump has TWO triggers: the ESP32-controlled MOSFET AND a mechanical float switch
- The float switch is wired in parallel with the MOSFET
- If ESP32 fails or drops WiFi, the float switch still turns on the pump
- **Test:** Lift the float switch manually. Pump should start regardless of ESP32 state.

### Fail-Safe 2: Secondary Pump
```
If ESP32 pump fails AND float switch fails → Second pump with independent float switch → Connected to electronics backup battery
```
- Two bilge pumps, each with its own float switch
- Each on a different electrical circuit
- If both fail, the crew hears the water sloshing. That's the final warning.

## Autopilot Mode

### Automated Path
```
Deckboss (heading command) → ESP32 → Servo → Rudder
```

### Fail-Safe: One-Button Disengage
```
Press "STANDBY" button on helm → ESP32 receives interrupt → Exits autopilot → Manual steering resumes
```
- The STANDBY button is a physical button on the bridge — big, red, labeled "MANUAL"
- It's wired directly to an ESP32 GPIO interrupt
- Pressing it exits autopilot mode immediately (no software debate)
- Also activates the servo disconnect (fail-safe 1 from steering)

## Integration Test

Before every trip, run this sequence:

```
1. Power on Deckboss + all ESP32s
2. Say "Test fail-safes"
3. Deckboss runs:

   ✅ Rudder servo: disconnect pin → manual steering works
   ✅ Rudder servo: pull-down → servo centers → ✅
   ✅ Throttle: pull MANUAL tag → override engages → idle → ✅
   ✅ Bilge: lift float switch → pump runs independent of ESP32 → ✅
   ✅ Autopilot: press STANDBY → manual steering → ✅
   
   All fail-safes verified. Vessel is safe to operate.
```

## Override Testing Schedule

| System | Frequency | Visual Check |
|--------|-----------|-------------|
| Rudder disconnect pin | Every trip | Red pin present, cotter pin intact |
| Rudder break-away | Monthly | Shear pin not brittle |
| Throttle override cable | Every trip | Yellow cable moves freely |
| Throttle return spring | Monthly | No rust, spring preload OK |
| Bilge float switch | Every trip | Float moves freely |
| Autopilot STANDBY | Every trip | Button illuminates |
| ALL overrides | Annual | Full mechanical inspection |

## The Override Label

Every override point has a label printed on waterproof vinyl:

```
╔══════════════════════════════╗
║    ⚠️  MANUAL OVERRIDE       ║
║                              ║
║  1. Pull RED PIN            ║
║  2. Confirm manual control  ║
║  3. Stow pin in holder      ║
║                              ║
║  Re-engage: Insert pin       ║
║  Test with "MANUAL" mode     ║
╚══════════════════════════════╝
```

## Final Word

> "If every wire rots and every chip fries, the boat should still sail home."
> — Deckboss design principle

The automation makes the boat better. The overrides ensure it's never worse.
