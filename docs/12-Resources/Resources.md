# Resources

## Final Code Project

The complete MicroPython firmware project for the Motor Drive subsystem is available as a zip file download below. The project includes all source files required to run the motor driver node on the ESP32-S3-WROOM-1-N4.

[Download Motor Drive Firmware (motor_drive_firmware.zip)](MotorSubsystemCode.zip)

### Files Included in the Zip

| File | Description |
|---|---|
| main.py | Entry point, imports and runs motor_driver module |
| motor_driver.py | Main motor driver node, UART bus handler, SPI motor control |
| team_config.py | Shared team protocol constants, node IDs, message types |
| uart_protocol.py | UART packet framing, validation, routing, and ACK logic |

---

## Demo Video


### Full System Integration Demo

The video below demonstrates all Team 305 nodes operating together on the daisy-chain UART bus, with the HMI node sending motor commands that are received and executed by the motor drive node. [Watch on YouTube](https://youtube.com/shorts/P3W65iciSMs)

<iframe
  width="315"
  height="560"
  src="https://www.youtube.com/embed/P3W65iciSMs"
  title="Team 305 Full System Integration Demo"
  frameborder="0"
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
  allowfullscreen>
</iframe>

---

## Additional Links

| Resource | Link |
|---|---|
| ESP32-S3-WROOM-1 Datasheet | [Espressif](https://www.espressif.com/sites/default/files/documentation/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf) |
| IFX9201SGAUMA1 Datasheet | [Infineon](https://www.infineon.com/dgdl/Infineon-IFX9201SG-DS-v01_01-EN.pdf) |
| TPS563201DDCT Datasheet | [Texas Instruments](https://www.ti.com/lit/ds/symlink/tps563201.pdf) |
| Pololu 4843 Motor Page | [Pololu](https://www.pololu.com/product/4843) |
| MicroPython ESP32-S3 Firmware | [micropython.org](https://micropython.org/download/ESP32_GENERIC_S3/) |
| KiCad PCB Design Software | [kicad.org](https://www.kicad.org) |
| JLCPCB PCB Fabrication | [jlcpcb.com](https://jlcpcb.com) |