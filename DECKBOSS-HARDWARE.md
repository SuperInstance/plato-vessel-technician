# Deckboss Hardware — Recommended Setup

## System Architecture

```
┌──────────────────────────────────────────────────────┐
│                  PILOTHOUSE / BRIDGE                 │
│                                                      │
│  ┌──────────────────┐    ┌──────────────────┐       │
│  │ Deckboss Core    │    │ Voice Interface  │       │
│  │ (Jetson Orin NX) │◄──►│ (USB mic + speaker)       │
│  │ PLATO server +   │    │                  │       │
│  │ agent runtime    │    └──────────────────┘       │
│  └───────┬──────────┘                               │
│          │                                           │
│          │ USB / CAN / RS-485                        │
│          │                                           │
└──────────┼───────────────────────────────────────────┘
           │
     ┌─────┴─────┬──────────────┬──────────────┐
     │           │              │              │
  ┌──▼──┐   ┌───▼───┐    ┌───▼───┐    ┌─────▼────┐
  │ESP32│   │ESP32  │    │ESP32  │    │ESP32     │
  │Ruddr│   │Thrott│    │Engine│    │Battery   │
  │Ctrl │   │Ctrl   │    │Monitor│    │Monitor   │
  └─────┘   └───────┘    └───────┘    └──────────┘
```

## Deckboss Core (Jetson Orin NX)

The recommended brain of the system:

| Component | Part | Price | Notes |
|-----------|------|-------|-------|
| **Module** | NVIDIA Jetson Orin NX 16GB | $599 | 40 TOPS AI, 6-core Cortex-A78AE |
| **Carrier Board** | Seeed Studio A603 | $99 | Rugged, automotive temp range |
| **Storage** | Samsung 990 Pro 1TB NVMe | $120 | OS + PLATO + log storage |
| **RAM** | On-module 16GB | Included | |
| **Power** | 12V marine input (8-36V) | — | Direct from house battery |
| **WiFi** | On-module | Included | Use wired ethernet for bridge, WiFi for crew tablets |

**Total: ~$820**

### Why Jetson and not RPi

- **Reliability:** Passive cooling, automotive temp range (-25°C to 80°C)
- **AI acceleration:** Real-time audio processing for voice commands
- **Vision:** Connect cameras for engine room monitoring, dock assist
- **CAN bus:** Native CAN FD support for NMEA 2000 gateway

### Alternative: Cheaper Setup

| Component | Part | Price |
|-----------|------|-------|
| **Module** | Raspberry Pi 5 (8GB) | $80 |
| **Storage** | 128GB microSD + 256GB USB SSD | $40 |
| **ADC** | Waveshare USB-CAN adapter | $25 |
| **Case** | Waterproof IP65 enclosure | $35 |
| **Total** | | **$180** |

The RPi 5 handles voice recognition, PLATO server, and 4-6 ESP32s. Add a Coral TPU ($60) for AI audio processing.

## Power System

```
┌────────────────────────────────────────────────────┐
│                 12V HOUSE BUS                       │
│  ┌──────────┐   ┌──────────┐   ┌──────────────┐   │
│  │ Startup  │   │ House    │   │ Electronics  │   │
│  │ Battery  │   │ Battery  │   │ Battery      │   │
│  │ (crank)  │   │ 200Ah Li │   │ 50Ah Li      │   │
│  └──────────┘   └────┬─────┘   └──────┬───────┘   │
│                      │                │           │
│                 ┌────▼────┐     ┌─────▼──────┐    │
│                 │ ACR     │     │ DC-DC      │    │
│                 │ (auto   │     │ 12V→5V     │    │
│                 │  combine│     │ (ESP32s)   │    │
│                 └─────────┘     └────────────┘    │
└────────────────────────────────────────────────────┘
```

- **Jetson/RPi:** Direct from house battery (12V input, wide range 8-36V)
- **ESP32s:** 5V bus from DC-DC converter (LM2596 or equivalent)
- **Critical systems (rudder, throttle):** Dual power — main bus + backup from electronics battery
- **Voice system:** USB powered from Jetson/RPi

## ESP32 Node Types

### Rudder Controller
```
ESP32 ──┬── Servo (rudder actuator, GPIO18, PWM)
        ├── Position feedback (potentiometer, ADC GPIO36)
        ├── Manual override sensor (limit switch, GPIO15)
        ├── CAN bus (MCP2515, SPI — for NMEA 0183 gateway)
        └── Power: 5V bus (500mA peak)
```

### Throttle Controller
```
ESP32 ──┬── Servo (throttle cable actuator, GPIO18, PWM)
        ├── Tachometer input (hall sensor, GPIO4, interrupt)
        ├── Mechanical linkage sensor (draw wire encoder, ADC GPIO39)
        └── Power: 5V bus (300mA peak, 1A with servo)
```

### Engine Monitor
```
ESP32 ──┬── Temperature (DS18B20, GPIO4, one-wire — coolant)
        ├── Temperature (DS18B20, GPIO15, one-wire — exhaust)
        ├── Oil pressure (analog sender, ADC GPIO36)
        ├── RPM (inductive pickup on ignition wire, GPIO34)
        ├── Fuel level (resistive sender, ADC GPIO39)
        └── Power: 5V bus (200mA)
```

### Battery Monitor (per bank)
```
ESP32 ──┬── Voltage divider (12V → 3.3V, ADC GPIO36)
        ├── Current sensor (ACS758, ADC GPIO39)
        ├── Temperature (DS18B20, GPIO4 — battery terminal)
        └── Power: 5V bus (150mA)
```

### Bilge Pump Controller
```
ESP32 ──┬── Water level sensor (float switch, GPIO12)
        ├── MOSFET (pump relay, GPIO18, 12V pump)
        ├── Current sensor (pump run detection, ADC GPIO36)
        └── Power: 5V bus (200mA + pump relay coil)
```

## Waterproofing

- **ESP32s:** Pot in epoxy (MG Chemicals 832HD) or use pre-potted modules
- **Connectors:** Deutsch DT series (IP67) for all sensor connections
- **Cable:** Tinned marine wire (Ancor), adhesive-lined heat shrink
- **Jetson/RPi:** IP65 enclosure (Polycase ML-55F) with cable glands
- **Antenna:** External WiFi antenna (Shakespeare 3056-RD) on the flybridge

## Networking

```
┌──────────────────┐
│  Deckboss Core   │
│  ┌─────────┐     │
│  │ WiFi AP │─────│─── Crew tablets (chart plotter, engine room)
│  │ 10.0.1.1│     │
│  └─────────┘     │
│  ┌─────────┐     │
│  │ Ethernet│─────│─── NMEA gateway (0183/2000 → PLATO)
│  │ 10.0.1.2│     │
│  └─────────┘     │
│  ┌─────────┐     │
│  │ WiFi STA│─────│─── Cellular (harbor WiFi or Starlink)
│  │ DHCP    │     │
│  └─────────┘     │
└──────────────────┘

ESP32s connect to Deckboss's WiFi AP:
  SSID: deckboss-nnnn (random suffix)
  Password: printed on Deckboss core case
  IP range: 10.0.1.50 - 10.0.1.100
```

**No internet required.** Deckboss runs entirely on local network. Cellular/Starlink is optional for remote alerts.
