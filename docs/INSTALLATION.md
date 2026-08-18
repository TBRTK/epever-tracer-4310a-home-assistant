# EPEver Tracer 4310A — Home Assistant Installation

This guide explains how to install and verify the Home Assistant integration for an
**EPEver Tracer 4310A** using Modbus RTU over an FTDI USB-RS485 adapter.

It assumes the hardware has already been connected as described in
[`HARDWARE_SETUP.md`](HARDWARE_SETUP.md).

---

## 1. Requirements

You need:

- EPEver Tracer 4310A
- Home Assistant Green (Or Home Assistant OS running on a platform that connects to Tracer 4310A and the FTDI adapter)
- FTDI USB-RS485 adapter
- Working RS485 connection between the Tracer and the adapter
- Home Assistant OS
- Access to edit Home Assistant YAML files
- Terminal access to Home Assistant for configuration checking

This project has been physically tested with a 24 V battery bank consisting of
2 × 12 V LiFePO4 batteries in series.

---

## 2. Serial settings

The working Modbus RTU settings used in this project are:

```text
Protocol: Modbus RTU
Slave address: 1
Baud rate: 115200
Data bits: 8
Parity: None
Stop bits: 1
```

Short form:

```text
115200 8N1
```

The adapter used during testing appears in Home Assistant as:

```text
/dev/serial/by-id/usb-FTDI_USB-RS485_Cable_AV0M39PA-if00-port0
```

Your adapter may have a different device path.

Use `/dev/serial/by-id/...` when possible instead of `/dev/ttyUSB0`, because the
`by-id` path is normally stable across reboots.

---

## 3. Find the USB serial device

In Home Assistant, open:

```text
Settings
→ System
→ Hardware
```

Find the FTDI USB serial adapter and note the complete device path.

It should normally begin with:

```text
/dev/serial/by-id/
```

Example:

```text
/dev/serial/by-id/usb-FTDI_USB-RS485_Cable_AV0M39PA-if00-port0
```

You will use this exact path in the Modbus configuration.

---

## 4. Add the Modbus include to `configuration.yaml`

Open:

```text
/config/configuration.yaml
```

Add:

```yaml
modbus: !include modbus_epever_v2.yaml
```

If you already have a `modbus:` section, do not add a second top-level `modbus:` key.
Merge the EPEver hub into your existing Modbus configuration instead.

---

## 5. Add `modbus_epever_v2.yaml`

Create:

```text
/config/modbus_epever_v2.yaml
```

The repository contains the project version here:

```text
home-assistant/modbus_epever_v2.yaml
```

Copy that file into your Home Assistant `/config/` directory.

The serial connection section should look like this:

```yaml
- name: epever
  type: serial
  port: /dev/serial/by-id/usb-FTDI_USB-RS485_Cable_AV0M39PA-if00-port0
  baudrate: 115200
  bytesize: 8
  method: rtu
  parity: N
  stopbits: 1
  timeout: 3
```

Replace the `port:` value with the path from your own Home Assistant system if it differs.

---

## 6. Telemetry included in the project

The current project configuration includes sensors for:

### PV

- PV voltage
- PV current
- PV power

### Battery

- Battery voltage
- Battery charging current
- Battery charging power
- Battery SOC
- Battery temperature

### Controller

- Controller temperature

### LOAD output

- LOAD voltage
- LOAD current
- LOAD power

### Status

- Raw battery status (`0x3200`)
- Raw charging status (`0x3201`)
- Raw discharging / LOAD status (`0x3202`)

### Control

- Manual LOAD ON/OFF using Modbus coil 2

---

## 7. Charging-state sensor

The EPEver protocol reports charging state in register:

```text
0x3201
```

Bits D3-D2 contain the charging mode:

```text
00 = No charging
01 = Float
10 = Boost
11 = Equalization
```

After extracting those two bits:

```text
0 = No charging
1 = Float
2 = Boost
3 = Equalization
```

The raw Home Assistant sensor used by this project is:

```text
sensor.epever_charging_status_raw
```

To create a human-readable charging-state sensor, add the following under the
top-level `template:` section in `configuration.yaml`, or place it in your existing
template include structure:

```yaml
template:
  - sensor:
      - name: "EPever Charging State"
        unique_id: epever_charging_state
        icon: mdi:battery-charging
        availability: >
          {{ states('sensor.epever_charging_status_raw') not in ['unknown', 'unavailable', 'none'] }}
        state: >
          {% set raw = states('sensor.epever_charging_status_raw') | int(0) %}
          {% set mode = (raw // 4) % 4 %}
          {% if mode == 0 %}
            No charging
          {% elif mode == 1 %}
            Float
          {% elif mode == 2 %}
            Boost
          {% elif mode == 3 %}
            Equalization
          {% else %}
            Unknown
          {% endif %}
```

The calculation:

```text
(raw // 4) % 4
```

extracts bits D3-D2 from the 16-bit status word.

---

## 8. LOAD control

The project uses Modbus coil:

```text
2
```

for manual LOAD ON/OFF control.

Example:

```yaml
switches:
  - name: EPever Load
    slave: 1
    address: 2
    write_type: coil
    command_on: 1
    command_off: 0
```

The EPEver controller must be configured for the appropriate manual output-control mode
for this to behave predictably.

This controls only the **LOAD output** on the controller.

It does **not** control MPPT charging current.

