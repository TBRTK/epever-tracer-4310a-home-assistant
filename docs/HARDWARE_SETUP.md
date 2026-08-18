# EPEver Tracer 4310A Hardware Setup

This guide documents the tested hardware setup used to connect an **EPEver Tracer 4310A** solar charge controller to **Home Assistant Green** using Modbus RTU over RS485.

The setup described here is based on the hardware that has been physically tested in this project.

---

## 1. Tested hardware

### Solar charge controller

- Model: **EPEver Tracer 4310A**
- System voltage: 12 / 24 V
- Communication: RS485
- Protocol: Modbus RTU

### Home Assistant

- Controller: **Home Assistant Green**
- Operating system: Home Assistant OS
- Connection to EPEver: USB

### USB to RS485 adapter

- Adapter type: **FTDI USB to RS485**
- USB chipset: FTDI
- Linux device path used in this project:

```text
/dev/serial/by-id/usb-FTDI_USB-RS485_Cable_YOUR_DEVICE_ID-if00-port0
```

Using `/dev/serial/by-id/` is preferred over `/dev/ttyUSB0` because the device path normally remains stable after rebooting Home Assistant.

### Battery system used during testing

- Nominal battery voltage: **24 V**
- Battery configuration: **2 × 12 V LiFePO4 batteries in series**

The Modbus integration itself is independent of the battery chemistry, but charging parameters must always match the actual battery system.

---

## 2. Connection overview

The physical connection is:

```text
┌────────────────────────┐
│ EPEver Tracer 4310A    │
│                        │
│ Communication port     │
└────────────┬───────────┘
             │
             │ RS485
             │
┌────────────▼───────────┐
│ FTDI USB ↔ RS485       │
│ Adapter                │
└────────────┬───────────┘
             │
             │ USB
             │
┌────────────▼───────────┐
│ Home Assistant Green   │
└────────────────────────┘
```

The Home Assistant Green communicates with the EPEver controller directly over Modbus RTU.

No Ethernet, Wi-Fi, cloud service or external gateway is required for this connection.

---

## 3. RS485 wiring

RS485 normally uses two differential signal wires:

```text
RS485 A
RS485 B
```

These may also be labelled:

```text
A / B
D+ / D-
485+ / 485-
```

depending on the adapter manufacturer.

Connect the EPEver RS485 communication lines to the corresponding terminals on the USB-RS485 adapter.

Typical connection:

```text
EPEver                 USB-RS485 adapter

RS485 A  ------------  A
RS485 B  ------------  B
```

Some adapters also provide a GND terminal.

If a signal ground is used, connect it only according to the controller and adapter documentation.

Do not connect power from the USB-RS485 adapter to the EPEver controller unless the interface documentation explicitly requires it.

---

## 4. Important RS485 polarity note

If Modbus communication does not work, one of the first things to check is the A/B polarity.

Different manufacturers sometimes use opposite naming conventions for RS485 A and B.

Symptoms of reversed polarity usually include:

- no Modbus response
- timeout errors
- CRC errors
- Home Assistant showing unavailable sensors

If the wiring is otherwise correct and there is no response, swap A and B and test again.

Do not change anything on the battery or PV wiring when troubleshooting RS485 communication.

---

## 5. Connect the USB-RS485 adapter

Connect the FTDI USB-RS485 adapter to one of the USB ports on the Home Assistant Green.

Home Assistant OS should automatically detect the FTDI serial adapter.

The adapter used in this project appears as:

```text
/dev/serial/by-id/usb-FTDI_USB-RS485_Cable_YOUR_DEVICE_ID-if00-port0
```

---

## 6. Find the serial device in Home Assistant

Open Home Assistant.

Go to:

```text
Settings
→ System
→ Hardware
```

Open the hardware list and look for the USB serial adapter.

The preferred device path starts with:

```text
/dev/serial/by-id/
```

Example from this project:

```text
/dev/serial/by-id/usb-FTDI_USB-RS485_Cable_YOUR_DEVICE_ID-if00-port0
```

Copy the complete path.

Do not rely on:

```text
/dev/ttyUSB0
```

unless necessary.

`ttyUSB0` may change after reboot or if another USB serial adapter is connected.

---

## 7. Modbus communication settings

The working configuration used in this project is:

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

Home Assistant communicates directly with slave address:

```text
1
```

---

## 8. Home Assistant Modbus configuration

The project uses a separate YAML file:

```text
modbus_epever_v2.yaml
```

The main Home Assistant configuration contains:

```yaml
modbus: !include modbus_epever_v2.yaml
```

The serial connection section is:

```yaml
- name: epever
  type: serial
  method: rtu
  port: /dev/serial/by-id/usb-FTDI_USB-RS485_Cable_YOUR_DEVICE_ID-if00-port0
  baudrate: 115200
  bytesize: 8
  parity: N
  stopbits: 1
  timeout: 3
```

Replace the `port:` value if your FTDI adapter has a different `/dev/serial/by-id/` path.

---

## 9. Tested Modbus data

The following Modbus data has been used in this project.

### PV data

```text
0x3100  PV voltage
0x3101  PV current
0x3102  PV power low word
0x3103  PV power high word
```

