# Hardware V2.0

## Overview

This page discusses the improvements that would be made to the Drive Train subsystem PCB if a second hardware revision were to be produced. The discussion is informed by lessons learned during schematic design, PCB layout, component bring-up, and firmware integration. The schematic below is used as a reference throughout.

---

## Identified Issues and Proposed Improvements

### 1. Fully Populate and Integrate the Second Motor Driver

The original schematic was designed with two IFX9201SGAUMA1 H-bridge ICs (U3 and U4) to drive two motors independently, supporting true differential steering for the rover. However in the Version 1.0 implementation only one driver (U4, Motor B) was populated and tested due to time constraints during the individual subsystem phase. Version 2.0 would fully populate and integrate both motor drivers, realising the complete two-motor differential drive system the schematic was designed for.

Having both motor drivers operational would bring significant capability improvements to the drivetrain subsystem. Differential steering allows the rover to turn in place by running the two motors in opposite directions, navigate curves smoothly by varying the relative speed of each wheel, and correct for terrain-induced drift by independently adjusting each motor's duty cycle based on encoder feedback. With only one motor the rover can only drive in a straight line and cannot steer, which severely limits its usefulness as an exploration platform.

From a firmware perspective the second motor driver would use the already-allocated GPIO connections (GPIO1 for Motor A PWM, GPIO2 for Motor A DIR, GPIO42 for Motor A DIS, and GPIO38 for Motor A CSN) that are present in the schematic but were unused in V1.0. The SPI bus is already shared between both ICs with independent CSN lines, so the firmware additions would be minimal. The UART protocol would be extended to accept a second speed byte in the 0x01 motor command payload, allowing Christo's HMI node to independently command both motors with a single packet.

The power budget implications of running two motors simultaneously were already accounted for in the V1.0 power budget analysis. At typical operating load both motors together draw approximately 1.9A, which remains within the 3A wall supply rating with margin. The 9V power rail traces were already sized at 2.5mm width to handle the combined current, so no PCB layout changes are required to support dual motor operation. The second set of VS decoupling capacitors (C7 and C8 in the schematic) are already placed and footprinted adjacent to U3, requiring only physical population during assembly.

### 2. VSO Pin Not Connected in Original Design

During hardware bring-up it was discovered that the VSO pin (pin 2) of the IFX9201SG must be connected to 3.3V to power the internal SO output buffer. In the original schematic VSO was left as a no-connect, which prevented any SPI readback from the diagnosis register and returned 0x00 for all SPI reads. This was a significant debugging setback that cost several hours of troubleshooting time.

In Version 2.0 VSO would be connected directly to the 3.3V rail with a 100nF decoupling capacitor to GND placed immediately adjacent to the pin. This is a one-component fix that would have eliminated the entire SPI communication debugging session and is clearly documented in the IFX9201SG datasheet section 4.1.

### 3. CSN Pullup Resistor Missing from Schematic

The original schematic did not include a hardware pullup resistor on the CSN pin (pin 9) of the IFX9201SG. CSN is active low with an internal pullup, meaning it defaults to deselected when floating. However during ESP32 boot there is a window where GPIOs are not yet driven, and any transient low on CSN during this period puts the IC into SPI mode where it ignores all PWM and DIR inputs.

This caused the motor to kick on unexpectedly at startup before the firmware had initialised the CSN pin high. The firmware workaround was to drive DIS high immediately on boot before any other initialisation, but this is not a robust solution. Version 2.0 would add a 10K ohm pullup resistor from CSN to 3.3V on the schematic, ensuring the IC remains in a known safe state during the ESP32 boot sequence regardless of GPIO initialisation order.

### 4. Inductor Current Rating Undersized

The inductor selected for the TPS563201DDCT buck converter (LQM21PN2R2NGCD, 2.2uH) has a saturation current rating of only 800mA. The TPS563201DDCT is rated for 3A output current and the design requires at least 4A saturation current in the inductor to prevent core saturation under full load conditions. Operating the inductor beyond its saturation current causes inductance to collapse, which significantly increases output ripple and can cause the regulator to become unstable.

Version 2.0 would replace this inductor with a properly rated alternative such as the Bourns SRR1260-2R2Y (2.2uH, 4.4A saturation current, low DCR) or the Wurth 744043002 (2.2uH, 3.8A). The replacement part would use the same 0805 footprint so no PCB layout changes would be required, making this a straightforward bill of materials change.

### 5. Resettable Fuse Rating

