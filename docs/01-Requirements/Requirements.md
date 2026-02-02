---
title: Mobility & Motor Control Board – Project Requirements
---

## Project Requirements

The purpose of this requirements table is to clearly define the functional and design constraints of the Mobility & Motor Control Board before proceeding further into development. By identifying the minimum acceptable performance levels, target goals, and potential stretch objectives, this table ensures that design decisions remain aligned with the overall rover concept and system-level integration requirements.

This module is responsible for providing rover mobility through motor actuation, speed regulation, steering control, feedback monitoring, and safety interlocks. The requirements listed below capture the critical features this module must support to successfully integrate with the team’s overall rover design.

| **Requirement Description** | **Measure of<br>Threshold** | **Target<br>Measure** | **Stretch<br>Requirement<br>(Y/N)** |
|-----------------------------|----------------------------|----------------------|:----------------------------------:|
| Microcontroller power regulation | Stable 3.2 V supply | Regulated 3.3 V supply | N |
| Microcontroller selection | PIC or ESP-class MCU | ESP32 microcontroller | N |
| Motor configuration support | 2-wheel drive (2WD) | Expandable to 4-wheel drive (4WD) | Y |
| Motor type compatibility | Brushed DC motors | High-efficiency DC gear motors | N |
| Motor driver requirement | External motor driver IC | Integrated motor driver with protection | N |
| Motor speed control | Open-loop PWM control | Closed-loop speed control | N |
| Steering control method | Differential steering | Independent wheel steering control | Y |
| Motor feedback sensing | Single feedback signal (current or speed) | Current and speed feedback | N |
| Safety interlock implementation | Software-based motor disable | Hardware and software interlocks | N |
| Communication with main controller | Digital control signals | Fully synchronized control interface | N |
