# Pig Health Monitoring System - Digital Control Subsystem

## Project Overview

**Original User Request:**
> Low power sound recognition with low power wireless connectivity to local hubs. On the ear of a pig that works for at least 6 months on a button cell, disposable. Ability to detect all the time for a specific animal. Example to focus on the one animal while isolating the noise of other animals around. Like when a pig gets ill, coughing. Suggest other vitals that could be integrated.

## System Architecture

This Caravel user project implements the **digital control subsystem** for an ultra-low-power livestock health monitoring system. The ASIC provides all necessary digital interfaces to coordinate external analog/RF components for comprehensive pig health monitoring.

### Key Features
- ✅ **I2S Audio Interface** - Digital MEMS microphone input
- ✅ **SPI Master** - Wireless module + sensor communication
- ✅ **I2C Master** - Temperature sensor interface
- ✅ **4KB SRAM** - Audio sample buffering
- ✅ **Audio Feature Extractor** - Time-domain analysis (energy, threshold, zero-crossing)
- ✅ **Programmable Interrupt Controller** - Event management for up to 16 sources
- ✅ **UART Debug Interface** - Configuration and monitoring
- ✅ **GPIO Controller** - General-purpose I/O
- ✅ **Wishbone Bus Infrastructure** - Peripheral interconnect

### Power Budget
- Digital logic (sleep): ~1-5 µA
- Periodic wake-up for sampling: 10-100 µA (averaged)
- Wireless TX (duty-cycled): 50-200 µA (averaged)
- **Target: 6-month operation on CR2032 (220 mAh)**

## External Components Required

This ASIC must be integrated with the following external components:

| Component | Example Part | Interface | Purpose |
|-----------|--------------|-----------|---------|
| Digital MEMS Mic | ICS-43434, INMP441 | I2S | Audio capture |
| Wireless Module | nRF24L01+, RFM95W | SPI | Low-power wireless |
| Temperature Sensor | TMP117, MCP9808 | I2C | Fever detection |
| Accelerometer | ADXL345, LIS3DH | SPI/I2C | Activity monitoring |
| Boost Converter | TPS61220 | - | CR2032 to 1.8V |
| CR2032 Battery | - | - | Primary power |

## Monitored Health Vitals

1. **Acoustic Monitoring**
   - Cough detection (energy + pattern)
   - Respiration sounds
   - Vocalization patterns
   - Distress signals

2. **Body Temperature**
   - Continuous monitoring
   - Fever detection threshold
   - Trend analysis

3. **Activity Level**
   - Movement patterns
   - Lethargy detection
   - Behavioral changes

4. **Event Correlation**
   - Timestamp all events
   - Multi-parameter health scoring

## Project Status

**Current Stage:** Project Setup  
**Completion:** 10%

See [docs/README.md](docs/README.md) for detailed documentation.

## Quick Start

### Prerequisites
- OpenLane/LibreLane flow
- Caravel harness
- cocotb for verification
- SKY130 PDK

### Directory Structure
```
nc-project-20260204-072105/
├── verilog/
│   ├── rtl/              # RTL source files
│   └── dv/cocotb/        # Verification testbenches
├── openlane/             # Physical design configs
├── docs/                 # Documentation
├── fw/                   # Firmware examples
└── sim/                  # Simulation artifacts
```

### Build & Test
```bash
# Lint RTL
make lint

# Run verification
make verify

# Synthesize with OpenLane
make harden
```

## License
Apache 2.0