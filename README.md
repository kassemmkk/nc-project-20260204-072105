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
**Completion:** 15%

## 📋 Complete Documentation

### **[📘 SYSTEM DATASHEET - START HERE](docs/SYSTEM_DATASHEET.md)**

This comprehensive 54-page technical datasheet is designed for outsourcing to an engineering house. It includes:

#### **Detailed Power Budget Analysis with Calculations**
- Battery capacity analysis (CR2032: 220 mAh → 151 mAh usable after derating)
- Operating mode breakdown with duty cycles
- Current consumption for every component in every mode
- **Three scenarios analyzed:**
  - Worst-case: 104 µA (2 months life) ❌
  - Optimized: 43 µA (4.8 months life) ⚠️
  - Motion-gated: 16.4 µA (12.8 months life) ✅
- Justification for every design decision

#### **Complete Audio ADC Specification**
- Why custom ADC is required (no commercial parts meet ≤500µA @ 16kHz)
- Sample rate selection: Why 16 kHz (pig vocalization analysis)
- Resolution selection: Why 12-14 bit (50 dB dynamic range requirement)
- Sigma-delta architecture specification
- Full Statement of Work for analog design house
- $200k NRE budget with milestone-based payments

#### **System Architecture**
- Block diagrams showing all external components
- Signal flow diagrams for audio, temperature, activity paths
- Interface specifications (I2S, I2C, SPI, UART, GPIO)
- Pin assignments for Caravel integration

#### **Environmental & Mechanical Specs**
- Form factor: 25×25×8 mm, <10g with battery
- IP67 enclosure (dust-tight, water-immersive)
- Operating temperature: -10°C to +50°C
- Mounting on pig ear (industry-standard snap-through tag)
- Durability testing (drop, vibration, UV, water immersion)

#### **Wireless Link Budget**
- nRF24L01+ at 2.4 GHz, 0 dBm (1 mW)
- Path loss calculation for 100m range
- +1 dB link margin (tight but acceptable)
- Packet format (32 bytes: device ID, timestamp, events, sensor data)

#### **Health Monitoring Vitals**
- **Acoustic:** Cough detection (500Hz-4kHz, 0.2-0.5s duration)
- **Temperature:** Continuous fever monitoring (>40°C alert)
- **Activity:** Accelerometer-based lethargy detection
- **Correlation:** Multi-parameter health scoring

#### **Complete Bill of Materials**
- $14.50 per unit at 100k volume
- Cost reduction path to <$12 at 500k volume
- Detailed component selection with alternates

#### **Testing & Validation Plan**
- Component characterization procedures
- System integration testing (benchtop + environmental)
- Field trial protocol (pilot + 6-month validation study)
- Success criteria: >80% cough detection accuracy, <10% false positives

#### **Risk Analysis & Mitigation**
- Technical risks (ADC power, detection accuracy, battery life)
- Schedule risks (12-month ADC design, 6-month field trials)
- Cost risks (NRE overruns, component shortages)

#### **30-Month Project Timeline**
- Phase 1 (Months 1-12): Design & prototyping
- Phase 2 (Months 13-18): Verification & testing
- Phase 3 (Months 19-24): Field trials
- Phase 4 (Months 25-30): Production ramp

See [docs/SYSTEM_DATASHEET.md](docs/SYSTEM_DATASHEET.md) for the complete specification.

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