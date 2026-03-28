# Motor Controller — Message Datasheet

**Network:** UART daisy-chain  
**Data types:** MicroPython / C  
**Byte order:** Little-endian for all multi-byte fields

| Legend | Meaning |
|--------|---------|
| 🔵 Send | This board transmits the message |
| 🟢 Receive | This board listens and acts on the message |
| 🟡 Both | This board sends and receives this message |

---

## Message Type 1 — Motor Control State Report 🔵 Send

Broadcast by the motor controller to report commanded motor states.

| Byte(s) | Variable Name | C Type | Bytes | Min Value | Max Value | Example | Notes |
|---------|--------------|--------|-------|-----------|-----------|---------|-------|
| 1–2 | `message_type` | `uint16_t` | 2 | 1 | 1 | `1` | Always 1 for this message |
| 3–4 | `left_motor_speed` | `int16_t` | 2 | -255 | 255 | `-200` | PWM duty cycle; negative = reverse ⚠️ *team must confirm range* |
| 5–6 | `right_motor_speed` | `int16_t` | 2 | -255 | 255 | `200` | PWM duty cycle; negative = reverse ⚠️ *team must confirm range* |
| 7 | `left_motor_dir` | `uint8_t` | 1 | 0 | 1 | `0` | 0 = reverse, 1 = forward |
| 8 | `right_motor_dir` | `uint8_t` | 1 | 0 | 1 | `1` | 0 = reverse, 1 = forward |

**Total payload:** 8 bytes

> ⚠️ **Missing from team table:** No payload length field or checksum/CRC — recommend adding for UART reliability.

> ⚠️ **Redundancy concern:** Direction is encoded twice — via the sign of `int16_t` speed AND via the direction byte. Team should decide: use sign of `int16_t` only, or use unsigned speed + direction byte exclusively.

---

## Message Type 2 — Motor Status Report 🔵 Send

Periodic report of measured motor operating conditions sent by this board.

| Byte(s) | Variable Name | C Type | Bytes | Min Value | Max Value | Example | Notes |
|---------|--------------|--------|-------|-----------|-----------|---------|-------|
| 1–2 | `message_type` | `uint16_t` | 2 | 2 | 2 | `2` | Always 2 for this message |
| 3–4 | `left_motor_current_ma` | `uint16_t` | 2 | 0 | 5000 | `1200` | Measured current in mA; `uint16_t` supports 0–65535 |
| 5–6 | `right_motor_current_ma` | `uint16_t` | 2 | 0 | 5000 | `1350` | Measured current in mA |
| 7 | `left_motor_status` | `uint8_t` | 1 | 0 | 255 | `0` | Status code ⚠️ *team must define code table* |
| 8 | `right_motor_status` | `uint8_t` | 1 | 0 | 255 | `0` | Status code ⚠️ *team must define code table* |

**Total payload:** 8 bytes

> ⚠️ **Missing from team table:** No actual motor RPM or encoder feedback field. If encoders are present, a `uint16_t rpm` field per motor is recommended for closed-loop control.

> ⚠️ **Motor status codes are undefined.** Suggested definitions:
> - `0` = OK
> - `1` = Stalled
> - `2` = Overcurrent
> - `3` = Fault

---

## Message Type 13 — System Status Report 🟢 Receive / Act On

Broadcast from the main controller. The motor controller monitors system state and battery voltage to adjust behavior (e.g., emergency stop, low-battery throttling).

| Byte(s) | Variable Name | C Type | Bytes | Min Value | Max Value | Example | Notes |
|---------|--------------|--------|-------|-----------|-----------|---------|-------|
| 1–2 | `message_type` | `uint16_t` | 2 | 13 | 13 | `13` | Filter on this value to identify message |
| 3 | `system_state` | `uint8_t` | 1 | 0 | 255 | `1` | Act on this: stop motors if state = emergency stop ⚠️ *state codes undefined* |
| 4–7 | `uptime_ms` | `uint32_t` | 4 | 0 | 4294967295 | `12400` | Informational; can ignore or log |
| 8–9 | `battery_voltage_mv` | `uint16_t` | 2 | 0 | 65535 | `11800` | Act on this: throttle motors if battery low ⚠️ *team must define low-voltage threshold* |

**Total payload:** 9 bytes

> ⚠️ **Missing from team table:** `system_state` codes are not defined. The motor controller must know which value triggers an emergency stop vs. normal operation.

---

## Message Type 14 — System Error Code Report 🟡 Send & Receive

Sent by this board to report motor faults; also received from other subsystems to monitor overall system health.

| Byte(s) | Variable Name | C Type | Bytes | Min Value | Max Value | Example | Notes |
|---------|--------------|--------|-------|-----------|-----------|---------|-------|
| 1–2 | `message_type` | `uint16_t` | 2 | 14 | 14 | `14` | Always 14 for this message |
| 3 | `subsystem_id` | `uint8_t` | 1 | 0 | 255 | `2` | When sending: use this board's assigned ID ⚠️ *team must assign subsystem IDs* |
| 4 | `error_code` | `uint8_t` | 1 | 0 | 255 | `2` | When sending: motor fault code ⚠️ *team must define error code table* |

**Total payload:** 4 bytes

> ⚠️ **Missing from team table:** No subsystem ID assignments. This board needs a unique `subsystem_id` to identify itself when sending errors.

> ⚠️ **Error codes are undefined.** Suggested definitions:
> - `0` = None
> - `1` = Left motor stall
> - `2` = Right motor stall
> - `3` = Left motor overcurrent
> - `4` = Right motor overcurrent
> - `5` = Communication timeout

---

## Summary

| Message | Direction | Total Payload Bytes |
|---------|-----------|-------------------|
| Type 1 — Motor control state report | 🔵 Send | 8 |
| Type 2 — Motor status report | 🔵 Send | 8 |
| Type 13 — System status report | 🟢 Receive | 9 |
| Type 14 — System error code report | 🟡 Both | 4 |

---

## Open Items for Team Resolution

| # | Issue | Affects |
|---|-------|---------|
| 1 | Confirm PWM speed range (currently assumed -255 to 255) | Type 1 |
| 2 | Resolve direction encoding redundancy (sign vs. direction byte) | Type 1 |
| 3 | Define motor status codes (0=OK, 1=stall, etc.) | Type 2 |
| 4 | Add RPM/encoder feedback fields if using closed-loop control | Type 2 |
| 5 | Define `system_state` code values (especially emergency stop) | Type 13 |
| 6 | Define low-battery voltage threshold for motor throttling | Type 13 |
| 7 | Assign unique `subsystem_id` to the motor controller board | Type 14 |
| 8 | Define error code table for motor faults | Type 14 |
| 9 | Add payload length and CRC/checksum to all messages | All |