The MF-MSMF260/12X-2 fuse ordered for this design has a hold current of 2.6A. As shown in the power budget, the motor draws up to 2.0A under typical operating conditions with a 25% safety margin bringing the required hold current to 2.5A. This leaves only 100mA of margin between the expected operating current and the fuse trip threshold, meaning the fuse could nuisance trip under moderate motor load conditions such as climbing a small incline or accelerating quickly.

Version 2.0 would replace the fuse with the MF-MSMF300/12 (3A hold, 12V, 1812 package) which was the originally specified part. This provides 500mA of margin above the 2.5A safety-margined requirement, reducing the risk of nuisance trips during normal rover operation while still providing protection against genuine fault conditions such as a motor stall or short circuit.

### 6. EN Pin Reset Circuit Improvement

The current schematic uses a simple 10K pullup resistor and 100nF capacitor on the EN pin for power-on reset. While this is functional it does not provide the clean reset behaviour that a dedicated supervisor IC would give. During bench testing it was observed that the ESP32 occasionally failed to boot cleanly when power was applied quickly, requiring a manual reset button press.

Version 2.0 would add a dedicated reset supervisor IC such as the Microchip MCP101 or Texas Instruments TPS3839, which monitors the 3.3V supply and holds EN low until the supply is fully stable. This would eliminate the occasional boot failure and is particularly important for a rover application where the board may be power cycled frequently in the field without access to a reset button.

### 7. Motor Connector Type

The current design uses standard 2.54mm pitch pin headers for the motor and encoder connections. While these are easy to source and solder, they are not mechanically secure and can vibrate loose during rover operation over rough terrain. During testing the motor connector was found to be intermittently loose, causing erratic motor behaviour.

Version 2.0 would replace the motor power header with a locking connector such as the Molex KK 254 series or JST-XH series, which includes a positive latch that prevents accidental disconnection. The encoder connector would similarly be replaced with a JST-ZH 1.5mm or similar fine-pitch locking connector that matches the Pololu 4843 motor's native connector more closely, eliminating the need for an adapter harness.

### 8. Test Points

The original schematic includes a single test point (TP2) on the 3.3V output rail of the voltage regulator. During bring-up it was necessary to probe multiple nodes including the SW switching node, VBST, VFB, EN, VS, and motor output pins to diagnose faults. The absence of dedicated test points on these nets made probing difficult on the densely populated PCB.

Version 2.0 would add test points to the following nets as a minimum:

- GND (adjacent to every power IC)
- 9V input rail
- 3.3V output rail (already present)
- SW node (TPS563201 switching node)
- VFB (voltage regulator feedback)
- IFX9201SG OUT1 and OUT2
- IFX9201SG VS
- UART TX and RX lines

These test points would be placed on the PCB edge where possible to allow easy access with standard oscilloscope probes without disturbing the rest of the circuit.

---

## Summary of Proposed Changes

| # | Issue | Change | Impact |
|---|---|---|---|
| 1 | Second H-bridge not populated | Fully populate U3 and integrate dual motor control | Enables differential steering and full rover mobility |
| 2 | VSO not connected | Connect VSO to 3.3V with decoupling cap | Enables SPI diagnosis readback |
| 3 | CSN pullup missing | Add 10K pullup to 3.3V on CSN | Prevents startup motor kick |
| 4 | Inductor undersized | Replace with 4A rated 2.2uH inductor | Prevents regulator instability at full load |
| 5 | Fuse hold current too low | Replace with MF-MSMF300/12 (3A hold) | Eliminates nuisance trips under motor load |
| 6 | EN reset circuit basic | Add dedicated reset supervisor IC | Eliminates occasional boot failures |
| 7 | Motor connectors not locking | Replace with locking JST or Molex connectors | Prevents vibration-induced disconnection |
| 8 | Insufficient test points | Add test points on all critical nets | Speeds up debugging significantly |

---

## Conclusion

The Version 1.0 hardware functioned correctly for its core purpose of receiving UART motor commands and driving a single DC gearmotor with SPI diagnostic feedback. However the bring-up process revealed several design oversights that added significant debugging time and required firmware workarounds to compensate for hardware deficiencies. The single most impactful improvement for Version 2.0 is the full population and integration of the second IFX9201SG motor driver, which was already designed into the schematic but not implemented in V1.0. This change alone would transform the subsystem from a single-wheel drive into a true differential drive system capable of steering, turning in place, and navigating terrain independently. Combined with the VSO connection fix, the CSN pullup resistor, and the inductor current rating correction, Version 2.0 would be a significantly more capable, reliable, and debuggable board that fully realises the original design intent of the drivetrain subsystem.