# Reflection

## Review of Module Success

### Requirements Met

Looking back at the product requirements defined at the start of the project, the following requirements were successfully accomplished:

**Microcontroller Power Regulation:** The TPS563201DDCT switching regulator was successfully designed, assembled, and verified to produce a stable 3.3V output from the 9V input rail. After resolving an EN pin wiring issue discovered during bring-up, the regulator functioned correctly and powered the ESP32 reliably throughout all testing.

**Microcontroller Selection:** The ESP32-S3-WROOM-1-N4 was successfully integrated as the central controller. MicroPython firmware was loaded via the native USB Serial/JTAG interface without requiring an external USB-UART chip, confirming the microcontroller selection was appropriate for the application.

**Motor Type Compatibility:** The Pololu 4843 brushed DC gearmotor was successfully driven bidirectionally at variable speed, satisfying the requirement for brushed DC motor support. The motor operated correctly at 9V, producing adequate torque and speed for the rover's intended use.

**Motor Driver Requirement:** The IFX9201SGAUMA1 H-bridge was successfully integrated with full SPI communication, hardware overcurrent protection, and thermal shutdown capability active. The IC's built-in protections operated correctly during testing including during motor stall conditions.

**Motor Speed Control:** Open-loop PWM speed control was successfully implemented using the ESP32 LEDC peripheral at 20kHz, satisfying the threshold requirement. The encoder connections were wired and footprinted on the PCB, leaving the path open for closed-loop PID control in a future firmware update.

**Safety Interlock Implementation:** Both the hardware DIS pin interlock and the software SPI SEN bit interlock were successfully implemented. The DIS pin was driven high on boot before any other initialisation to prevent uncontrolled motor startup, satisfying both the threshold and target safety requirements.

**Communication with Main Controller:** The UART daisy-chain bus protocol was successfully implemented and validated. The node correctly receives motor speed commands from Christo's HMI node, sends ACK responses, and forwards non-addressed packets downstream, satisfying the fully synchronised control interface target requirement.

### Requirements Not Fully Met

**Closed-Loop Speed Control:** While the encoder was wired and footprinted, closed-loop PID speed control was not implemented in firmware during this project cycle. The motor speed control remained open-loop, meeting only the threshold requirement rather than the target.

**Motor Configuration Support (4WD stretch goal):** The schematic was designed for two motors and the PCB was fabricated with both H-bridge footprints populated, however only one motor driver was fully brought up and tested. The 4WD expandability stretch goal was not realised in the V1.0 hardware.

**Steering Control Method:** As a direct consequence of only one motor driver being operational, differential steering was not achieved. The rover could only drive in a single direction without the ability to turn or steer during testing.

---

## Microcontroller and Module Startup Tips

The following tips are based on real issues encountered during the bring-up of the ESP32-S3 and IFX9201SG on this project. These are things that would have saved significant debugging time if known earlier.

1. **Always check the VSO pin on the IFX9201SG first.** If SPI reads are returning 0x00 for every transaction, the SO output buffer is not powered. Connect VSO (pin 2) to 3.3V. This is easy to miss because the pin is labelled VSO rather than VCC and its function is not obvious from the pin name alone.

2. **Drive DIS high before any other GPIO initialisation on boot.** If the motor kicks on immediately at power-up, DIS is floating during the ESP32 boot sequence. Set the DIS pin high as the very first line of code before any SPI or PWM initialisation to ensure the motor outputs are disabled during startup.

3. **Drive CSN high immediately on boot.** If CSN floats low during boot the IFX9201SG enters SPI mode and ignores all PWM and DIR inputs. Add CSN as a hardware pullup to 3.3V on the PCB, and also set it high in firmware before any other motor initialisation.

4. **Verify the EN pin voltage before assuming the regulator is broken.** If the TPS563201DDCT output is very low, measure EN before replacing the IC. A reading well below 9V on EN means the pullup resistor is not connected to VIN. This was the root cause of the 0.33V output issue on this board and took significant time to diagnose.

5. **Zero PWM duty before switching motor direction via SPI.** The IFX9201SG datasheet states that SIN can only be set if DIS=0, PWM=0, and DIR=0. Failing to zero the PWM before changing the SPI control register causes the direction change to be ignored.

6. **MicroPython functions must be defined before they are called.** Unlike some languages, MicroPython executes sequentially from top to bottom. Calling a function before its definition results in a NameError. Always define all functions before the startup sequence and main loop.

7. **Use Ctrl+C in the Thonny shell to interrupt a running main.py.** If the board appears busy and Thonny cannot connect, the previous main.py is still running. Press Ctrl+C in the shell panel to interrupt it before attempting to upload new files.

8. **Name test files something other than main.py during development.** Any file named main.py runs automatically on boot. Naming test files motor_test.py or similar and running them manually from Thonny prevents the board from entering an unresponsive state if the code has a bug.

9. **Verify your SPI mode against the datasheet before writing any firmware.** The IFX9201SG uses Mode 1 (CPOL=0, CPHA=1) with data sampled on the falling edge of SCK. Using the wrong SPI mode produces garbage reads from the diagnosis register that look plausible but are incorrect.

