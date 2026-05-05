# Power Budget

**Team Number:** 305
**Project Name:** Mars Exploration Rover
**Team Member:** Liam Mabbutt
**Version:** 1

## Overview

This power budget confirms that the selected power source and voltage regulator can sufficiently supply all major components in the Drive Train subsystem under worst-case operating conditions.

---

## Section A: Major Components and Maximum Current

| Component | Part Number | Supply Voltage | # | Absolute Max Current (mA) | Total Current (mA) |
|---|---|---|---|---|---|
| ESP32 Microcontroller | ESP32-S3-WROOM-1-N4 | 3.3V | 1 | 500 | 500 |
| H-Bridge Motor Driver | IFX9201SGAUMA1 | 9V | 1 | 13 | 13 |
| DC Gearmotor | Pololu 4843 | 9V | 1 | 2000 | 2000 |

**Note:** The IFX9201SG motor driver acts as a switch for the motor. The IC itself draws only 13mA of quiescent supply current. The motor current flows through the driver but is consumed by the motor and is listed under the motor entry above. The motor current figure of 2000mA represents the expected operating current rather than the absolute stall current of 3750mA at 9V, reflecting realistic operating conditions for the rover.

---

## Section B: Power Rail Assignment

### Rail 1: +9V Power Rail

| Component | Part Number | Supply Voltage | # | Absolute Max Current (mA) | Total Current (mA) |
|---|---|---|---|---|---|
| H-Bridge Motor Driver | IFX9201SGAUMA1 | 9V | 1 | 13 | 13 |
| DC Gearmotor | Pololu 4843 | 9V | 1 | 2000 | 2000 |
| **Subtotal** | | | | | **2013** |
| **Safety Margin** | | | | | **25%** |
| **Total Current Required** | | | | | **2516.25 mA** |

**Regulator or Source Choice:** +9V Wall Supply (Barrel Jack) -- 9V, 3000mA maximum
**Total Remaining Current Available on +9V Rail: 483.75 mA**

### Rail 2: +3.3V Power Rail

| Component | Part Number | Supply Voltage | # | Absolute Max Current (mA) | Total Current (mA) |
|---|---|---|---|---|---|
| ESP32 Microcontroller | ESP32-S3-WROOM-1-N4 | 3.3V | 1 | 500 | 500 |
| **Subtotal** | | | | | **500** |
| **Safety Margin** | | | | | **25%** |
| **Total Current Required** | | | | | **625 mA** |

**Regulator or Source Choice:** TPS563201DDCT -- 3.3V, 3000mA maximum
**Total Remaining Current Available on +3.3V Rail: 2375 mA**

---

## Section C: Voltage Regulator Selection

### 3.3V Rail: Texas Instruments TPS563201DDCT

| Parameter | Value |
|---|---|
| Input Voltage Range | 4.5V to 17V |
| Output Voltage | 3.3V |
| Maximum Output Current | 3000mA |
| Required Output Current | 625mA |
| Remaining Headroom | 2375mA |
| Efficiency | ~90% |

#### Alternative Regulators Considered

| | Option 1 (Selected) | Option 2 |
|---|---|---|
| **Part** | TPS563201DDCT | LM1117-3.3 (Linear) |
| **Type** | Switching Buck | Linear |
| **Max Output Current** | 3000mA | 800mA |
| **Input Voltage Range** | 4.5V to 17V | Up to 15V |
| **Efficiency** | ~90% | ~37% (9V to 3.3V) |
| **Pros** | High efficiency, low heat dissipation, 3A output, wide input range | Simple design, low noise output, low component count, low cost |
| **Cons** | Requires external inductor and careful PCB layout, switching noise | Wastes 5.7V as heat at 9V input, only 800mA maximum output |
| **Selected** | Yes | No |

**Rationale:** The TPS563201DDCT was selected for its switching efficiency. A linear regulator dropping 9V to 3.3V would dissipate 5.7V x load current as heat. At 500mA ESP32 load this is 2.85W of wasted power. The switching regulator wastes less than 10% of this, significantly extending battery life in a rover application. The 3A output rating also provides 2375mA of headroom above the 625mA required, leaving capacity for future expansion.

