---
title: Welcome
tags:
- motor drive
- ESP32
- EGR314
---
<center>
<font size= "6">Liam Mabbutt's Individual Datasheet</font><br>
as part of<br>
<font size= "8">Mars Exploration Rover</font><br>
for<br>
<font size= "5">Team 305 | EGR314 Spring 2026</font><br>

**Submission: May 4th, 2026**
</center>

---

## Introduction

This datasheet documents the design, implementation, and testing of the Motor Drive subsystem developed by Liam Mabbutt as part of Team 305's Mars Exploration Rover project. The Motor Drive subsystem is responsible for providing rover mobility through bidirectional DC motor control, receiving speed and direction commands from the team's HMI node over a shared UART daisy-chain bus, and reporting motor fault status in real time via SPI diagnostics. The board is built around an ESP32-S3-WROOM-1-N4 microcontroller, an IFX9201SGAUMA1 H-bridge motor driver, and a TPS563201DDCT switching voltage regulator, and is designed to integrate seamlessly with the rest of the team's multi-node communication architecture.

---

## Project Summary

Team 305's Mars Exploration Rover is a small autonomous rover designed for surface exploration tasks. The rover uses a distributed control architecture where each team member is responsible for an individual subsystem node connected via a shared UART daisy-chain bus. The HMI node operated by Christo serves as the central user interface, collecting sensor data from all nodes and issuing motor commands to the drive system. The remaining nodes each handle a specific sensing or actuation function and communicate using a fixed 64-byte packet protocol with consistent addressing, framing, and acknowledgement behaviour across all nodes.

The Motor Drive subsystem contributes the actuation capability of the rover, translating high-level speed and direction commands from the HMI into physical wheel motion. Without a functioning drive system the rover cannot move, making this subsystem foundational to the overall project success. The subsystem was designed to support two independent motors for differential steering, though Version 1.0 brought up and validated a single motor channel during the individual subsystem phase.

For the full team context, system-level block diagram, and integrated project report, visit the [Team 305 Report](https://egr314-s-2026-30.github.io/EGR314-S-2026-305.github.io/).

---

## My Contribution

My role on Team 305 was the Actuation Subsystem member, responsible for designing and implementing the motor drive hardware and firmware. This included selecting the motor, H-bridge driver, and voltage regulator; designing the KiCad schematic and PCB layout; assembling and bringing up the board; and writing the MicroPython firmware to integrate with the team's UART bus protocol.

The key deliverables from this subsystem are a custom two-layer PCB capable of driving a DC gearmotor bidirectionally at variable speed, a fully integrated UART node that receives motor commands from the HMI and forwards all other bus traffic downstream, and SPI-based fault monitoring of the H-bridge in real time.

---

## Datasheet Navigation

Use the sections below to navigate to specific areas of this datasheet:

| Section | Description |
|---|---|
| [Requirements](../01-Requirements/Requirements/) | Product requirements and design constraints for the motor drive subsystem |
| [Component Selection](../02-Component-Selection/Component-Selection/) | Final component selections, trade-off analysis, and rationale for all major ICs |
| [Hardware Proposal](../03-Hardware-Proposal/Hardware-Proposal/) | Schematic design and PCB layout for the motor drive board |
| [BOM](../04-BOM/BOM/) | Full bill of materials with part numbers, quantities, and DigiKey links |
| [Power Budget](../05-Power-Budget/Power-Budget/) | Power rail analysis, regulator selection, and battery life estimate |
| [API](../06-API/API/) | UART communication protocol, message types, and SPI register interface |
| [PCB](../07-PCB/PCB/) | PCB front and back images, Gerber files, and ECAD project download |
| [Hardware V2.0](../08-Hardware-V2/Hardware-V2/) | Proposed improvements for a second hardware revision |
| [Reflection](../09-Reflection/Reflection/) | Requirements review, lessons learned, and recommendations for future students |
| [Resources](../10-Resources/Resources/) | Final firmware download and demo videos |