10. **Check the heat slug solder joint first if the H-bridge has no output.** If all input pins read correct voltages but OUT1 and OUT2 show nothing, the heat slug is likely not soldered to the GND plane. The heat slug is an electrical ground connection as well as a thermal pad. An unsoldered heat slug prevents correct IC operation entirely.

---

## Lessons Learned

**1. Read the full datasheet before writing a single line of code or placing a single component.** The VSO pin connection, the CSN internal pullup behaviour, and the SPI mode requirements were all documented in the IFX9201SG datasheet. Had the datasheet been read cover to cover before beginning the design, multiple hours of debugging time would have been saved. Datasheets contain critical information that is not obvious from the component name or pinout alone.

**2. PCB layout is just as important as the schematic.** Several issues encountered during bring-up were caused by layout decisions rather than schematic errors. The VBST capacitor placement relative to the TPS563201DDCT, the heat slug via array under the IFX9201SG, and the length and width of high-current traces all had direct impacts on circuit performance. Good layout practice is a skill that requires deliberate study and cannot be improvised.

**3. Component selection requires more than a datasheet check.** The LQM21PN2R2NGCD inductor was selected based on its inductance value alone without verifying the saturation current rating. A 2.2uH inductor that saturates at 800mA is fundamentally incompatible with a 3A buck converter. Always verify all relevant specifications including saturation current, voltage rating, and DC bias derating before finalising a component selection.

**4. Power budgeting must happen early and inform component selection.** The power budget for this project was completed after the component selections were finalised. Completing the power budget first would have revealed the fuse hold current margin issue and informed the wall supply current requirement before the DigiKey order was placed, avoiding the need for last-minute substitutions.

**5. A modular firmware architecture makes debugging much faster.** Separating the SPI communication functions, motor control logic, and UART protocol handling into distinct functions made it possible to test each subsystem independently. When the SPI reads were returning 0x00, the modular structure made it immediately clear that the fault was in the hardware rather than the SPI transfer function.

**6. Boot sequence matters as much as steady-state operation.** Several of the most difficult bugs encountered in this project only occurred during the first few milliseconds of power-up, when GPIO pins were in an undefined state. Designing the startup sequence carefully, driving safety-critical pins like DIS to their safe state first, and adding hardware pullups to ensure safe default states are habits that should be applied to every design from the start.

**7. The UART bus protocol requires every node to be well-behaved.** A daisy-chain bus has no arbitration mechanism and any node that transmits at the wrong time or forwards a malformed packet can disrupt communication for the entire team. The team_config.py and uart_protocol.py shared libraries were essential for maintaining consistency across all nodes, and deviating from the protocol even slightly caused difficult-to-trace communication failures.

**8. Test points are not optional.** The absence of test points on the SW node, VFB, and motor output pins made oscilloscope probing significantly more difficult on the densely populated PCB. Adding test points to every critical net is a small investment in PCB area that pays back many times over during bring-up and debugging.

**9. Tolerance stack-up in resistor dividers matters.** The VFB feedback divider used 5% tolerance resistors for the 330K ohm component (R1) when the design required 1% tolerance. A 5% variation on a 330K resistor in the feedback network can shift the output voltage by up to 165mV from the target, which is outside the acceptable regulation window. Always use 1% or better resistors in any precision voltage setting application.

**10. Communication with teammates early and often prevents integration failures.** The UART protocol, message types, and board IDs were defined as a team early in the project, which made the final integration significantly smoother than it would have been otherwise. Any discrepancy in the protocol between nodes would have caused complete communication failure. Agreeing on shared constants, data formats, and timing requirements as a team before any individual node begins firmware development is essential for a multi-node bus system.

---

## Recommendations for Future Students

1. **Learn KiCad before the project starts, not during it.** The schematic and PCB design tools have a significant learning curve, and trying to learn them at the same time as designing a complex multi-component board leads to avoidable mistakes. Work through at least one complete tutorial project including schematic entry, footprint assignment, PCB layout, and Gerber export before beginning your class project.

2. **Order your components as early as possible and order extras.** Shipping delays, out-of-stock parts, and assembly errors are all guaranteed to happen. Ordering two to three times the quantity you need for passive components and at least two of every active IC gives you the buffer to recover from mistakes without waiting for a second delivery. A one-week shipping delay at a critical point in the project can cost you an entire design iteration.

3. **Practice soldering SMD components on a practice board before soldering your real PCB.** Hand soldering 0805 passives and SOT-23 ICs requires a different technique than through-hole soldering and the skill does not transfer automatically. Ruining a component or a PCB pad on a practice board costs almost nothing, whereas damaging your only PCB at a critical deadline is a serious setback.

4. **Understand your microcontroller's boot sequence and GPIO initialisation order before writing firmware.** The ESP32 in particular has strapping pins, GPIO default states, and a boot mode selection mechanism that can cause unexpected behaviour if not understood. Read the microcontroller datasheet's power-on reset and boot configuration sections before writing any firmware, and design your hardware to ensure safe default states on all critical pins during the boot window.

5. **Document every design decision as you make it, not at the end.** The website and report requirements ask you to explain your rationale for every component selection, every requirement, and every design trade-off. If you write these explanations as you make the decisions the documentation is accurate, detailed, and takes very little extra time. If you try to reconstruct your reasoning weeks later from memory it is both difficult and less accurate. Keep a simple engineering notebook or running document throughout the project and update it every time you make a significant decision.