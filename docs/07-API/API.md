# Motor Drive Subsystem API

## Overview

My Motor Drive subsystem is the actuation node on Team 305's daisy-chain UART bus. The board is an ESP32-S3-WROOM-1-N4 running MicroPython, paired with an IFX9201SGAUMA1 H-bridge motor driver controlled via SPI. It receives motor speed commands from Christo's HMI node, drives the Pololu 4843 DC gearmotor bidirectionally using PWM and SPI control, forwards all non-addressed packets downstream, and sends ACK responses for every packet addressed directly to this node.

The IFX9201SG is operated in SPI control mode, with the LEDC PWM peripheral on GPIO4 providing the speed signal and the SPI diagnosis register polled after every motor command to report fault status in real time.

---

## Packet Frame

| Byte(s) | Field | Value |
|---|---|---|
| 0-1 | Start | 0xA5 0x5A |
| 2 | Sender ID | see table below |
| 3 | Receiver ID | see table below |
| 4 | Message Type | uint8_t |
| 5-61 | Data | up to 57 bytes |
| 62-63 | Stop | 0x59 0x42 |

Total packet size is always 64 bytes. All fields are fixed-position. The message type is a single byte at position 4. The data payload begins at position 5.

Baud rate: 9600, 8N1, no parity, no flow control.

Byte stuffing: `build_packet()` subtracts 1 (mod 256) from any byte at positions 4-61 that equals a reserved value (0xA5, 0x5A, 0x59, 0x42). Any subsystem parsing packets from this board must add 1 back to stuffed bytes before interpreting data.

---

## Teammate IDs

Each node ID is the ASCII value of the person's first initial, which keeps addresses readable in a serial monitor without a lookup table.

| Name | Subsystem | Board ID | Initial |
|---|---|---|---|
| Christo | HMI | 0x43 | C |
| Liam | Motor Drive | 0x4C | L |
| Isaiah | Environmental Sensor | 0x49 | I |
| Arianna | Camera + Encoder | 0x41 | A |
| Myles | Distance Sensor | 0x4D | M |
| Ragul | Gyroscope | 0x52 | R |
| Damian | MQTT / Wireless | 0x44 | D |
| (none) | Broadcast / Everyone | 0x58 | X |

---

## Receive State Machine

Every byte off the UART runs through three states before any routing decision happens. Junk between frames is silently ignored. A false start (0xA5 not followed by 0x5A) drops back to WAIT_P1 without losing the second byte if it is another 0xA5.

```
WAIT_P1  ->  byte == 0xA5?  yes -> WAIT_P2,  no -> ignore
WAIT_P2  ->  byte == 0x5A?  yes -> COLLECT,  no -> WAIT_P1
COLLECT  ->  accumulate 64 bytes, then VALIDATE
VALIDATE ->  stop bits present? sender known? receiver known? not loopback?
             all pass -> ROUTE,  any fail -> discard, back to WAIT_P1
ROUTE    ->  receiver == 0x4C:  handle + ACK, do NOT forward
             receiver == 0x58:  handle AND forward downstream
             receiver == other: forward only, do NOT handle
```

RX drains completely before any outbound packet goes out. Packets with bad start/stop bits, unknown IDs, or a sender of 0x4C (loopback) are dropped with an error log to the REPL.

---

## Messages I Receive

All multi-byte fields are little-endian. Data bytes are numbered from 0 starting at packet byte 5.

### Type 0x01: Motor Speed Command

**From:** Christo HMI (0x43)

