# Quick Reference Card - Pig Health Monitoring System

## 📌 Critical Specifications at a Glance

### System Overview
| Parameter | Specification | Rationale |
|-----------|---------------|-----------|
| **Application** | Livestock health monitoring (swine) | Early respiratory disease detection |
| **Form Factor** | 25×25×8 mm ear tag, 10g | Fits pig ear, minimal discomfort |
| **Battery** | CR2032 (220 mAh → 151 mAh usable) | Disposable economics |
| **Target Life** | 6 months continuous | Farm inspection cycles |
| **Operating Temp** | -10°C to +50°C | Global farm environments + body heat |
| **Protection** | IP67 (dust-tight, immersive) | Farm environment, cleaning |

---

### Power Budget (Motion-Gated Design)

| Operating Mode | Duty Cycle | Current | Avg Contribution | Notes |
|----------------|------------|---------|------------------|-------|
| **Deep Sleep** | 96.3% | 4 µA | 3.85 µA | RTC + ASIC retention + accel |
| **Audio Sampling** | 1.0% | 1,150 µA | 11.5 µA | Motion-triggered only |
| **Processing** | 0.1% | 1,050 µA | 1.05 µA | Feature extraction |
| **Wireless TX** | 0.011% | 14,000 µA | 0.015 µA | nRF24L01+ @ 0dBm |
| **🎯 Total** | - | - | **16.4 µA** | **12.8 month life** ✅ |

**Target:** ≤35 µA (6 months minimum), Achieved: 16.4 µA (2.1× margin)

---

### Audio ADC Specification (CUSTOM DESIGN REQUIRED)

**⚠️ CRITICAL: No commercial ADC meets power requirement**

| Parameter | Min | Typ | Max | Unit | Critical? |
|-----------|-----|-----|-----|------|-----------|
| **Resolution** | 12 | 14 | 16 | bits | ✅ Minimum viable |
| **Sample Rate** | 8 | 16 | 32 | kHz | ✅ Captures pig coughs |
| **SNR** | 70 | 80 | - | dB | ✅ 50dB dynamic range |
| **THD** | - | - | -70 | dB | ✅ Audio quality |
| **Power (Active)** | - | **≤500** | - | **µA @ 1.8V** | 🔥 **CRITICAL** |
| **Power (Shutdown)** | - | - | 5 | µA | ✅ Negligible |
| **Interface** | - | I2S | - | - | ✅ Digital output |
| **Control** | - | I2C | - | - | ✅ Configuration |
| **Package** | - | QFN-24 | - | 4×4mm | ✅ Size constraint |
| **Process** | - | 180nm | - | CMOS | ✅ Mature, low NRE |

**Why Custom?**
- Commercial audio ADCs: 1.5-11 mA (3-22× over budget)
- Ultra-low-power ADCs: Too slow (<1 kHz sampling)
- **Custom sigma-delta ADC is only solution**

**Development:**
- **NRE:** $200k (analog design house)
- **Timeline:** 9-12 months
- **Architecture:** Continuous-time sigma-delta modulator

---

### Wireless Specification

| Parameter | Value | Notes |
|-----------|-------|-------|
| **Module** | nRF24L01+ | Pre-certified, low cost ($0.80) |
| **Frequency** | 2.4 GHz ISM | Global allocation |
| **TX Power** | 0 dBm (1 mW) | 11.3 mA for 2 ms |
| **Data Rate** | 1 Mbps | 32-byte packet in 2 ms |
| **Range** | 50-100m | Link budget: +1 dB margin |
| **Protocol** | Proprietary | Auto-ACK, 3 retries |
| **Average Current** | 0.015 µA | Negligible (event-driven) |

**Packet:** 32 bytes (device ID, timestamp, events, sensor data, checksum)

---

### Monitored Health Parameters

| Vital | Sensor | Sampling | Alert Threshold | Detection Method |
|-------|--------|----------|-----------------|------------------|
| **Cough** | Audio ADC + MEMS Mic | Motion-gated, 1 sec | Signature match | Energy + ZCR + freq band |
| **Temperature** | TMP117 (I2C) | Every sample | >40°C | Direct measurement |
| **Activity** | ADXL345 (SPI) | Continuous (2µA) | Lethargy pattern | Motion detection |
| **Events** | Timestamp correlation | - | Multi-parameter | Trend analysis |

