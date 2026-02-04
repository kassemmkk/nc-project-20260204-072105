# Pig Health Monitoring System - Complete System Datasheet
## Version 1.0 | Confidential | For Engineering House Outsourcing

---

# DOCUMENT CONTROL

| **Field** | **Value** |
|-----------|-----------|
| **Document Title** | Pig Health Monitoring System - Complete System Specification |
| **Document Number** | PIG-HEALTH-SYS-001 |
| **Revision** | 1.0 |
| **Date** | February 4, 2026 |
| **Status** | For External Development |
| **Confidentiality** | Proprietary - NDA Required |
| **Author** | NativeChips Systems Team |
| **Approver** | [To Be Assigned] |

---

# 1. EXECUTIVE SUMMARY

## 1.1 Project Overview

This document specifies a complete ultra-low-power livestock health monitoring system designed to operate continuously for 6 months on a single CR2032 coin cell battery. The system is deployed as an on-ear wearable tag for individual pig monitoring in commercial farming operations.

**Primary Application:** Real-time detection of respiratory illness (coughing) and other health indicators in swine populations.

**Key Innovation:** Combination of acoustic monitoring with multi-parameter vital sign tracking in a disposable, maintenance-free form factor.

## 1.2 System Requirements Summary

| **Parameter** | **Specification** | **Rationale** |
|---------------|-------------------|---------------|
| Battery Life | 6 months continuous | Commercial farm inspection cycles |
| Battery Type | CR2032 (220 mAh @ 3V) | Cost, availability, size constraint |
| Average Current | ≤50 µA | Derived from battery capacity / lifetime |
| Operating Temp | -10°C to +50°C | Farm environment + pig body heat |
| Encapsulation | IP67 | Farm environment, animal contact, cleaning |
| Wireless Range | 50-100m LOS | Barn dimensions (hub per 50-100 animals) |
| Audio Sampling | 8-16 kHz, 12-16 bit | Pig vocalization range + sufficient resolution |
| Form Factor | ≤25mm × 30mm × 8mm | Ear tag mounting constraint |
| Weight | ≤10g with battery | Animal welfare, tag retention |

## 1.3 Monitored Health Parameters

1. **Acoustic Signatures**
   - Coughing events (primary indicator for respiratory disease)
   - Respiratory sounds (wheezing, labored breathing)
   - Vocalization patterns (distress calls)
   - Feeding/drinking sounds (behavior monitoring)

2. **Body Temperature**
   - Continuous monitoring for fever detection
   - Threshold: >40°C = fever alert
   - Normal pig body temperature: 38.5-39.5°C

3. **Activity Level**
   - Movement/acceleration patterns
   - Lethargy detection
   - Abnormal behavior (head shaking, scratching indicating discomfort)

4. **Event Correlation**
   - Multi-parameter scoring algorithm
   - Timestamp all events for behavior analysis
   - Trend detection over hours/days

---

# 2. POWER BUDGET ANALYSIS (DETAILED)

## 2.1 Battery Specification

**Selected Battery:** CR2032 Lithium Coin Cell

| **Parameter** | **Typical** | **Min** | **Max** | **Unit** | **Notes** |
|---------------|-------------|---------|---------|----------|-----------|
| Nominal Voltage | 3.0 | 2.7 | 3.3 | V | End-of-life: 2.0V |
| Capacity (to 2.0V) | 220 | 180 | 240 | mAh | Temperature dependent |
| Self-discharge | 1-2 | - | - | %/year | Negligible over 6 months |
| Operating Temp | -20 to +70 | - | - | °C | Capacity reduced at extremes |
| Weight | 3.0 | - | - | g | Panasonic CR2032 |
| Dimensions | Ø20 × 3.2 | - | - | mm | Standard form factor |

**Derating for Real-World Use:**
- Temperature derating: 85% (average farm temp fluctuation)
- End-of-life voltage cutoff: 90% (stop at 2.0V, not complete discharge)
- Manufacturing variance: 90% (use conservative spec)

**Effective Capacity:** 220 mAh × 0.85 × 0.90 × 0.90 = **151 mAh usable**

## 2.2 Target Lifetime Calculation

**Target Lifetime:** 6 months = 180 days = 4,320 hours

**Maximum Average Current:**

```
I_avg_max = Capacity / Lifetime
I_avg_max = 151 mAh / 4320 hours
I_avg_max = 35 µA average (absolute maximum)
```

**Design Target with 30% Margin:**

```
I_avg_design = 35 µA × 0.70 = 24.5 µA average
```

This provides margin for:
- Temperature extremes reducing battery capacity
- Manufacturing variations
- Unexpected firmware overhead
- Battery aging during storage

## 2.3 Operating Mode Power Budget

The system operates in multiple power states with different duty cycles:

### Mode 1: Deep Sleep (Baseline)
**Duty Cycle:** 90% of time (bulk of operating life)

| **Component** | **Current** | **Justification** |
|---------------|-------------|-------------------|
| Digital ASIC (retention) | 0.5 µA | SKY130 leakage estimate at 1.8V, ~50k gates |
| SRAM retention (4KB) | 0.5 µA | Standard SRAM leakage |
| RTC/Wake timer | 0.3 µA | 32 kHz oscillator for wake scheduling |
| Temperature sensor | 0.2 µA | Nanopower mode (TMP117: 150nA typ) |
| Boost converter quiescent | 0.5 µA | TPS61220 shutdown mode: 400nA typ |
| **Total Deep Sleep** | **2.0 µA** | |

### Mode 2: Audio Sampling (Periodic)
**Duty Cycle:** 8% of time (5 seconds every minute = 8.3%)

| **Component** | **Current** | **Justification** |
|---------------|-------------|-------------------|
| Audio ADC (active) | 500 µA | Sigma-delta ADC @ 16 kHz (see Section 3) |
| Digital MEMS microphone | 150 µA | ICS-43434 typical |
| ASIC digital logic (active) | 300 µA | I2S RX, DMA, feature extraction |
| SRAM (active) | 50 µA | Buffering audio samples |
| Boost converter (loaded) | 100 µA | 80% efficiency, delivering ~900 µA |
| Temperature sensor | 50 µA | Active conversion |
| **Total Sampling Mode** | **1,150 µA** | |

**Time-Averaged Contribution:**
```
I_sampling_avg = 1,150 µA × 0.08 = 92 µA
```

### Mode 3: Processing (After Audio Sample)
**Duty Cycle:** 1% of time (post-processing after each sample)

| **Component** | **Current** | **Justification** |
|---------------|-------------|-------------------|
| ASIC (full speed) | 800 µA | 25 MHz clock, feature extraction algorithm |
| SRAM (active) | 100 µA | Reading audio buffer |
| Accelerometer | 150 µA | SPI read, ADXL345 normal mode |
| **Total Processing** | **1,050 µA** | |

**Time-Averaged Contribution:**
```
I_processing_avg = 1,050 µA × 0.01 = 10.5 µA
```

### Mode 4: Wireless Transmission (Event-Driven)
**Duty Cycle:** 1% of time (transmit every 5 minutes if event detected)

| **Component** | **Current** | **Justification** |
|---------------|-------------|-------------------|
| Wireless module (TX) | 12,000 µA | nRF24L01+ @ 0dBm: 11.3 mA typ |
| ASIC (SPI master) | 500 µA | Driving SPI at 4 MHz |
| Boost converter | 1,500 µA | Delivering ~14 mA |
| **Total TX Mode** | **14,000 µA** | |

**Transmission Profile:**
- Packet size: 32 bytes (header + sensor data + timestamp)
- Data rate: 1 Mbps (nRF24L01+)
- TX time per packet: ~2 ms (including protocol overhead)
- Frequency: Every 5 minutes if event detected, every 30 minutes for heartbeat

**Worst Case (Event Every 5 Minutes):**
```
TX_duty = (2 ms / 300 seconds) = 0.00067% of total time
I_tx_avg = 14,000 µA × 0.0000067 = 0.094 µA
```

**Typical Case (Heartbeat Only, Every 30 Minutes):**
```
TX_duty = (2 ms / 1800 seconds) = 0.00011%
I_tx_avg_typical = 14,000 µA × 0.0000011 = 0.015 µA
```

## 2.4 Total Power Budget Summary

### Worst-Case Scenario (Frequent Events)

| **Operating Mode** | **Duty Cycle** | **Current** | **Avg Contribution** |
|-------------------|----------------|-------------|----------------------|
| Deep Sleep | 90% | 2.0 µA | 1.8 µA |
| Audio Sampling | 8% | 1,150 µA | 92.0 µA |
| Processing | 1% | 1,050 µA | 10.5 µA |
| Wireless TX (frequent) | 0.0067% | 14,000 µA | 0.1 µA |
| **Total Average Current** | - | - | **104.4 µA** |

**Battery Life (Worst Case):**
```
Lifetime = 151 mAh / 104.4 µA = 1,446 hours = 60 days = 2 months
```

⚠️ **ISSUE IDENTIFIED:** Worst-case scenario does NOT meet 6-month target.

### Optimized Scenario (Required to Meet Spec)

**Solution: Reduce Sampling Duty Cycle**

To meet 6-month target with 24.5 µA average:
```
Required: Deep Sleep + Sampling + Processing + TX = 24.5 µA

1.8 µA (sleep) + X (sampling) + Y (processing) + 0.05 µA (TX) = 24.5 µA
X + Y ≈ 22.65 µA
```

