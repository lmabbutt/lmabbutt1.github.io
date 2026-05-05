# Power Budget

## Overview

This power budget confirms that the selected power source and voltage regulator can sufficiently supply all major components in the Drive Train subsystem under worst-case operating conditions.

---

## Section A: Major Components and Maximum Current

| Component | Part Number | Supply Voltage | Absolute Max Current | Datasheet Reference |
|---|---|---|---|---|
| Microcontroller Module | ESP32-S3-WROOM-1-N4 | 3.3V | 500mA | ESP32-S3 datasheet, power consumption table |
| H-Bridge Motor Driver | IFX9201SGAUMA1 | 9V | 13mA (IC quiescent only) | IFX9201SG datasheet Table 4, P_6.0.1 |
| DC Gearmotor with Encoder | Pololu 4843 | 9V | 3750mA (stall current at 9V) | Pololu product page, scaled from 5A at 12V |

**Note:** The IFX9201SG motor driver acts as a switch for the motor. The IC itself draws only 13mA of quiescent supply current. The motor stall current of 3750mA flows through the driver but is consumed by the motor and is therefore listed under the motor entry above.

---

## Section B: Power Rail Assignment

### Rail 1: 3.3V Logic Rail

| Component | Max Current |
|---|---|
| ESP32-S3-WROOM-1-N4 | 500mA |
| **Subtotal** | **500mA** |
| **25% Safety Margin** | **125mA** |
| **Total Required** | **625mA** |

### Rail 2: 9V Motor Power Rail

| Component | Max Current |
|---|---|
| IFX9201SGAUMA1 (IC quiescent) | 13mA |
| Pololu 4843 (stall at 9V) | 3750mA |
| **Subtotal** | **3763mA** |
| **25% Safety Margin** | **941mA** |
| **Total Required** | **4704mA** |

---

## Section C: Voltage Regulator Selection

### 3.3V Rail Regulator: Texas Instruments TPS563201DDCT

| Parameter | Value |
|---|---|
| Input Voltage | 9V |
| Output Voltage | 3.3V |
| Maximum Output Current | 3000mA |
| Required Output Current | 625mA |
| Remaining Headroom | 2375mA |
| Efficiency | ~90% |
| Input Current Required | ~210mA (625mA / 0.9 efficiency) |

**Regulator or Source Choice: TPS563201DDCT**

#### Alternative Regulators Considered

| | Option 1 | Option 2 |
|---|---|---|
| **Part** | TPS563201DDCT | LM1117-3.3 (Linear) |
| **Type** | Switching (Buck) | Linear |
| **Max Output Current** | 3A | 800mA |
| **Efficiency** | ~90% | ~37% (9V to 3.3V) |
| **Pros** | High efficiency, low heat, adequate current headroom | Simple design, low noise, low cost |
| **Cons** | Requires inductor and careful PCB layout | Wastes 5.7V as heat, only 800mA max |
| **Selected** | Yes | No |

**Rationale:** The TPS563201DDCT was selected for its switching efficiency. A linear regulator dropping 9V to 3.3V dissipates 5.7V x load current as heat. At 500mA ESP32 load this is 2.85W of wasted power. The switching regulator wastes less than 10% of this, significantly extending battery life in a rover application.

### 9V Rail Regulator: None (Direct from Power Source)

The 9V rail is supplied directly from the external power source via the barrel jack. No regulator is required on this rail.

---

## Section D: External Power Source

**Selected Power Source: 9V DC Wall Supply (5A minimum)**

| Rail | Required Current (with 25% margin) |
|---|---|
| 9V motor rail (direct) | 4704mA |
| 9V input to TPS563201DDCT | 210mA |
| **Total Required from 9V Source** | **4914mA (~4.9A)** |

| Parameter | Value |
|---|---|
| Required Supply Voltage | 9V |
| Required Supply Current | 5A minimum |
| Barrel Plug | 5.5mm OD / 2.1mm ID |
| Connector | Wurth 694106301002 |
| Protection | Bourns MF-MSMF300/12 resettable fuse on input rail |

**Total Remaining Current Available:** 9V 5A supply provides 5000mA. Total required is 4914mA. Remaining margin is 86mA. A 9V 6A supply is recommended for additional headroom.

---

## Section E: Battery Life Estimate

If operating from a battery rather than a wall supply, worst-case battery life is calculated assuming all subsystems running simultaneously at maximum current.

### Worst Case (Motor at Stall)

| Battery | Capacity | Worst Case Life |
|---|---|---|
| 9V 2000mAh LiPo | 2000mAh | 2000 / 4914 = 0.41 hours (24 minutes) |
| 9V 5000mAh LiPo | 5000mAh | 5000 / 4914 = 1.02 hours (61 minutes) |
| 9V 10000mAh LiPo | 10000mAh | 10000 / 4914 = 2.04 hours (122 minutes) |

### Realistic Case (Motor at 25% of Stall)

At normal operating load the motor draws approximately 940mA, giving a realistic total system draw of approximately 1200mA.

| Battery | Capacity | Realistic Life |
|---|---|---|
| 9V 2000mAh LiPo | 2000mAh | 2000 / 1200 = 1.67 hours (100 minutes) |
| 9V 5000mAh LiPo | 5000mAh | 5000 / 1200 = 4.17 hours (250 minutes) |
| 9V 10000mAh LiPo | 10000mAh | 10000 / 1200 = 8.33 hours (500 minutes) |

**Recommended battery for portable operation:** 9V 5000mAh LiPo, providing over 1 hour of worst-case runtime and over 4 hours of typical runtime.