### 9V Rail: No Regulator Required

The 9V rail is supplied directly from the external wall supply via the barrel jack connector. No regulator is required on this rail.

---

## Section D: External Power Source

**Power Source: Plug-in Wall Supply -- 9V DC, 3000mA**

| Component | Part Number | Supply Voltage Range | Output Voltage | Max Current (mA) | Total Current (mA) |
|---|---|---|---|---|---|
| Plug-in Wall Supply | (full part number) | 9V DC | 9V | 3000 | 3000 |

**Power Rails Connected to External Power Source:**

| Rail | Regulator | Input Voltage Range | Output Voltage | Max Current (mA) | Total Current (mA) |
|---|---|---|---|---|---|
| 3.3V Logic Rail | TPS563201DDCT | 4.5V to 17V | 3.3V | 3000 | 3000 |

**Total Remaining Current Available on External Power Source: 0 mA**

**Note:** The wall supply is running at its rated maximum with 0mA remaining. A higher rated supply of 5A or greater is strongly recommended to provide margin for motor startup surges and to avoid voltage sag under peak load. A 9V 5A supply would leave 2483.75mA of headroom on the 9V rail.

---


### Realistic Case (Motor at 25% of Maximum)

At normal operating load the motor draws approximately 500mA, giving a realistic total system draw of approximately 730mA.

| Battery | Capacity | Realistic Life |
|---|---|---|
| 9V 2000mAh LiPo | 2000mAh | 2000 / 730 = 2.74 hours (164 minutes) |
| 9V 5000mAh LiPo | 5000mAh | 5000 / 730 = 6.85 hours (411 minutes) |
| 9V 10000mAh LiPo | 10000mAh | 10000 / 730 = 13.70 hours (822 minutes) |

---

## Section F: Power Budget Analysis and Conclusions

### How the Power Budget Was Used

The power budget was constructed by first identifying all active components in the subsystem and sourcing their maximum current draw directly from the relevant datasheets. The ESP32-S3-WROOM-1-N4 maximum current of 500mA was taken from the ESP32-S3 datasheet power consumption table. The IFX9201SG quiescent supply current of 13mA was taken from Table 4 parameter P_6.0.1 of the IFX9201SG datasheet. The Pololu 4843 motor current of 2000mA was estimated from the product specifications scaled to 9V operation.

Components were then assigned to two power rails based on their operating voltage. The 9V rail powers the motor and H-bridge directly from the wall supply, avoiding the power loss that would occur if the motor current were first passed through a regulator. The 3.3V rail powers the ESP32 exclusively via the TPS563201DDCT switching regulator. A 25% safety margin was applied to each rail subtotal to account for transient current spikes, component variation, and future expansion.

### Key Conclusions

**Power source sizing is critical.** The selected 3A wall supply leaves 0mA of remaining headroom after accounting for the 9V rail load. This is insufficient for reliable operation, as motor startup surges can briefly exceed the rated operating current. A minimum 5A supply at 9V is recommended for the final design to provide adequate margin.

**The switching regulator choice is well justified.** The TPS563201DDCT leaves 2375mA of headroom on the 3.3V rail above the 625mA required. This means the regulator is operating at approximately 21% of its rated capacity, which reduces heat dissipation and improves long-term reliability.

**Motor current dominates the power budget.** The Pololu 4843 motor accounts for over 79% of total system current draw at the 9V rail level. This confirms that motor driver and wiring selection are the most critical power design decisions in this subsystem. The 2.5mm wide traces used on the 9V motor power rail in the PCB design were directly informed by this finding.

**The 9V single-rail architecture simplifies the design.** By choosing components that operate at either 9V or 3.3V and using a single external supply, the design avoids the complexity and cost of multiple wall supplies or multi-output regulators. The TPS563201DDCT efficiently bridges the two voltage domains with minimal component overhead.

### Power Budget Spreadsheet

[Download Power Budget Spreadsheet](PB.xlsx)