**Revised Sampling Strategy:**
- Sample 1 second every 30 seconds = 3.3% duty cycle (vs 8%)
- Process immediately after sample = 0.33% duty cycle (vs 1%)

| **Operating Mode** | **Duty Cycle** | **Current** | **Avg Contribution** |
|-------------------|----------------|-------------|----------------------|
| Deep Sleep | 96.3% | 2.0 µA | 1.93 µA |
| Audio Sampling | 3.3% | 1,150 µA | 38.0 µA |
| Processing | 0.33% | 1,050 µA | 3.5 µA |
| Wireless TX | 0.011% | 14,000 µA | 0.015 µA |
| **Total Average Current** | - | - | **43.4 µA** |

**Battery Life (Optimized):**
```
Lifetime = 151 mAh / 43.4 µA = 3,479 hours = 145 days = 4.8 months
```

✅ **Close to 6-month target, achievable with further optimizations:**
- Lower ADC power (use 8 kHz sample rate, not 16 kHz)
- Optimize ASIC clock frequency (run at 12 MHz vs 25 MHz)
- Use motion-triggered sampling (only sample audio when pig moves)

### Ultra-Optimized Scenario (Motion-Triggered Sampling)

**Strategy:** Use accelerometer as wake source. Only sample audio when motion detected.

**Assumptions:**
- Accelerometer in low-power mode: 2 µA average (ADXL345 motion detect mode)
- Motion triggers audio sampling: 30% of the time (pig resting 70% of day)

| **Operating Mode** | **Duty Cycle** | **Current** | **Avg Contribution** |
|-------------------|----------------|-------------|----------------------|
| Deep Sleep + Accel | 96.3% | 4.0 µA | 3.85 µA |
| Audio Sampling (motion-gated) | 1.0% | 1,150 µA | 11.5 µA |
| Processing | 0.1% | 1,050 µA | 1.05 µA |
| Wireless TX | 0.011% | 14,000 µA | 0.015 µA |
| **Total Average Current** | - | - | **16.4 µA** |

**Battery Life (Motion-Gated):**
```
Lifetime = 151 mAh / 16.4 µA = 9,207 hours = 384 days = 12.8 months
```

✅ **Exceeds 6-month target with comfortable margin.**

## 2.5 Power Budget Recommendations

**Baseline Design:** Motion-triggered audio sampling (16.4 µA average)

**Critical Component Power Targets:**

| **Component** | **Target Power** | **Operating Mode** | **Vendor Examples** |
|---------------|------------------|--------------------|---------------------|
| Audio ADC | ≤500 µA @ 16 kHz | Active conversion | TI ADS1115, Analog Devices AD7124 |
| | ≤5 µA | Shutdown mode | Must have hardware shutdown |
| Digital MEMS Mic | ≤150 µA | Active | Knowles SPH0645LM4H, Infineon IM69D130 |
| | ≤1 µA | Standby | Hardware shutdown required |
| Wireless Module | ≤12 mA | TX @ 0 dBm | Nordic nRF24L01+, nRF52810 |
| | ≤15 µA | RX mode | For ACK reception |
| | ≤0.5 µA | Deep sleep | Must have power-down mode |
| Accelerometer | ≤150 µA | Normal mode | Analog Devices ADXL345 |
| | ≤2 µA | Motion detect mode | Must support interrupt-driven wake |
| Temperature Sensor | ≤50 µA | Active conversion | TI TMP117, Microchip MCP9808 |
| | ≤0.2 µA | Shutdown mode | I2C programmable |
| Boost Converter | ≤0.4 µA | Shutdown/bypass | TI TPS61220, TPS62740 |
| | >85% | Efficiency @ 1mA | At nominal load |

---

# 3. AUDIO ADC SPECIFICATION (PRIMARY ANALOG REQUIREMENT)

## 3.1 Functional Requirements

The audio ADC is the **most critical analog component** requiring custom design or careful selection. It must digitize acoustic signals from the pig's ear environment for cough detection and health monitoring.

### 3.1.1 Acoustic Environment Analysis

**Pig Vocalization Characteristics:**
| **Sound Type** | **Frequency Range** | **Typical SPL** | **Duration** | **Significance** |
|----------------|---------------------|-----------------|--------------|------------------|
| Normal vocalization | 200 Hz - 2 kHz | 70-90 dB SPL | 0.5-2 sec | Baseline behavior |
| Coughing | 500 Hz - 4 kHz | 80-100 dB SPL | 0.2-0.5 sec | **Primary detection target** |
| Wheezing | 400 Hz - 1.5 kHz | 60-75 dB SPL | Variable | Respiratory distress |
| Distress calls | 500 Hz - 3 kHz | 90-110 dB SPL | 1-5 sec | Pain/stress indicator |
| Environmental noise | 50 Hz - 8 kHz | 50-85 dB SPL | Continuous | Farm equipment, other animals |

**Design Implications:**
- Must capture fundamental frequencies: 200 Hz - 4 kHz (primary band)
- Harmonics extend to 8 kHz (secondary band for feature extraction)
- Dynamic range: 60-110 dB SPL = 50 dB dynamic range minimum

### 3.1.2 Sample Rate Selection

**Nyquist Requirement:**
- Maximum signal frequency: 8 kHz (to capture harmonics)
- Minimum sample rate: 16 kHz (2× Nyquist)

**Trade-off Analysis:**

| **Sample Rate** | **Pros** | **Cons** | **Power** | **Recommendation** |
|-----------------|----------|----------|-----------|---------------------|
| 8 kHz | Lowest power, sufficient for fundamentals | Miss harmonic content | 300 µA | Use for motion-free periods |
| 16 kHz | Good compromise, captures harmonics | Moderate power | 500 µA | **Baseline design** |
| 32 kHz | Excellent frequency resolution | High power, excessive for application | 800 µA | Not recommended |

**Selected Sample Rate:** 16 kHz (configurable to 8 kHz for power saving)

### 3.1.3 Resolution Selection

**Dynamic Range Requirement:**
- Loudest sound (distress): 110 dB SPL
- Quietest sound (wheezing): 60 dB SPL
- Required dynamic range: 50 dB

**Microphone Sensitivity:**
- Typical MEMS mic sensitivity: -26 dBFS (@ 94 dB SPL, 1 kHz)
- Output voltage swing: 0.1 - 1.0 V peak-to-peak (for 60-110 dB SPL)

**ADC Resolution Calculation:**

```
SNR_required = 50 dB (dynamic range)
SNR_ADC = 6.02 × N + 1.76 dB (theoretical)

Solving for N:
50 = 6.02 × N + 1.76
N = (50 - 1.76) / 6.02 = 8.0 bits (minimum)
```

**Practical Considerations:**
- 8-bit: Insufficient for noise floor separation
- 10-bit: Marginal, susceptible to quantization noise
- 12-bit: 74 dB SNR, provides 24 dB margin → **Adequate**
- 16-bit: 98 dB SNR, provides 48 dB margin → **Excellent, but higher power**

**Trade-off:**

| **Resolution** | **SNR** | **Power** | **Cost** | **Recommendation** |
|----------------|---------|-----------|----------|---------------------|
| 10-bit | 62 dB | Low | Low | Insufficient margin |
| 12-bit | 74 dB | Medium | Medium | **Minimum viable** |
| 14-bit | 86 dB | Medium | Medium | Good compromise |
| 16-bit | 98 dB | High | High | Over-specified |

**Selected Resolution:** 12-bit minimum, 14-bit preferred

## 3.2 Audio ADC Electrical Specification

### 3.2.1 Performance Requirements

| **Parameter** | **Min** | **Typ** | **Max** | **Unit** | **Notes** |
|---------------|---------|---------|---------|----------|-----------|
| **Resolution** | 12 | 14 | 16 | bits | Configurable preferred |
| **Sample Rate** | 8 | 16 | 32 | kHz | Configurable preferred |
| **SNR** | 70 | 80 | - | dB | At full scale |
| **THD** | - | - | -70 | dB | Total Harmonic Distortion |
| **ENOB** | 10.5 | 11.5 | - | bits | Effective Number of Bits |
| **Input Range** | - | 0.1 - 1.0 | - | V p-p | Single-ended or differential |
| **Input Impedance** | 10 | 100 | - | kΩ | High-Z for MEMS mic interface |
| **Common-Mode Rejection** | 60 | 80 | - | dB | For differential input |
| **Anti-Aliasing Filter** | - | - | - | - | Internal or external, fc = 8 kHz |
| **Latency** | - | - | 4 | ms | Group delay at 16 kHz |

### 3.2.2 Power Requirements

| **Operating Mode** | **Supply Voltage** | **Current** | **Power** | **Notes** |
|--------------------|--------------------|-------------|-----------|-----------|
| Active Conversion (16 kHz) | 1.8 V | ≤500 µA | ≤900 µW | **Critical requirement** |
| Active Conversion (8 kHz) | 1.8 V | ≤300 µA | ≤540 µW | Low-power mode |
| Standby (oscillator on) | 1.8 V | ≤50 µA | ≤90 µW | Quick wake |
| Shutdown | 1.8 V | ≤5 µA | ≤9 µW | Deep sleep |

**Power Budget Compliance:**
- Active mode @ 16 kHz: 500 µA fits within 1,150 µA sampling budget (see Section 2.3)
- Shutdown mode: 5 µA negligible during deep sleep

### 3.2.3 Digital Interface

**Output Format:** I2S (Inter-IC Sound) standard