---

## 9. Check the configuration

Before restarting Home Assistant, run:

```bash
ha core check
```

Do not continue until the configuration check succeeds.

If the check reports an error, fix the YAML before restarting.

---

## 10. Restart Home Assistant Core

After a successful configuration check:

```bash
ha core restart
```

Wait for Home Assistant to come back online.

---

## 11. Verify the entities

Open:

```text
Settings
→ Devices & services
→ Entities
```

Search for:

```text
EPever
```

Expected entities include names similar to:

```text
EPever PV Voltage
EPever PV Current
EPever PV Power

EPever Battery Voltage
EPever Battery Charge Current
EPever Battery Charge Power
EPever Battery SOC

EPever Battery Temperature
EPever Controller Temperature

EPever Load Voltage
EPever Load Current
EPever Load Power

EPever Battery Status Raw
EPever Charging Status Raw
EPever Discharging Status Raw

EPever Load
EPever Charging State
```

Entity IDs can vary if Home Assistant already has entities with similar names.

---

## 12. Verify the measurements

Compare Home Assistant values with the EPEver display and, where practical, external
measurements.

Check:

```text
PV voltage
Battery voltage
Charging current
LOAD voltage
LOAD current
LOAD power
Battery SOC
```

Small differences can occur because values may be sampled at different times.

Large or obviously incorrect differences should be investigated before using the data
for automations.

---

## 13. Verify LOAD switching

Only perform this test if the LOAD output is safe to switch.

1. Confirm the controller is in the intended manual LOAD-control mode.
2. Turn `EPever Load` ON in Home Assistant.
3. Confirm the physical LOAD output becomes active.
4. Check that LOAD voltage/current/power sensors respond.
5. Turn `EPever Load` OFF.
6. Confirm the physical output switches off.

Do not use an important or safety-critical load for the first test.

---

## 14. Basic troubleshooting

### All EPEver entities are unavailable

Check:

- controller is powered
- USB-RS485 adapter is connected
- correct `/dev/serial/by-id/...` path
- RS485 A/B wiring
- slave address = 1
- 115200 baud
- 8 data bits
- no parity
- 1 stop bit

### USB adapter is visible but the Tracer does not respond

Check:

- RS485 A/B polarity
- communication connector
- loose terminals
- controller power
- serial settings

If A/B naming is uncertain, swapping A and B is a common diagnostic step.

### Home Assistant reports that the serial port is already in use

Only one Modbus hub/process should control the same serial port.

Check that:

- the same adapter is not configured twice
- another integration is not using the port
- no other Modbus master is connected to the same USB serial device

### Values are present but incorrect

Check:

- correct controller model
- correct register address
- correct `input_type`
- correct data type
- correct scale
- correct 16/32-bit word handling

Do not compensate for obviously incorrect values with arbitrary scaling until the register
definition has been verified.

---

## 15. Useful Modbus registers

### Real-time data

| Address | Description |
|---|---|
| `0x3100` | PV voltage |
| `0x3101` | PV current |
| `0x3102–0x3103` | PV power |
| `0x3104` | Battery voltage |
| `0x3105` | Battery charging current |
| `0x3106–0x3107` | Battery charging power |
| `0x310C` | LOAD voltage |
| `0x310D` | LOAD current |
| `0x310E–0x310F` | LOAD power |
| `0x3110` | Battery temperature |
| `0x3111` | Controller temperature |
| `0x311A` | Battery SOC |

### Status registers

| Address | Description |
|---|---|
| `0x3200` | Battery status |
| `0x3201` | Charging equipment status |
| `0x3202` | Discharging / LOAD status |

### Control

| Address | Description |
|---|---|
| Coil `2` | Manual LOAD ON/OFF |

For more details, see:

- [`REGISTER_MAP.md`](REGISTER_MAP.md)
- [`STATUS_BITS.md`](STATUS_BITS.md)

---

## 16. Current project status

| Function | Status |
|---|---|
| Modbus RTU communication | ✅ Verified |
| PV telemetry | ✅ Verified |
| Battery telemetry | ✅ Verified |
| LOAD telemetry | ✅ Verified |
| Battery SOC | ✅ Verified |
| Temperature telemetry | ✅ Available |
| LOAD ON/OFF through Modbus | ✅ Verified |
| Raw battery status | ✅ Available |
| Raw charging status | ✅ Available |
| Raw discharging status | ✅ Available |
| Human-readable charging state | ✅ Configuration provided |
| Detailed fault decoding | 🚧 In progress |
| Remote charging-current control | ❌ Not implemented |

---

## 17. Important model note

This project has been specifically developed and tested for:

**EPEver Tracer 4310A**

Other EPEver Tracer, AN, XTRA or related controllers may use similar register maps.

Do not assume that writable registers, communication settings, or firmware behaviour are
identical across different controller families.

Verify compatibility with your exact controller before making Modbus writes.

---

## 18. Safety and responsibility

Read-only monitoring is lower risk than writing settings or control commands.

Modbus writes can change controller behaviour.

Incorrect charging parameters or control commands may cause:

- incorrect battery charging
- BMS disconnects
- loss of power to connected equipment
- battery damage
- equipment damage
- unsafe operating conditions

Do not experiment with undocumented writable registers on an operational system.

Use the code and documentation in this repository at your own risk.

See the main repository README for the full disclaimer.
