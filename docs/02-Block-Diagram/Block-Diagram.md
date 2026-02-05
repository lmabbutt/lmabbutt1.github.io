---
title: Module's Block Diagram
tags:
- tag1
- tag2
---

## Drive Train Control Subsystem Overview

The drive train control subsystem is responsible for translating user commands into precise and reliable motor motion for the robot platform. This subsystem uses a dual-microcontroller architecture to separate high-level communication from low-level real-time motor control.

An **ESP32** is used to handle **Bluetooth Low Energy (BLE)** communication with an external user device, such as a laptop or mobile phone. It receives motion commands wirelessly and forwards them to the motor control subsystem using a **UART serial interface**. This separation allows the ESP32 to manage wireless communication without impacting time-critical motor control tasks.

A **PIC18F47K42 microcontroller** serves as the dedicated motor controller. It interprets commands received from the ESP32 and generates the required **PWM signals and digital control outputs** to drive the H-bridge motor driver. The PIC is responsible for controlling motor speed, direction, and enable signals, ensuring deterministic and reliable operation of the drive motors.

An external **H-bridge motor driver** interfaces between the PIC and the DC motors, allowing high-current motor operation while protecting the microcontroller from electrical stress. The subsystem operates across multiple voltage domains, with regulated logic supplies for the microcontrollers and a higher-voltage supply dedicated to motor power.

Overall, this architecture provides a modular, robust, and scalable solution for robotic drivetrain control by cleanly separating communication, control, and power stages.


To get some initial formatting help, one can view ["here"](https://embedded-systems-design.github.io/EGR304DataSheetTemplate/Appendix/basic-markdown-examples/) some basic techniques.


## Example Block Diagram 
Showing an example of how to import a screenshot of the block diagram created outside of git and brought into a page.

![Example of Indivial Block diagram ](INDIVIDUALBLOCKDIAGRAM.png)