| **Signal** | **Type** | **Timing** | **Notes** |
|------------|----------|------------|-----------|
| **BCLK** (Bit Clock) | Output | 512 kHz @ 16 kHz Fs | 32 bits/channel × 2 channels × Fs |
| **LRCLK** (Word Clock) | Output | 16 kHz | Left/Right channel select, = Fs |
| **SDATA** (Serial Data) | Output | MSB first | 16-bit words, left-justified |
| **MCLK** (Master Clock) | Input (optional) | 256×Fs - 512×Fs | If PLL used for clock generation |

**Configuration Interface:** I2C or SPI

| **Parameter** | **Specification** |
|---------------|-------------------|
| Interface Type | I2C (preferred) or SPI |
| I2C Address | 7-bit, user-selectable via pin strapping |
| I2C Speed | 100 kHz (standard) or 400 kHz (fast) |
| Configurable Parameters | Sample rate, resolution, gain, power mode |

### 3.2.4 Analog Front-End Requirements

**Microphone Interface:**

```
[MEMS Mic] ---> [DC Blocking Cap] ---> [PGA] ---> [Anti-Alias Filter] ---> [Sigma-Delta ADC]
                     (1 µF)              0-30 dB       Fc = 8 kHz            12-16 bit
```

| **Block** | **Specification** | **Rationale** |
|-----------|-------------------|---------------|
| **DC Blocking** | 1 µF capacitor, high-pass fc = 10 Hz | Remove DC bias from MEMS mic |
| **Programmable Gain Amplifier (PGA)** | 0 to +30 dB, 6 dB steps | Adapt to different mic sensitivities |
| **Anti-Aliasing Filter** | 4th order Butterworth, fc = 8 kHz | Prevent aliasing, flat passband |
| **ADC Architecture** | Sigma-Delta preferred | Best SNR/power tradeoff for audio |
| **Reference Voltage** | Internal bandgap (1.25V typical) | External reference increases power |

**Input Configuration:**
- Differential input preferred (better noise rejection)
- Single-ended acceptable if cost/pin count constrained
- AC-coupled to handle varying DC levels from MEMS microphones

## 3.3 Audio ADC Vendor Selection Guide

Since this is a highly power-constrained application, recommend:

### Option 1: Commercial Off-The-Shelf ADC (Preferred for Prototype)

| **Part Number** | **Vendor** | **Resolution** | **Sample Rate** | **Power @ 16kHz** | **Interface** | **Cost** | **Pros/Cons** |
|-----------------|------------|----------------|-----------------|-------------------|---------------|----------|---------------|
| **TI ADS1115** | Texas Instruments | 16-bit | 8-860 SPS | 150 µA (standby) | I2C | $4-6 | ✅ Very low power, ❌ Low sample rate (max 860 Hz, insufficient) |
| **Cirrus Logic CS5343** | Cirrus Logic | 24-bit | 50 kHz | 20 mW (11 mA) | I2S | $3-5 | ✅ Excellent audio quality, ❌ High power (exceeds budget) |
| **TI TLV320ADC3100** | Texas Instruments | 16-bit | 8-192 kHz | 3.5 mW | I2S + I2C | $3-4 | ✅ Low power, integrated PGA, ❌ Still ~2 mA (exceeds budget) |
| **Analog Devices AD7768** | Analog Devices | 24-bit | 256 kHz | 22 mW | SPI | $15-20 | ❌ High power, expensive |

⚠️ **Critical Finding:** No commercial ADC meets the ≤500 µA @ 16 kHz requirement.

### Option 2: Custom ASIC ADC (Recommended for Production)

**Why Custom Design is Necessary:**
1. Commercial audio ADCs target ≥1 mA power consumption
2. Ultra-low-power ADCs (like ADS1115) have insufficient sample rates
3. Cost reduction at volume production
4. Integration opportunity with digital ASIC in future (multi-die package)

**Custom ADC Specification Summary:**
- Architecture: Sigma-Delta modulator + digital decimation filter
- Process: 180nm or 130nm CMOS (mature process for analog)
- Supply: Single 1.8V rail (from boost converter)
- Package: QFN-24 or smaller
- Target cost: <$1 at 100k volume

**Development Path:**
1. **Phase 1 (Prototype):** Use closest commercial ADC (e.g., TLV320ADC3100) with higher power, validate system
2. **Phase 2 (Pre-Production):** Outsource custom ADC design to analog engineering house (subject of this datasheet)
3. **Phase 3 (Production):** Integrate ADC die with digital ASIC in SiP (System-in-Package)

---

# 4. COMPLETE SYSTEM BLOCK DIAGRAM

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          PIG EAR TAG SYSTEM                             │
└─────────────────────────────────────────────────────────────────────────┘

┌──────────────────┐        ┌──────────────────────────────────────────┐
│   CR2032 Battery │        │         POWER MANAGEMENT                 │
│   3V, 220 mAh    │───────▶│  TPS61220 Boost Converter 3V → 1.8V     │
└──────────────────┘        │  Quiescent: 0.4µA, Efficiency: 85%      │
                            └──────────────────────────────────────────┘
                                              │ 1.8V Rail
                                              ▼
        ┌────────────────────────────────────────────────────────────────┐
        │                    1.8V POWER RAIL                             │
        └────────────────────────────────────────────────────────────────┘
         │           │          │          │          │            │
         ▼           ▼          ▼          ▼          ▼            ▼
┌─────────────┐ ┌─────────┐ ┌──────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐
│ Digital     │ │ Audio   │ │ MEMS │ │ Temp    │ │ Accel   │ │Wireless │
│ ASIC        │ │ ADC     │ │ Mic  │ │ Sensor  │ │ Sensor  │ │ Module  │
│ (Caravel)   │ │**CUSTOM │ │      │ │ TMP117  │ │ ADXL345 │ │nRF24L01+│
└─────────────┘ └─────────┘ └──────┘ └─────────┘ └─────────┘ └─────────┘
      │              │           │         │           │            │
      │ I2S          │◀──────────┘         │           │            │
      │ (16kHz)      │                     │           │            │
      │              │                     │           │            │
      │ I2C          │                     │           │            │
      │◀─────────────┴─────────────────────┴───────────┘            │
      │                                                              │
      │ SPI                                                          │
      │◀─────────────────────────────────────────────────────────────┘
      │
      │ UART (Debug)
      │───────────────▶ [External Programmer/Debug]
      │
      │ GPIO (Control)
      │───────────────▶ [Power control, interrupts, status LEDs]
```

## 4.1 Signal Flow

### Audio Path:
```
Acoustic Signal → MEMS Mic → Audio ADC → I2S → Digital ASIC → Feature Extraction → SRAM Buffer
                                                                       │
                                                                       ▼
                                                              Event Detection
                                                                       │
                                                                       ▼
                                                              Wireless TX
```

### Temperature Path:
```
Body Heat → Temp Sensor → I2C → Digital ASIC → Threshold Check → Event Flag
```

### Activity Path:
```
Movement → Accelerometer → SPI/I2C → Digital ASIC → Motion Detection → Wake Trigger
```

### Power Management:
```
CR2032 → Boost Converter → 1.8V Rail → [All Components]
                  ▲
                  │
                  └─── Enable Control from Digital ASIC (per-peripheral power gating)