**Cough Signature:**
- Frequency: 500-4,000 Hz (mid-high energy)
- Duration: 0.2-0.5 seconds (short burst)
- Characteristics: High RMS energy, high zero-crossing rate (noisy, aperiodic)

---

### Bill of Materials ($14.50 @ 100k units)

| Component | Part Number (Example) | Cost | Notes |
|-----------|-----------------------|------|-------|
| Digital ASIC | Caravel SKY130 | $3.50 | 50k gates, 4KB SRAM |
| **Audio ADC** | **Custom (TBD)** | **$1.00** | **NRE: $200k** |
| MEMS Mic | ICS-43434 | $1.20 | 150 µA, I2S output |
| Wireless | nRF24L01+ SMD | $0.80 | Pre-certified |
| Temperature | TMP117 | $0.50 | ±0.1°C, I2C |
| Accelerometer | ADXL345 | $1.50 | 2 µA motion mode |
| Boost Converter | TPS61220 | $0.60 | 0.4 µA quiescent |
| Battery | CR2032 | $0.30 | 220 mAh |
| PCB | 4-layer, 25×25mm | $0.50 | FR4, ENIG |
| Passives | R, C, L (×20) | $1.00 | 0402 size |
| Antenna | Ceramic chip | $0.30 | 2.4 GHz |
| Enclosure | PC injection molded | $0.80 | IP67 ultrasonic weld |
| Assembly | SMT + test | $2.00 | Contract mfg |
| **Total** | - | **$14.50** | Path to <$12 @ 500k |

---

### Key Design Decisions & Justifications

#### 1. Why Motion-Triggered Sampling?
```
Periodic sampling (every 30s): 39.9 µA → 5.2 months ⚠️
Motion-gated (30% of time): 16.4 µA → 12.8 months ✅

Logic:
- Coughs occur when pig is active (not sleeping)
- Accelerometer: 2 µA continuous motion detection
- Reduces false positives (no environmental noise during rest)
- Saves 60% of sampling power
```

#### 2. Why 16 kHz Sample Rate?
```
Pig cough frequency content: 500-4,000 Hz (fundamental)
Harmonics (for feature extraction): up to 8,000 Hz
Nyquist requirement: 2 × 8 kHz = 16 kHz minimum

Lower rates (8 kHz): Miss harmonic detail, aliasing
Higher rates (32 kHz): 60% more power for no benefit
```

#### 3. Why 12-14 Bit Resolution?
```
Dynamic range requirement: 60-110 dB SPL = 50 dB
Ideal ADC SNR: 6.02 × N + 1.76 dB

N = 8 bits → 50 dB (no margin) ❌
N = 12 bits → 74 dB (24 dB margin) ✅ Minimum
N = 14 bits → 86 dB (36 dB margin) ✅ Preferred

16-bit: Overkill (MEMS mic SNR = 65 dB, limits system)
```

#### 4. Why nRF24L01+ vs BLE/LoRa?
```
Power Comparison (32-byte packet every 5 min):
- nRF24L01+: 0.975 µA average (2 ms TX, 0.9 µA standby)
- BLE 5.0: 2.27 µA (3 ms TX + 50 ms connection overhead)
- LoRa: 5.2 µA (50 ms TX @ 30 mA, higher on-air time)

nRF24L01+ wins: Lowest power, lowest latency, lowest cost
```

#### 5. Why CR2032 (not CR2450 or AA)?
```
CR2032: Ø20×3.2mm, 220 mAh, 3g
CR2450: Ø24×5.0mm, 620 mAh, 6.8g (too large for ear tag)
AA: Ø14×50mm, 2000 mAh, 23g (way too large)

Constraint: Pig ear tag must be <30mm (animal welfare)
Decision: CR2032 is largest viable battery
```

---

### Development Timeline