| Data Byte | 0 |
|---|---|
| Field | speed |
| C Type | int8_t (sent as uint8_t, two's complement) |
| Range | -100 to 100 |
| Example | -30 sent as 0xE2 |

Speed is a PWM duty cycle percentage clamped to +/-100. Negative values command reverse direction, positive values command forward direction, and zero stops the motor. On receipt the node decodes the signed value, sets the IFX9201SG direction via SPI control register (SDIR bit), and applies the duty cycle to the PWM peripheral on GPIO4. The SPI diagnosis register is polled immediately after the motor command is applied and any fault is printed to the REPL.

**Motor control logic:**

| Speed Value | Direction | Action |
|---|---|---|
| 1 to 100 | Forward | SDIR=1, PWM duty = speed% |
| -1 to -100 | Reverse | SDIR=0, PWM duty = abs(speed)% |
| 0 | Stop | SEN=0, PWM duty = 0 |

### Type 0x02: Stop Command

**From:** Any node (0x58 broadcast or 0x4C direct)

| Data Byte | none |
|---|---|
| Field | (no payload) |
| C Type | (none) |

Immediately sets PWM duty to 0, disables the IFX9201SG outputs via SPI (SEN=0), resets the SPI diagnosis register, and polls fault status. Motor speed is set to 0 in firmware state. An ACK is returned if the packet was addressed directly to 0x4C.

### Type 0xAA: ACK

**From:** Any node

| Packet Byte | 4 | 5 |
|---|---|---|
| Field | msg_type | received_type |
| Value | 0xAA | echoes the type that triggered the ACK |

ACK packets addressed to 0x4C are logged to the REPL and consumed. They are not forwarded downstream.

---

## Messages I Send

### Type 0xAA: ACK

**To:** Sender of any valid packet addressed to 0x4C

| Packet Byte | 4 | 5 |
|---|---|---|
| Field | msg_type | received_type |
| Value | 0xAA (fixed) | echoes the type that triggered it |
| Example | 0xAA | 0x01 |

Every valid packet addressed directly to 0x4C receives an immediate ACK back to the sender. Broadcast packets (0x58) do not receive an ACK. Sending ACKs to a broadcast address would flood the bus with simultaneous responses from every node.

---

## Routing Logic

| Condition | Action |
|---|---|
| Receiver == 0x4C (me) | Handle, send ACK, do not forward |
| Receiver == 0x58 (broadcast) | Handle and forward downstream |
| Sender == 0x4C (loopback) | Discard silently (E09) |
| Receiver == other known node | Forward immediately, do not handle |
| Malformed / unknown ID | Discard, print error to REPL |

---

## IFX9201SG SPI Control Interface

The motor driver is operated in SPI mode (SIN=1 in the control register). The SPI bus runs at 1MHz, Mode 1 (CPOL=0, CPHA=1), 8-bit words, MSB first, per the IFX9201SG datasheet section 4.12.

### SPI Commands

| Command | Byte Value | Description |
|---|---|---|
| RD_DIA | 0x00 | Read diagnosis register |
| RES_DIA | 0x80 | Reset diagnosis register |
| RD_REV | 0x20 | Read device revision |
| RD_CTRL | 0x60 | Read control register |
| WR_CTRL | 0xE0 | Write control register |
| WR_CTRL_RD_DIA | 0xC0 | Write control and read diagnosis |

### Control Register Bits (WR_CTRL: 0xE0 | data)

| Bit | Name | Description |
|---|---|---|
| 4 | OLDIS | 1 = disconnect open load current source |
| 3 | SIN | 1 = SPI control mode, 0 = PWM/DIR mode |
| 2 | SEN | 1 = enable outputs, 0 = disable outputs |
| 1 | SDIR | 1 = forward, 0 = reverse |
| 0 | SPWM | 1 = PWM input active |

### Diagnosis Register Fault Codes (lower nibble)

| DIA[3:0] | Hex | Fault | Latched |
|---|---|---|---|
| 1111 | 0xF | No failure | No |
| 1110 | 0xE | Short to GND at OUT1 | Yes |
| 1101 | 0xD | Short to VS at OUT1 | Yes |
| 1100 | 0xC | Open load | No |
| 1011 | 0xB | Short to GND at OUT2 | Yes |
| 1010 | 0xA | Short to GND at OUT1 and OUT2 | Yes |
| 1001 | 0x9 | Short to VS at OUT1 and GND at OUT2 | Yes |
| 0111 | 0x7 | Short to VS at OUT2 | Yes |
| 0110 | 0x6 | Short to GND at OUT1 and VS at OUT2 | Yes |
| 0101 | 0x5 | Short to VS at OUT1 and OUT2 | Yes |
| 0011 | 0x3 | VS undervoltage | No |

Latched faults require an explicit RES_DIA command or a DIS pin toggle to clear. Non-latched faults clear automatically when the fault condition is resolved.

---

## GPIO Pin Assignment

| GPIO | Function | Direction | Connected To |
|---|---|---|---|
| GPIO4 | Motor PWM | Output | IFX9201SG PWM (pin 12) |
| GPIO5 | Motor DIR | Output | IFX9201SG DIR (pin 1) |
| GPIO6 | Motor DIS | Output | IFX9201SG DIS (pin 11) |
| GPIO11 | SPI MOSI | Output | IFX9201SG SI (pin 8) |
| GPIO12 | SPI SCK | Output | IFX9201SG SCK (pin 10) |
| GPIO13 | SPI MISO | Input | IFX9201SG SO (pin 3) |
| GPIO39 | SPI CSN | Output | IFX9201SG CSN (pin 9) |
| GPIO43 | UART TX | Output | UART bus connector OUT |
| GPIO44 | UART RX | Input | UART bus connector IN |
| GPIO47 | Debug LED | Output | 330 ohm resistor to LED to GND |

---

## Byte Stuffing

`build_packet()` subtracts 1 (mod 256) from any byte at positions 4-61 (msg_type through end of data) that matches a reserved value. Reserved set: 0xA5, 0x5A, 0x59, 0x42. Any node receiving a packet from this board must add 1 back to recover stuffed bytes before interpreting the data.

---

## Error Codes

| Code | Meaning |
|---|---|
| E01 | Missing start bits |
| E02 | Missing stop bits |
| E03 | Bad packet length |
| E04 | Sender ID not in known node list |
| E05 | Receiver ID not in known node list |
| E07 | Payload too long |
| E08 | Zero ID detected in sender or receiver field |
| E09 | Loopback detected, packet discarded |