```

---

# 5. MECHANICAL & ENVIRONMENTAL SPECIFICATION

## 5.1 Mechanical Constraints

### 5.1.1 Pig Ear Tag Mounting

**Pig Ear Anatomy:**
- Ear thickness: 3-6 mm (varies by breed, age)
- Ear dimensions: ~100mm × 150mm (adult pig)
- Mounting location: Outer edge of ear (minimize discomfort)

**Tag Mounting Method:**
- **Male/Female snap-through design** (industry standard for livestock tags)
- Mounting hole punch: Ø 4-5 mm
- Retention force: >10 N (prevents accidental detachment)
- Tag must be symmetrical (no preferred ear orientation)

### 5.1.2 Form Factor

| **Dimension** | **Max** | **Target** | **Notes** |
|---------------|---------|------------|-----------|
| Length | 30 mm | 25 mm | Along ear edge |
| Width | 30 mm | 25 mm | Perpendicular to ear |
| Thickness | 10 mm | 8 mm | Total assembly with battery |
| Weight | 15 g | 10 g | Including battery, enclosure |
| Mounting hole | Ø 5 mm | Ø 4 mm | Center of tag |

**Volume Budget:**
- Battery (CR2032): Ø20mm × 3.2mm = 1.0 cm³
- PCB: 25mm × 25mm × 1.6mm = 1.0 cm³
- Components: ~0.5 cm³ (one-sided assembly)
- Enclosure: ~1.5 cm³ (walls, strain relief)
- **Total: ~4 cm³** (comfortable within 25×25×8mm envelope)

## 5.2 Environmental Requirements

### 5.2.1 Operating Conditions

| **Parameter** | **Min** | **Typ** | **Max** | **Unit** | **Rationale** |
|---------------|---------|---------|---------|----------|---------------|
| **Ambient Temperature** | -10 | +20 | +40 | °C | Farm environment (winter to summer) |
| **Pig Body Temperature** | +38 | +39 | +50 | °C | Normal body temp + fever scenarios |
| **Humidity** | 20 | 60 | 95 | % RH | Farm environment, non-condensing |
| **Atmospheric Pressure** | 80 | 101 | 110 | kPa | Sea level to 2000m altitude |

**Thermal Management:**
- Tag mounted on ear → good natural convection
- Power dissipation: <2 mW average → negligible self-heating
- No active cooling required

### 5.2.2 Environmental Protection

**IP Rating:** IP67 (Dust-tight, temporary immersion)

**Rationale:**
- Farm environment: Dust, dirt, feed particles
- Cleaning/disinfection: High-pressure water wash
- Animal behavior: Scratching, rubbing, exposure to water troughs

**Sealing Requirements:**
- Ultrasonic welded enclosure (no screws/gaskets)
- Potted electronics (conformal coating or full potting)
- Sealed microphone port (acoustic membrane, IP67 rated)
- Battery compartment: Hermetically sealed (prevent corrosion)

### 5.2.3 Mechanical Durability

| **Test** | **Requirement** | **Standard** |
|----------|-----------------|--------------|
| Drop Test | 1.5 m onto concrete, 10 drops | IEC 60068-2-32 |
| Vibration | 10-500 Hz, 2g RMS, 1 hour | IEC 60068-2-64 |
| Shock | 50g, 11ms half-sine, 3 axes | IEC 60068-2-27 |
| UV Exposure | 500 hours, no cracking | ASTM G154 |
| Water Immersion | 1m depth, 30 minutes | IEC 60529 (IP67) |

**Expected Failure Modes:**
- Battery depletion (6 months design life)
- Mechanical damage from aggressive pig behavior (<5% expected)
- Detachment from ear (<2% expected with proper installation)

## 5.3 Regulatory & Safety

### 5.3.1 Animal Welfare

- **ISO 11784/11785:** RFID animal identification (if RFID added)
- **FDA/USDA:** No toxic materials, food-safe enclosure
- **Biocompatibility:** ISO 10993-5 (cytotoxicity), ISO 10993-10 (irritation)

**Material Selection:**
- Enclosure: Polycarbonate (PC) or ABS, UV-stabilized
- No sharp edges (>0.5mm radius)
- No materials toxic to pigs (no lead, cadmium, mercury)

### 5.3.2 Wireless Regulations

**ISM Band Operation:**
- **Frequency:** 2.4 GHz ISM band (global allocation)
- **Transmit Power:** ≤0 dBm (1 mW EIRP)
- **Compliance:** FCC Part 15, ETSI EN 300 328, IC RSS-247

**Certification Path:**
- Wireless module: Use pre-certified module (nRF24L01+) → no additional RF testing
- Final product: FCC/CE self-declaration (if using certified module)

### 5.3.3 Battery Safety

**CR2032 Lithium Coin Cell:**
- **UN 38.3:** Lithium battery transport testing
- **IEC 62133:** Secondary cell safety (if rechargeable used instead)
- **UL 1642:** Lithium battery safety

**Design Requirements:**
- Over-discharge protection (disconnect at 2.0V)
- Short-circuit protection (PTC fuse or boost converter current limit)
- Battery holder: Prevent swallowing hazard (ultrasonic weld, no user access)

---

# 6. DIGITAL ASIC INTERFACE SPECIFICATION

The digital ASIC (Caravel user project) provides the following interfaces to external components. This section defines the electrical and protocol requirements for integration.

## 6.1 I2S Audio Interface (ADC Input)

**Function:** Receive digitized audio samples from the Audio ADC

| **Signal** | **Direction** | **Type** | **Voltage** | **Timing** | **Notes** |
|------------|---------------|----------|-------------|------------|-----------|
| **I2S_BCLK** | Input | CMOS | 1.8V | 512 kHz @ Fs=16kHz | Bit clock from ADC |
| **I2S_LRCLK** | Input | CMOS | 1.8V | 16 kHz | Word clock from ADC |
| **I2S_SDATA** | Input | CMOS | 1.8V | Setup: 10ns, Hold: 10ns | Serial audio data |

**Protocol:**
- Format: I2S standard (MSB first, 2's complement)
- Word length: 16 bits (configurable to 12/14 bit)
- Channels: Mono (left channel only, right ignored)
- Sampling: Rising edge of BCLK
- DMA: Direct to SRAM buffer (4KB circular buffer)

**Pad Assignment:**
- I2S_BCLK → `mprj_io[8]` (input)
- I2S_LRCLK → `mprj_io[9]` (input)
- I2S_SDATA → `mprj_io[10]` (input)

## 6.2 I2C Master Interface (Temperature Sensor)

**Function:** Configure and read temperature sensor

| **Signal** | **Direction** | **Type** | **Voltage** | **Frequency** | **Notes** |
|------------|---------------|----------|-------------|---------------|-----------|
| **I2C_SCL** | Bidirectional | Open-drain | 1.8V | 100 kHz | Clock, 4.7kΩ pull-up |
| **I2C_SDA** | Bidirectional | Open-drain | 1.8V | - | Data, 4.7kΩ pull-up |

**Supported Devices:**
- TMP117 (0x48-0x4B)
- MCP9808 (0x18-0x1F)
- Generic I2C temperature sensors

**Protocol:**
- Standard mode: 100 kHz (preferred for low power)
- Fast mode: 400 kHz (optional)
- 7-bit addressing
- Multi-byte read/write support

**Pad Assignment:**
- I2C_SCL → `mprj_io[11]` (bidirectional, open-drain)
- I2C_SDA → `mprj_io[12]` (bidirectional, open-drain)

## 6.3 SPI Master Interface (Wireless + Accelerometer)

**Function:** Communicate with wireless module and accelerometer (shared bus)

| **Signal** | **Direction** | **Type** | **Voltage** | **Frequency** | **Notes** |
|------------|---------------|----------|-------------|---------------|-----------|
| **SPI_SCLK** | Output | CMOS | 1.8V | 4 MHz | Clock |
| **SPI_MOSI** | Output | CMOS | 1.8V | - | Master Out Slave In |
| **SPI_MISO** | Input | CMOS | 1.8V | - | Master In Slave Out |
| **SPI_CS_RF** | Output | CMOS | 1.8V | - | Chip select for wireless |
| **SPI_CS_ACCEL** | Output | CMOS | 1.8V | - | Chip select for accel |

**Protocol:**
- Mode: SPI Mode 0 (CPOL=0, CPHA=0)
- Bit order: MSB first
- Max frequency: 8 MHz (hardware capable), 4 MHz typical (power optimized)

**Pad Assignment:**
- SPI_SCLK → `mprj_io[13]` (output)
- SPI_MOSI → `mprj_io[14]` (output)
- SPI_MISO → `mprj_io[15]` (input)
- SPI_CS_RF → `mprj_io[16]` (output)
- SPI_CS_ACCEL → `mprj_io[17]` (output)

## 6.4 UART Interface (Debug/Configuration)

**Function:** Debug output, firmware update, configuration

| **Signal** | **Direction** | **Type** | **Voltage** | **Baud Rate** | **Notes** |
|------------|---------------|----------|-------------|---------------|-----------|
| **UART_TX** | Output | CMOS | 1.8V | 115200 bps | Transmit data |
| **UART_RX** | Input | CMOS | 1.8V | 115200 bps | Receive data |

**Protocol:**
- Format: 8N1 (8 data bits, no parity, 1 stop bit)
- Baud rate: 115200 bps (configurable down to 9600 bps)
- Flow control: None (2-wire only)

**Pad Assignment:**
- UART_TX → `mprj_io[6]` (output)
- UART_RX → `mprj_io[7]` (input)

## 6.5 GPIO Interface (Control Signals)

**Function:** General-purpose I/O for power control, interrupts, status

| **Signal** | **Direction** | **Function** | **Voltage** | **Notes** |
|------------|---------------|--------------|-------------|-----------|
| **GPIO[0]** | Output | RF_PWR_EN | 1.8V | Enable power to RF module |
| **GPIO[1]** | Output | ADC_PWR_EN | 1.8V | Enable power to Audio ADC |
| **GPIO[2]** | Output | ACCEL_PWR_EN | 1.8V | Enable power to Accelerometer |
| **GPIO[3]** | Input | ACCEL_INT | 1.8V | Motion detect interrupt (rising edge) |
| **GPIO[4]** | Input | ADC_DRDY | 1.8V | ADC data ready (falling edge) |
| **GPIO[5]** | Output | STATUS_LED | 1.8V | Optional status LED (for debug) |

**Pad Assignment:**
- GPIO[0] (RF_PWR_EN) → `mprj_io[18]` (output)
- GPIO[1] (ADC_PWR_EN) → `mprj_io[19]` (output)
- GPIO[2] (ACCEL_PWR_EN) → `mprj_io[20]` (output)
- GPIO[3] (ACCEL_INT) → `mprj_io[21]` (input, interrupt)
- GPIO[4] (ADC_DRDY) → `mprj_io[22]` (input, interrupt)
- GPIO[5] (STATUS_LED) → `mprj_io[23]` (output, optional)

## 6.6 Power Domain

**Supply Voltage:** 1.8V ± 5% (1.71V - 1.89V)

**Power Consumption:**
| **Mode** | **Typical** | **Max** | **Notes** |
|----------|-------------|---------|-----------|
| Deep Sleep | 2 µA | 5 µA | RTC + retention |
| Active (25 MHz) | 800 µA | 1.2 mA | Full speed processing |
| Active (12 MHz) | 400 µA | 600 µA | Low-power processing |

**Power Sequencing:**
1. Apply 1.8V to ASIC
2. Release reset (active-low, external RC or power supervisor)
3. ASIC firmware enables peripherals via GPIO power enables
4. Begin operation

**Reset:**
- External reset required (active-low)
- RC time constant: >100ms (ensure stable power-on)
- Alternative: Dedicated reset supervisor IC (TPS3840 or similar)

---

# 7. WIRELESS SPECIFICATION

## 7.1 Wireless Protocol Requirements

**Purpose:** Transmit health event data from ear tag to local hub (gateway)

**Protocol Selection Criteria:**

| **Protocol** | **Range** | **Power (TX)** | **Data Rate** | **Latency** | **Cost** | **Verdict** |
|--------------|-----------|----------------|---------------|-------------|----------|-------------|
| **BLE 5.x** | 50-100m | 5-10 mA | 1 Mbps | <10ms | Medium | ✅ Good option |
| **Zigbee** | 10-100m | 25-35 mA | 250 kbps | 15-30ms | Medium | ❌ Higher power |
| **LoRa** | 1-10 km | 30-120 mA | 0.3-50 kbps | 100ms-1s | High | ❌ Overkill range, high power |
| **nRF24L01+** | 50-100m | 11 mA | 1-2 Mbps | <5ms | Low | ✅ **Selected** |
| **WiFi** | 50-100m | 100-300 mA | 1-54 Mbps | <10ms | Medium | ❌ Excessive power |

**Selected:** nRF24L01+ (2.4 GHz proprietary protocol)

**Rationale:**
- Lowest power consumption for required range
- Sufficient data rate (1 Mbps >> required ~1 kbps)
- Low cost (<$1 at volume)
- Mature ecosystem, proven in battery applications
- Simple SPI interface

## 7.2 Wireless Link Budget

**Assumptions:**
- Transmit power: 0 dBm (1 mW)
- Frequency: 2.4 GHz
- Data rate: 1 Mbps
- Modulation: GFSK

**Path Loss (Friis Equation):**
```
PL(dB) = 20 log(d) + 20 log(f) + 32.44
Where:
  d = distance in km
  f = frequency in MHz