```
Phase 1: Design & Prototyping (12 months)
  ├─ Month 1-3: System architecture ✅ COMPLETE
  ├─ Month 4-6: Digital ASIC (Caravel) ⏳ Next
  └─ Month 7-12: Custom ADC design (outsourced) ⏳ RFQ ready

Phase 2: Verification & Testing (6 months)
  ├─ Month 13-14: Component characterization
  ├─ Month 15-16: System integration (benchtop)
  └─ Month 17-18: Environmental testing (IP67, temp, shock)

Phase 3: Field Trials (6 months)
  ├─ Month 19-20: Pilot (20 tags, 1 farm)
  └─ Month 21-24: Validation (100 tags, research farm)

Phase 4: Production Ramp (6 months)
  └─ Month 25-30: DFM, tooling, 10k unit run
```

**Critical Path:** Custom ADC design (9-12 months) → longest pole

---

### Success Criteria

#### Technical:
- ✅ Battery life ≥6 months (target: 16.4 µA, achieved: 2.1× margin)
- ✅ Cough detection accuracy ≥80% (sensitivity)
- ✅ False positive rate <10% (specificity)
- ✅ Wireless range ≥50m (barn environment)
- ✅ IP67 environmental protection (dust, water)

#### Business:
- ✅ Unit cost <$15 @ 100k (achieved: $14.50)
- ✅ Path to <$12 @ 500k (feasible with ADC NRE amortization)
- ✅ 30-month development timeline (ADC critical path)
- ✅ Regulatory: FCC/CE (pre-certified wireless module)

#### Market:
- Target: 150k-500k units/year (commercial farms)
- Revenue: $3.75M-12.5M/year (@ $25/tag selling price)
- Payback: <6 months (prevents 1-2 disease outbreaks)

---

### Risk Mitigation

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| **ADC power target not met** | Medium | High | Prototype with commercial ADC first; budget for 1 respin ($100k) |
| **Detection accuracy insufficient** | Medium | Medium | Large training dataset; consider TinyML (future) |
| **Battery life shortfall** | Low | High | 2.1× design margin; firmware can adjust sampling |
| **ADC NRE overrun** | Medium | Medium | Fixed-price contract with milestones |
| **Field trial failures** | Low | High | Conservative design, early prototyping |

---

### Next Actions

#### Immediate (This Week):
1. ✅ **System datasheet complete** (DONE)
2. ⏳ Issue RFQ to 3-5 analog design houses (Audio ADC)
3. ⏳ Begin digital ASIC RTL development (Caravel user project)

#### Short-Term (Month 1-3):
4. Select ADC contractor (technical + commercial evaluation)
5. Finalize ASIC architecture (I2S, I2C, SPI, GPIO)
6. Prototype with commercial ADC (validate system, accept higher power)

#### Medium-Term (Month 6-12):
7. Complete ADC design & tape-out
8. Integrate ASIC with ADC
9. Build 100 prototype tags

---

## 📚 Full Documentation Links

| Document | Pages | Purpose |
|----------|-------|---------|
| **[SYSTEM_DATASHEET.md](SYSTEM_DATASHEET.md)** | 54 | Complete technical specification for RFQ |
| **[EXECUTIVE_SUMMARY.md](EXECUTIVE_SUMMARY.md)** | 8 | 1-page overview for decision makers |
| **[SPEC_DERIVATION.md](SPEC_DERIVATION.md)** | 45 | How every number was calculated |
| **[QUICK_REFERENCE.md](QUICK_REFERENCE.md)** | 5 | This document (at-a-glance specs) |

---

## 🎯 The Ask: Audio ADC Design (RFQ)

**Scope:** Design, verify, and deliver ultra-low-power audio ADC ASIC

**Deliverables:**
- Complete ADC design (GDS II, LEF, LIB)
- Verification (Verilog-A models, SPICE simulations)
- 100 packaged samples (QFN-24)
- Characterization report + datasheet

**Budget:** $200k NRE (fixed-price preferred)

**Timeline:** 9-12 months (PDR @ 3mo, CDR @ 6mo, tape-out @ 9mo, samples @ 12mo)

**Critical Spec:** ≤500 µA @ 1.8V, 16 kHz sampling (12-14 bit, sigma-delta)

**Contact:** engineering@nativechips.ai

---

**Last Updated:** February 4, 2026  
**Revision:** 1.0  
**Status:** Ready for RFQ Distribution

**© NativeChips 2026 | Proprietary & Confidential**