### Battery data

```text
0x3104  Battery voltage
0x3105  Battery charging current
0x3106  Battery charging power low word
0x3107  Battery charging power high word
```

### LOAD output

```text
0x310C  LOAD voltage
0x310D  LOAD current
0x310E  LOAD power low word
0x310F  LOAD power high word
```

### Temperature

```text
0x3110  Battery temperature
0x3111  Controller temperature
```

### Battery SOC

```text
0x311A  Battery state of charge
```

### Status registers

```text
0x3200  Battery status
0x3201  Charging equipment status
0x3202  Discharging / LOAD status
```

---

## 10. Charging mode status

The EPEver charging status is available through register:

```text
0x3201
```

Bits D3-D2 indicate the charging state.

```text
00 = No charging
01 = Float
10 = Boost
11 = Equalization
```

Equivalent decimal values after extracting the two bits:

```text
0 = No charging
1 = Float
2 = Boost
3 = Equalization
```

This allows Home Assistant to display the actual charging state reported by the controller instead of estimating it from battery voltage.

---

## 11. LOAD output control

The EPEver LOAD output can be controlled through Modbus.

The project has successfully used:

```text
Coil 2
```

for manual LOAD ON/OFF control.

The controller must be configured for the correct manual output-control mode for this to behave predictably.

Example Home Assistant switch:

```yaml
switches:
  - name: EPever Load
    slave: 1
    address: 2
    write_type: coil
    command_on: 1
    command_off: 0
```

This controls only the EPEver LOAD output.

It does not control the MPPT charging current.

---

## 12. Verify configuration before restarting Home Assistant

After editing the YAML files, check the Home Assistant configuration.

Using the Home Assistant terminal:

```bash
ha core check
```

If the result is successful, restart Home Assistant Core:

```bash
ha core restart
```

Do not restart before fixing YAML errors reported by `ha core check`.

---

## 13. Confirm communication

After Home Assistant has restarted, check the Modbus entities.

Expected examples include:

```text
EPever PV Voltage
EPever PV Current
EPever PV Power

EPever Battery Voltage
EPever Battery Charge Current
EPever Battery Charge Power
EPever Battery SOC

EPever Load Voltage
EPever Load Current
EPever Load Power
```

The values should correspond approximately with the controller display and external measurements.

---

## 14. Basic troubleshooting

### No entities / entities unavailable

Check:

```text
USB-RS485 adapter detected
Correct /dev/serial/by-id/ path
RS485 A/B wiring
Slave address = 1
Baud rate = 115200
8 data bits
No parity
1 stop bit
```

### Adapter exists but controller does not reply

Check:

```text
RS485 polarity
A/B wiring
communication connector
controller powered
correct serial settings
```

Try swapping RS485 A and B if the wiring convention is uncertain.

### Communication works intermittently

Check:

- loose RS485 terminals
- poor connectors
- excessively long or unshielded communication wiring
- electrical noise
- unstable USB connection
- duplicate Modbus master devices on the same bus

---

## 15. Recommended installation practice

For a permanent installation:

- use a stable `/dev/serial/by-id/` USB path
- provide strain relief for the USB and RS485 cables
- avoid routing RS485 wiring directly beside high-current inverter cables
- label RS485 A and B
- keep a copy of the working Home Assistant configuration
- verify the controller model before writing configuration registers

For longer RS485 cable runs, twisted-pair cable is preferred.

---

## 16. What has been physically tested

This project has been tested with:

```text
EPEver Tracer 4310A
Home Assistant Green
FTDI USB-RS485 adapter
24 V battery system
2 × 12 V LiFePO4 batteries in series
```

Verified project functionality includes:

| Function | Status |
|---|---|
| Modbus RTU communication | ✅ Verified |
| PV voltage | ✅ Verified |
| PV current | ✅ Verified |
| PV power | ✅ Verified |
| Battery voltage | ✅ Verified |
| Battery charge current | ✅ Verified |
| Battery SOC | ✅ Verified |
| LOAD voltage | ✅ Verified |
| LOAD current | ✅ Verified |
| LOAD power | ✅ Verified |
| LOAD ON/OFF through Modbus | ✅ Verified |
| Raw battery status | ✅ Available |
| Raw charging status | ✅ Available |
| Raw LOAD/discharging status | ✅ Available |
| Charging state decoding | 🧪 Being documented |
| Detailed fault decoding | 🧪 Being documented |
| Remote charging-current control | ❌ Not implemented |

---

## 17. Model compatibility

This project has been specifically tested with:

**EPEver Tracer 4310A**

Other EPEver Tracer, AN, XTRA or related controllers may use similar Modbus registers, but compatibility should not be assumed unless it has been verified for the specific model and firmware.

Always check the documentation for your exact controller before using writable Modbus registers.

---

## 18. Safety

Modbus writes can change controller behaviour.

Incorrect battery charging parameters may result in:

- incorrect charging voltage
- battery damage
- BMS disconnects
- equipment damage
- unsafe operating conditions

Read-only monitoring is inherently lower risk than writing controller configuration.

Do not experiment with undocumented writable registers on an operational battery system.

See the main project README for the full project disclaimer.