For d = 100m = 0.1 km, f = 2400 MHz:
PL = 20 log(0.1) + 20 log(2400) + 32.44
PL = -20 + 67.6 + 32.44
PL = 80 dB
```

**Link Budget:**

| **Parameter** | **Value** | **Unit** |
|---------------|-----------|----------|
| TX Power | 0 | dBm |
| TX Antenna Gain | -5 | dBi (ceramic chip antenna, small size) |
| Path Loss (100m) | -80 | dB |
| Fade Margin | -10 | dB (farm environment, animals, metal) |
| RX Antenna Gain | 2 | dBi (gateway with external antenna) |
| **RX Power** | **-93** | **dBm** |
| RX Sensitivity (1 Mbps) | -94 | dBm (nRF24L01+ datasheet) |
| **Link Margin** | **+1** | **dB** |

**Analysis:**
- 1 dB margin is tight but acceptable for low-criticality monitoring
- Practical range: 50-80m in barn environment (metal structures, animals)
- Gateway placement: 1 per 50-100 pigs (typical barn density)

**Improvement Options (if needed):**
- Increase TX power to +4 dBm (+4 dB margin) → increases TX current to 11.5 mA
- Use better antenna (PIFA or external wire) → +3-5 dBi gain
- Lower data rate to 250 kbps → +6 dB sensitivity improvement

## 7.3 Wireless Data Protocol

**Packet Structure:**

```
┌─────────┬─────────┬──────────┬─────────────────┬───────┐
│ Preamble│ Address │  Payload │      CRC        │ Stop  │
│ (1 byte)│ (5 byte)│ (32 byte)│    (2 byte)     │(1 byte│
└─────────┴─────────┴──────────┴─────────────────┴───────┘
   Auto      Config     Data         Auto           Auto
```

**Payload Format (32 bytes):**

| **Field** | **Size** | **Type** | **Description** |
|-----------|----------|----------|-----------------|
| Device ID | 4 bytes | uint32 | Unique pig/tag identifier |
| Timestamp | 4 bytes | uint32 | Seconds since deployment (Unix epoch) |
| Event Type | 1 byte | uint8 | Bit flags: [7]=Cough, [6]=Fever, [5]=Lethargy, [4]=Reserved... |
| Temperature | 2 bytes | int16 | Temperature in 0.01°C units (e.g., 3900 = 39.00°C) |
| Audio Energy | 2 bytes | uint16 | RMS audio energy (arbitrary units) |
| Motion Level | 1 byte | uint8 | Activity level (0-255) |
| Battery Voltage | 2 bytes | uint16 | Battery voltage in mV (e.g., 3000 = 3.0V) |
| Audio Features | 8 bytes | uint64 | Compressed audio features (ZCR, spectral centroid, etc.) |
| Reserved | 4 bytes | - | Future use |
| Checksum | 4 bytes | uint32 | CRC32 of payload |

**Transmission Schedule:**

| **Condition** | **Frequency** | **Rationale** |
|---------------|---------------|---------------|
| **Event Detected** (cough, fever) | Immediate | Time-critical health alert |
| **Heartbeat** (no events) | Every 30 minutes | Confirm tag is alive, battery status |
| **Data Burst** (multiple events) | Aggregate 5 events, then TX | Reduce TX overhead |

**Acknowledgment:**
- nRF24L01+ auto-ACK enabled
- Retry: 3 attempts, 250µs delay between retries
- If all retries fail: Store event in SRAM, retry at next heartbeat

## 7.4 Gateway (Hub) Requirements

**Function:** Receive data from multiple ear tags, forward to central server

**Specification (for gateway, not part of ear tag):**

| **Parameter** | **Requirement** |
|---------------|-----------------|
| Number of Tags | 50-100 per gateway |
| Simultaneous Reception | 1 at a time (time-division) |
| Processing | Raspberry Pi or equivalent (ARM Cortex-A) |
| Connectivity | WiFi or Ethernet to farm network |
| Power | Mains powered (120/240VAC) |
| Antenna | External 2.4 GHz, 5 dBi gain |
| Software | Node-RED or custom Python |

---

# 8. BILL OF MATERIALS (ESTIMATED)

## 8.1 Component Cost Breakdown (100k Volume)

| **Component** | **Part Number (Example)** | **Qty** | **Unit Cost** | **Total** | **Notes** |
|---------------|---------------------------|---------|---------------|-----------|-----------|
| **Digital ASIC** | Caravel + User Macro | 1 | $3.50 | $3.50 | SKY130, 5mm² die, packaged |
| **Audio ADC** | Custom (TBD) | 1 | $1.00 | $1.00 | **Custom design required** |
| **MEMS Microphone** | Knowles SPH0645LM4H | 1 | $1.20 | $1.20 | Digital I2S output |
| **Wireless Module** | nRF24L01+ SMD | 1 | $0.80 | $0.80 | Pre-certified module |
| **Temperature Sensor** | TI TMP117 | 1 | $0.50 | $0.50 | High-accuracy, low-power |
| **Accelerometer** | ADXL345 | 1 | $1.50 | $1.50 | 3-axis, motion detection |
| **Boost Converter** | TI TPS61220 | 1 | $0.60 | $0.60 | 3V → 1.8V, 0.4µA quiescent |
| **CR2032 Battery** | Panasonic CR2032 | 1 | $0.30 | $0.30 | 220 mAh |
| **PCB** | 4-layer, 25×25mm | 1 | $0.50 | $0.50 | FR4, ENIG finish |
| **Passives** | Resistors, caps, inductors | ~20 | $0.05 | $1.00 | 0402 size |
| **Antenna** | Ceramic chip 2.4GHz | 1 | $0.30 | $0.30 | Johanson Technology |
| **Enclosure** | PC molded housing | 1 | $0.80 | $0.80 | Injection molded |
| **Assembly** | SMT pick-and-place | 1 | $1.50 | $1.50 | Contract manufacturer |
| **Testing** | Functional test, program | 1 | $0.50 | $0.50 | Automated test fixture |
| **Total Component Cost** | - | - | - | **$14.50** | Target: <$15 per unit |

**Cost Reduction Opportunities:**
- Integrate ADC with digital ASIC → save $1-2
- Use smaller accelerometer (LIS3DH) → save $0.50
- Optimize PCB to 2-layer → save $0.20
- Target: <$12 at 500k volume

## 8.2 Complete BOM (Production)

See separate document: **[BOM.md](BOM.md)** for detailed part numbers, suppliers, and alternates.

---

# 9. AUDIO ADC DESIGN REQUIREMENTS (FOR OUTSOURCING)

## 9.1 Scope of Work for Engineering House

**Deliverables:**
1. Complete Audio ADC ASIC design (GDS II)
2. Verification test benches (Verilog-A, SPICE)
3. Post-layout simulation results
4. Characterization test plan
5. Datasheet and integration guide
6. 100 packaged samples for system validation

**Timeline:** 9-12 months (typical for analog ASIC)

**Budget:** $150k - $300k NRE (non-recurring engineering)

## 9.2 Detailed ADC Specification for RFQ

### 9.2.1 Architecture

**Recommended Architecture:** Continuous-Time Sigma-Delta ADC

**Rationale:**
- Best power efficiency for audio applications
- Built-in anti-aliasing (no large external filter)
- High SNR with low oversampling ratio (OSR)

**Block Diagram:**

```
               ┌─────────────────────────────────────────────────┐
               │    Audio ADC (Custom ASIC)                      │
               └─────────────────────────────────────────────────┘
                                                                   
 Analog Input   ┌──────┐  ┌──────────────┐  ┌──────────┐  Digital
 ──────────────▶│ PGA  │─▶│ CT Sigma-Delta│─▶│Decimation│─▶ I2S Out
 (MEMS Mic)     │0-30dB│  │  Modulator    │  │  Filter  │  (16-bit)
                └──────┘  └──────────────┘  └──────────┘
                    ▲             ▲                 ▲
                    │             │                 │
                ┌───┴─────────────┴─────────────────┴────┐
                │   Digital Control & Config (I2C)       │
                └────────────────────────────────────────┘
```

**Key Specifications:**

| **Block** | **Parameter** | **Specification** |
|-----------|---------------|-------------------|
| **PGA** | Gain Range | 0 to +30 dB |
| | Gain Steps | 6 dB (6 settings) |
| | Input Impedance | >50 kΩ |
| | Bandwidth | 10 Hz - 10 kHz |
| **Sigma-Delta Modulator** | Order | 3rd or 4th order |
| | OSR (Oversampling Ratio) | 64-128 |
| | Clock Frequency | 1-2 MHz |
| | Quantizer | 1-bit or 2-bit |
| **Decimation Filter** | Type | Sinc³ or Sinc⁴ + FIR |
| | Passband | DC - 7 kHz (flat ±0.5dB) |
| | Stopband | >10 kHz (>60 dB attenuation) |
| | Group Delay | <2 ms |
| **Clock** | Type | Internal RC oscillator (I2C trimmable) |
| | Frequency | 1-2 MHz ±5% |
| | Power | <20 µA |

### 9.2.2 Power Breakdown Target

| **Block** | **Power Budget** | **Typical Current @ 1.8V** |
|-----------|------------------|-----------------------------|
| PGA | 50 µW | 28 µA |
| Sigma-Delta Modulator | 200 µW | 111 µA |
| Decimation Filter | 150 µW | 83 µA |
| Clock Oscillator | 30 µW | 17 µA |
| I2C Interface | 20 µW | 11 µA |
| Digital Control | 50 µW | 28 µA |
| **Total** | **500 µW** | **278 µA @ 1.8V** |

**Critical:** Must meet ≤500 µW total at 16 kHz sample rate.

### 9.2.3 Performance Targets

| **Parameter** | **Min** | **Typ** | **Max** | **Unit** | **Test Condition** |
|---------------|---------|---------|---------|----------|---------------------|
| Resolution | 12 | 14 | 16 | bits | At max gain |
| Sample Rate | 8 | 16 | 32 | kHz | Configurable |
| SNR | 70 | 75 | - | dB | 1 kHz, -1 dBFS |
| THD | - | - | -70 | dB | 1 kHz, -1 dBFS |
| THD+N | - | - | -68 | dB | 1 kHz, -1 dBFS |
| ENOB | 10.5 | 11.5 | - | bits | Derived from SNR |
| SFDR | 70 | 80 | - | dB | Spurious-Free Dynamic Range |
| Input Range (full-scale) | 0.5 | 1.0 | 1.5 | Vpp | At 0 dB gain |
| Input Impedance | 50 | 100 | - | kΩ | Differential |
| CMRR | 60 | 70 | - | dB | Common-Mode Rejection Ratio |
| PSRR | 60 | 70 | - | dB | Power Supply Rejection Ratio |
| Crosstalk | - | - | -80 | dB | Inter-channel (if stereo) |

### 9.2.4 Electrical Characteristics

**Power Supply:**

| **Parameter** | **Min** | **Typ** | **Max** | **Unit** |
|---------------|---------|---------|---------|----------|
| Supply Voltage (AVDD, DVDD) | 1.71 | 1.8 | 1.89 | V |
| Supply Current (active @ 16kHz) | - | 300 | 500 | µA |
| Supply Current (standby) | - | 10 | 50 | µA |
| Supply Current (shutdown) | - | 1 | 5 | µA |
| Power-Up Time | - | 10 | 50 | ms |

**Digital I/O:**

| **Parameter** | **Min** | **Typ** | **Max** | **Unit** |
|---------------|---------|---------|---------|----------|
| I/O Voltage (VDDIO) | 1.71 | 1.8 | 3.6 | V |
| Input High (VIH) | 0.7×VDDIO | - | - | V |
| Input Low (VIL) | - | - | 0.3×VDDIO | V |
| Output High (VOH) | 0.8×VDDIO | - | - | V |
| Output Low (VOL) | - | - | 0.2×VDDIO | V |

**Reference Voltage:**

| **Parameter** | **Min** | **Typ** | **Max** | **Unit** |
|---------------|---------|---------|---------|----------|
| Internal Reference | 1.20 | 1.25 | 1.30 | V |
| Reference Tempco | - | 50 | - | ppm/°C |
| External Reference (option) | 1.0 | 1.25 | 1.5 | V |

### 9.2.5 Timing Specifications

**I2S Output:**

| **Parameter** | **Min** | **Typ** | **Max** | **Unit** | **Notes** |
|---------------|---------|---------|---------|----------|-----------|
| BCLK Frequency | - | 512 | - | kHz | At Fs=16kHz |
| LRCLK Frequency | - | 16 | - | kHz | = Sample rate |
| SDATA Setup Time | 10 | - | - | ns | Before BCLK rising edge |
| SDATA Hold Time | 10 | - | - | ns | After BCLK rising edge |
| BCLK Duty Cycle | 45 | 50 | 55 | % | |

**I2C Control Interface:**

| **Parameter** | **Min** | **Typ** | **Max** | **Unit** |
|---------------|---------|---------|---------|----------|
| SCL Frequency | - | 100 | 400 | kHz |
| SDA Setup Time | 100 | - | - | ns |
| SDA Hold Time | 300 | - | - | ns |
| I2C Address | - | 0x48 | - | 7-bit |

### 9.2.6 Package and Pinout

**Recommended Package:** QFN-24 (4mm × 4mm)

**Pin Count Justification:**

| **Function** | **Pins** |
|--------------|----------|
| Power (AVDD, AVSS, DVDD, DVSS) | 4 |
| Analog Input (INP, INM) | 2 |
| Reference (VREF, optional) | 1 |
| I2S (BCLK, LRCLK, SDATA) | 3 |
| I2C (SCL, SDA) | 2 |
| Control (PDN, TEST) | 2 |
| NC / Thermal Pad | 10 |
| **Total** | **24** |

**Pinout (Preliminary):**

```
              QFN-24 (Top View)
              4mm × 4mm

         1   2   3   4   5   6
       ┌───┬───┬───┬───┬───┬───┐
    24 │   │   │   │   │   │   │ 7
       ├───┤                 ├───┤
    23 │   │                 │   │ 8
       ├───┤                 ├───┤
    22 │   │   Thermal Pad   │   │ 9
       ├───┤                 ├───┤
    21 │   │                 │   │ 10
       ├───┤                 ├───┤
    20 │   │                 │   │ 11
       ├───┴───┴───┴───┴───┴───┤
    19  18  17  16  15  14  13  12

Pin Assignment:
1: AVDD          7: NC            13: DVSS         19: NC
2: INP           8: NC            14: BCLK         20: TEST
3: INM           9: NC            15: LRCLK        21: SCL
4: AVSS          10: NC           16: SDATA        22: SDA
5: VREF          11: NC           17: DVDD         23: PDN
6: NC            12: NC           18: DVSS         24: NC

Center Pad: AVSS / Thermal Ground
```

### 9.2.7 Process Technology

**Recommended Process:** 180nm CMOS or 130nm CMOS

**Rationale:**
- Mature process → lower NRE, higher yield
- Sufficient speed for 1-2 MHz sigma-delta modulator
- Good analog performance (mismatch, noise)
- Lower leakage than deep-submicron processes

**Foundry Options:**
- TSMC 180nm (CL018G)
- GlobalFoundries 180nm (BCDLite)
- Tower 180nm
- UMC 180nm

**Alternative:** 130nm for slightly lower power and smaller die size

### 9.2.8 Design Deliverables

1. **RTL & Analog Design Files**
   - Verilog HDL (digital decimation filter, control logic)
   - SPICE netlists (analog modulator, PGA)
   - Layout (GDS II)

2. **Verification**
   - Verilog-A behavioral models
   - Post-layout simulation results (typical, fast, slow corners)
   - Monte Carlo analysis (100 runs minimum)

3. **Documentation**
   - Design specification document
   - Datasheet (final)
   - Application note (integration with digital ASIC)
   - Register map (I2C configuration)

4. **Test Plan**
   - Bench characterization procedure
   - Automated test equipment (ATE) program
   - Pass/fail criteria

5. **Physical Samples**
   - 100 packaged units (QFN-24)
   - 10 bare die for failure analysis

### 9.2.9 Acceptance Criteria

**Must Pass All:**
1. ✅ SNR ≥ 70 dB (at 1 kHz, -1 dBFS)
2. ✅ THD ≤ -70 dB (at 1 kHz, -1 dBFS)
3. ✅ Power ≤ 500 µA @ 1.8V, 16 kHz sample rate
4. ✅ Shutdown current ≤ 5 µA
5. ✅ Functional I2S output (verified with oscilloscope)
6. ✅ I2C configuration (all registers accessible)
7. ✅ Temperature range: -10°C to +50°C (full spec)
8. ✅ Extended temp: -20°C to +70°C (reduced spec acceptable)
9. ✅ Package integrity (no cracks, delamination after thermal cycling)
10. ✅ Yield ≥ 80% (on first silicon spin)

---

# 10. FIRMWARE & SOFTWARE REQUIREMENTS

## 10.1 Embedded Firmware (ASIC)

**Platform:** RISC-V soft core (Caravel management SoC) or bare-metal on user project

**Language:** C/C++ (GCC RISC-V toolchain)

**Functionality:**

### 10.1.1 Boot Sequence
1. Power-on reset (wait 100ms for supplies to stabilize)
2. Initialize peripherals (I2S, I2C, SPI, UART)
3. Configure Audio ADC (sample rate, gain)
4. Configure temperature sensor (continuous mode, alert threshold)
5. Configure accelerometer (motion detection, threshold)
6. Configure wireless module (channel, power level, address)
7. Enter main loop

### 10.1.2 Main Loop (Interrupt-Driven)

```c
while (1) {
    // Deep sleep (wait for interrupt)
    enter_deep_sleep();
    
    // Wake sources:
    // - RTC timer (periodic sampling)
    // - Accelerometer INT (motion detected)
    // - ADC DRDY (audio sample ready)
    
    if (wake_source == RTC_TIMER) {
        // Periodic sampling (every 30 seconds)
        enable_audio_adc();
        start_audio_capture(duration = 1 second);
        wait_for_capture_complete();
        disable_audio_adc();
        
        // Process audio features
        audio_features = extract_features(audio_buffer);
        
        // Check for cough signature
        if (is_cough_detected(audio_features)) {
            event_flag |= EVENT_COUGH;
            event_queue_push(EVENT_COUGH, timestamp, audio_features);
        }
        
        // Read temperature
        temperature = read_temperature_sensor();
        if (temperature > FEVER_THRESHOLD) {
            event_flag |= EVENT_FEVER;
        }
        
        // Transmit if event detected
        if (event_flag != 0) {
            transmit_event_packet(event_flag, temperature, audio_features);
        }
    }
    
    else if (wake_source == ACCEL_INT) {
        // Motion-triggered sampling (low latency)
        // (Similar to RTC_TIMER, but immediate)
    }
    
    // Heartbeat transmission (every 30 minutes)
    if (time_since_last_tx > HEARTBEAT_INTERVAL) {
        transmit_heartbeat(battery_voltage, temperature);
    }
}
```

### 10.1.3 Audio Feature Extraction

**Implemented Features:**
1. **RMS Energy:** Time-domain energy (sqrt of mean square)
2. **Zero-Crossing Rate (ZCR):** Number of sign changes per second
3. **Spectral Centroid:** Center of mass of spectrum (FFT not feasible, approximate with filter banks)
4. **Peak Detection:** Identify sudden amplitude spikes
5. **Duration:** Length of audio event

**Cough Detection Algorithm (Simple Threshold-Based):**

```c
bool is_cough_detected(audio_features_t features) {
    // Cough characteristics:
    // - High energy burst (500 Hz - 4 kHz)
    // - Short duration (0.2 - 0.5 seconds)
    // - High ZCR (noisy, aperiodic)
    
    if (features.rms_energy > COUGH_ENERGY_THRESHOLD &&
        features.duration_ms > 200 && features.duration_ms < 500 &&
        features.zcr > COUGH_ZCR_THRESHOLD) {
        return true;
    }
    return false;
}
```

**Advanced Option (Future):**
- Use TinyML (TensorFlow Lite Micro) for ML-based classification
- Requires training dataset of pig coughs
- May need additional ASIC resources (MAC units, larger SRAM)

### 10.1.4 Wireless Protocol Handler

**Packet Transmission:**
```c
void transmit_event_packet(uint8_t event_flags, int16_t temperature, audio_features_t features) {
    packet_t packet;
    
    // Fill packet
    packet.device_id = DEVICE_ID;  // Unique tag ID
    packet.timestamp = rtc_get_timestamp();
    packet.event_type = event_flags;
    packet.temperature = temperature;
    packet.audio_energy = features.rms_energy;
    packet.motion_level = read_accelerometer();
    packet.battery_voltage = read_battery_voltage();
    
    // Compress audio features (8 bytes)
    compress_audio_features(&packet.audio_features, features);
    
    // Calculate checksum
    packet.checksum = crc32(&packet, sizeof(packet) - 4);
    
    // Transmit via nRF24L01+
    nrf24_send_packet(&packet, sizeof(packet));
    
    // Wait for ACK (with timeout)
    if (!nrf24_wait_for_ack(timeout = 10ms)) {
        // Retry or store for later
        event_queue_push_retry(&packet);
    }
}
```

## 10.2 Gateway Firmware

**Platform:** Raspberry Pi or similar (ARM Cortex-A)

**Language:** Python or Node-RED

**Functionality:**
1. Receive packets from multiple ear tags (nRF24L01+ connected via SPI)
2. Parse and validate packets
3. Store data locally (SQLite database)
4. Forward to cloud server (MQTT or HTTP)
5. Trigger alerts (email, SMS) for critical events

**Not part of this design (reference only).**

---

# 11. TESTING & VALIDATION PLAN

## 11.1 Component-Level Testing

### 11.1.1 Audio ADC Characterization

**Test Equipment:**
- Audio Precision APx555 or equivalent
- Low-distortion signal generator (<-100 dB THD)
- Oscilloscope (100 MHz, 4 channels)
- Power analyzer (Keysight N6705B)

**Test Procedure:**

| **Test** | **Stimulus** | **Measurement** | **Pass Criteria** |
|----------|--------------|-----------------|-------------------|
| SNR | 1 kHz sine, -1 dBFS | FFT analysis, 20 Hz - 20 kHz | ≥70 dB |
| THD | 1 kHz sine, -1 dBFS | Harmonic distortion up to 5th | ≤-70 dB |
| Frequency Response | Sweep 10 Hz - 10 kHz | Magnitude, phase | ±0.5 dB, 200 Hz - 8 kHz |
| Dynamic Range | -60 dBFS to 0 dBFS | SNR at each level | >60 dB |
| Power Consumption | All operating modes | Current @ 1.8V | ≤500 µA active, ≤5 µA shutdown |
| I2S Timing | Capture BCLK, LRCLK, SDATA | Setup/hold times | >10 ns |
| Temperature Sweep | -20°C to +70°C | SNR, THD at each temp | Within spec |

### 11.1.2 Digital ASIC Testing

**Test Equipment:**
- FPGA prototyping board (for pre-silicon validation)
- Logic analyzer (Saleae Logic Pro 16)
- Caravel test harness

**Test Cases:**
1. I2S receiver: Inject test patterns, verify DMA to SRAM
2. SPI master: Read/write to test peripheral (loopback)
3. I2C master: Configure test sensor (EEPROM)
4. UART: Echo test, baud rate accuracy
5. Interrupt controller: Verify all 16 sources
6. Power modes: Measure current in deep sleep, active modes

## 11.2 System-Level Integration Testing

### 11.2.1 Benchtop System Test

**Setup:**
- Assembled ear tag PCB (all components)
- Function generator (audio stimulus)
- Power supply (3V, current measurement)
- Wireless gateway (nRF24L01+ receiver)
- PC with serial terminal

**Test Sequence:**

| **Test** | **Procedure** | **Expected Result** |
|----------|---------------|---------------------|
| Power-On | Apply 3V from CR2032 simulator | Tag boots, UART outputs "System Ready" |
| Audio Capture | Inject 1 kHz sine @ -10 dBFS | I2S data captured, displayed via UART |
| Cough Simulation | Play recorded pig cough (speaker) | Tag detects cough, transmits event packet |
| Temperature | Heat tag to 45°C (heat gun) | Fever event triggered |
| Motion | Shake accelerometer | Motion interrupt triggers sampling |
| Wireless Range | Move tag away from gateway | Verify reception up to 50m (indoor) |
| Battery Life | Measure current over 1 hour | Average ≤50 µA |

### 11.2.2 Environmental Testing

**Test Standards:**
- IEC 60068-2-1: Cold (-10°C, 72 hours)
- IEC 60068-2-2: Dry Heat (+50°C, 72 hours)
- IEC 60068-2-30: Humidity (95% RH, +40°C, 48 hours)
- IEC 60068-2-32: Free Fall (1.5m, 10 drops)

**Pass Criteria:**
- ✅ Functional after each test
- ✅ No cracks in enclosure
- ✅ Wireless link maintained
- ✅ Battery voltage >2.5V after 1 week of operation

## 11.3 Field Trials

### 11.3.1 Pilot Deployment

**Location:** Commercial pig farm (50-100 pigs)

**Duration:** 4 weeks

**Procedure:**
1. Deploy 20 ear tags on selected pigs (mixed age, health status)
2. Deploy 1 gateway per barn
3. Monitor data collection (events, heartbeats)
4. Manually observe pigs daily (veterinarian notes)
5. Compare automated detections with manual observations

**Success Criteria:**
- ✅ Tag retention rate >95% (no more than 1 lost tag)
- ✅ Battery voltage >2.7V after 4 weeks (projected 6-month life)
- ✅ Cough detection accuracy >80% (true positive rate)
- ✅ False positive rate <10%
- ✅ Wireless connectivity >95% uptime

### 11.3.2 Validation Study

**Location:** Research farm with controlled health monitoring

**Duration:** 6 months (full battery life)

**Procedure:**
1. Deploy 100 ear tags across multiple pens
2. Intentionally expose subset of pigs to respiratory pathogens (with veterinary approval)
3. Collect daily health assessments by veterinarians (ground truth)
4. Analyze correlation between automated detections and clinical diagnosis

**Validation Metrics:**
- Sensitivity (true positive rate): >90%
- Specificity (true negative rate): >85%
- Positive predictive value: >75%
- Time to detection: <24 hours from symptom onset

---

# 12. RISK ANALYSIS & MITIGATION

## 12.1 Technical Risks

| **Risk** | **Probability** | **Impact** | **Mitigation** |
|----------|-----------------|------------|----------------|
| **Custom ADC fails to meet power target** | Medium | High | Prototype with commercial ADC first; allocate contingency budget for ADC redesign |
| **Cough detection accuracy insufficient** | Medium | Medium | Use larger dataset for algorithm tuning; consider ML approach; lower specificity acceptable if low false positives |
| **Battery life <6 months** | Low | High | Conservative power budgeting; motion-gated sampling; field trials validate before production |
| **Wireless range inadequate in barn** | Low | Medium | Link budget analysis; use higher TX power (+4 dBm) if needed; add repeaters |
| **Tag detachment from ear** | Medium | Low | Use industry-standard mounting; test with various pig breeds; veterinary consultation |
| **Environmental damage (water, dust)** | Low | Medium | IP67 enclosure; conformal coating; field trials in realistic conditions |

## 12.2 Schedule Risks

| **Milestone** | **Duration** | **Risk** | **Mitigation** |
|---------------|--------------|----------|----------------|
| Custom ADC design | 9-12 months | Analog design complexity | Engage experienced analog design house; fixed-price contract |
| Digital ASIC tape-out | 6 months | Caravel integration issues | Use proven IP cores; cocotb verification; leverage caravel-cocotb |
| System integration | 3 months | Interface bugs | Early prototyping with FPGA; extensive benchtop testing |
| Field trials | 6 months | Unexpected failures | Plan for multiple trial rounds; allocate buffer time |

## 12.3 Cost Risks

| **Risk** | **Impact** | **Mitigation** |
|----------|------------|----------------|
| ADC NRE exceeds budget | $100k-200k overrun | Negotiate fixed-price contract; use milestone-based payments |
| Component shortages | Delays, higher costs | Multi-source BOM; design with alternates; inventory buffer |
| Field trial failures require redesign | 6-12 month delay | Conservative design margins; early validation with prototypes |

---

# 13. PROJECT TIMELINE & MILESTONES

## 13.1 Development Phases

```
Phase 1: Design & Prototyping (Months 1-12)
├─ Month 1-3:   System architecture, specifications (COMPLETE - this document)
├─ Month 4-6:   Digital ASIC design (Caravel user project RTL)
├─ Month 7-12:  Custom ADC design (outsourced to analog house)
└─ Month 10-12: PCB design, prototype assembly

Phase 2: Verification & Testing (Months 13-18)
├─ Month 13-14: Component characterization (ADC, ASIC)
├─ Month 15-16: System integration testing (benchtop)
├─ Month 17-18: Environmental testing (temp, humidity, shock)
└─ Deliverable:  100 prototype tags ready for field trials

Phase 3: Field Trials (Months 19-24)
├─ Month 19-20: Pilot deployment (20 tags, 1 farm)
├─ Month 21-24: Validation study (100 tags, research farm)
└─ Deliverable:  Validated system, algorithm refinement

Phase 4: Production Ramp (Months 25-30)
├─ Month 25-26: Design for Manufacturing (DFM), yield optimization
├─ Month 27-28: Production tooling, test fixtures
├─ Month 29-30: Initial production run (10,000 tags)
└─ Deliverable:  Commercial product launch
```

## 13.2 Critical Path

**Longest pole:** Custom ADC design (9-12 months)

**Parallelization:**
- Digital ASIC design can proceed concurrently (use placeholder ADC model)
- Firmware development can start once digital ASIC RTL is frozen
- Gateway software can be developed independently

**De-Risk Strategy:**
- Order long-lead components early (CR2032, MEMS mics)
- Prototype with commercial ADC (even if power-hungry) to validate system
- Tape-out digital ASIC before ADC is complete (use I2S test vectors)

---

# 14. STATEMENT OF WORK (FOR ANALOG ENGINEERING HOUSE)

## 14.1 Scope

The Contractor shall design, verify, and deliver a custom Audio Analog-to-Digital Converter (ADC) ASIC meeting the specifications in Section 9 of this document.

## 14.2 Deliverables

| **Deliverable** | **Quantity** | **Due Date** | **Acceptance Criteria** |
|-----------------|--------------|--------------|-------------------------|
| **Preliminary Design Review (PDR)** | 1 report | Month 3 | Architecture approved, power analysis, risk assessment |
| **Critical Design Review (CDR)** | 1 report | Month 6 | Schematic complete, simulations pass spec |
| **Tape-Out Package** | GDS II, LEF, LIB | Month 9 | DRC/LVS clean, timing closure |
| **Test Plan** | 1 document | Month 9 | Characterization procedure, ATE program |
| **Packaged Samples** | 100 units | Month 12 | QFN-24, tested, passing yield >80% |
| **Datasheet** | 1 document | Month 12 | Final specifications, application notes |
| **Characterization Report** | 1 report | Month 12 | Measured SNR, THD, power, temp sweep |

## 14.3 Payment Terms

| **Milestone** | **Percentage** | **Amount** | **Trigger** |
|---------------|----------------|------------|-------------|
| Contract Award | 20% | $40k | Signed contract |
| PDR Approval | 20% | $40k | Customer accepts preliminary design |
| CDR Approval | 20% | $40k | Schematics and simulations approved |
| Tape-Out | 20% | $40k | GDS II submitted to foundry |
| First Silicon | 10% | $20k | Packaged samples received |
| Final Acceptance | 10% | $20k | Characterization report approved |
| **Total NRE** | **100%** | **$200k** | |

## 14.4 Technical Requirements (Summary)

**Must Meet:**
- Resolution: 12-bit minimum (14-bit preferred)
- Sample Rate: 8-16 kHz (configurable)
- SNR: ≥70 dB
- THD: ≤-70 dB
- Power: ≤500 µA @ 1.8V, 16 kHz (CRITICAL)
- Shutdown: ≤5 µA
- Interface: I2S output, I2C configuration
- Package: QFN-24, 4mm × 4mm
- Process: 180nm or 130nm CMOS
- Temperature: -10°C to +50°C (full spec)

**Acceptance Test:**
- Customer shall witness final characterization
- Pass/fail based on datasheet specifications
- Yield ≥80% on first silicon (no respin assumed in contract)

## 14.5 Customer-Furnished Items

- Digital ASIC test interface (I2S receiver, I2C master)
- Audio Precision test equipment (or equivalent)
- Test board PCB design (Contractor provides schematic, Customer fabricates)

## 14.6 Assumptions

- Foundry: TSMC 180nm or approved equivalent
- Tape-out schedule: No more than 1 MPW (multi-project wafer) per year
- Packaging: QFN-24, standard assembly house
- No military/aerospace qualification required
- Commercial temperature range: -10°C to +50°C (not -40°C to +125°C)

---

# 15. APPENDICES

## 15.1 Glossary

| **Term** | **Definition** |
|----------|----------------|
| **ADC** | Analog-to-Digital Converter |
| **ASIC** | Application-Specific Integrated Circuit |
| **BLE** | Bluetooth Low Energy |
| **CMRR** | Common-Mode Rejection Ratio |
| **ENOB** | Effective Number of Bits |
| **GFSK** | Gaussian Frequency Shift Keying |
| **I2C** | Inter-Integrated Circuit (serial bus) |
| **I2S** | Inter-IC Sound (audio serial bus) |
| **IP** | Intellectual Property (design block) or Ingress Protection (environmental rating) |
| **ISM** | Industrial, Scientific, Medical (radio band) |
| **MEMS** | Micro-Electro-Mechanical System |
| **OSR** | Oversampling Ratio |
| **PGA** | Programmable Gain Amplifier |
| **PSRR** | Power Supply Rejection Ratio |
| **RMS** | Root Mean Square |
| **SFDR** | Spurious-Free Dynamic Range |
| **SNR** | Signal-to-Noise Ratio |
| **SPI** | Serial Peripheral Interface |
| **THD** | Total Harmonic Distortion |
| **UART** | Universal Asynchronous Receiver-Transmitter |
| **ZCR** | Zero-Crossing Rate |

## 15.2 Reference Documents

1. **nRF24L01+ Datasheet** (Nordic Semiconductor)
2. **TPS61220 Datasheet** (Texas Instruments)
3. **Knowles MEMS Microphone Selection Guide**
4. **IEC 60529 - IP Code Standard**
5. **FCC Part 15 - Unlicensed Transmitters**
6. **ISO 11784/11785 - RFID Animal Identification**
7. **Caravel User Project Template** (Efabless)

## 15.3 Revision History

| **Rev** | **Date** | **Author** | **Changes** |
|---------|----------|------------|-------------|
| 0.1 | 2026-01-15 | Systems Team | Initial draft |
| 0.9 | 2026-02-01 | Systems Team | Internal review, power budget refinement |
| 1.0 | 2026-02-04 | Systems Team | **Released for RFQ to engineering houses** |

---

# END OF DOCUMENT

**Prepared by:** NativeChips Systems Engineering  
**Date:** February 4, 2026  
**Status:** Released for External Development (RFQ)  
**Confidentiality:** Proprietary - Requires NDA

---

**Next Steps:**
1. Issue RFQ to 3-5 analog design houses (ADC design)
2. Select contractor based on cost, schedule, and technical approach
3. Proceed with digital ASIC implementation (Caravel user project)
4. Parallel track: Prototype with commercial ADC for system validation

**Contact:**
For technical questions or clarifications, contact:  
NativeChips Engineering  
Email: engineering@nativechips.ai
