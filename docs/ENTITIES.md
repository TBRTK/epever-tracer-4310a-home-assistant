# EPEver Tracer 4310A — Home Assistant Entities

This document lists the Home Assistant entities provided by the
**EPEver Tracer 4310A Modbus RTU integration** in this repository.

The project is designed for use with **Home Assistant Green** and an
**FTDI USB-RS485 adapter**.

Entity IDs may differ slightly if Home Assistant has already created entities with
similar names. The names below reflect the project configuration and intended naming.

---

## 1. Entity overview

| Entity name | Type | Source | Purpose |
|---|---|---|---|
| EPever PV Voltage | Sensor | `0x3100` | Solar array input voltage |
| EPever PV Current | Sensor | `0x3101` | Solar array input current |
| EPever PV Power | Sensor | `0x3102–0x3103` | Solar input power |
| EPever Battery Voltage | Sensor | `0x3104` | Battery-bank voltage |
| EPever Battery Charge Current | Sensor | `0x3105` | Charging current to battery |
| EPever Battery Charge Power | Sensor | `0x3106–0x3107` | Charging power to battery |
| EPever Load Voltage | Sensor | `0x310C` | LOAD output voltage |
| EPever Load Current | Sensor | `0x310D` | LOAD output current |
| EPever Load Power | Sensor | `0x310E–0x310F` | LOAD output power |
| EPever Battery Temperature | Sensor | `0x3110` | Battery temperature |
| EPever Controller Temperature | Sensor | `0x3111` | Controller temperature |
| EPever Battery SOC | Sensor | `0x311A` | Controller-reported battery SOC |
| EPever Battery Status Raw | Sensor | `0x3200` | Raw battery status word |
| EPever Charging Status Raw | Sensor | `0x3201` | Raw charging status word |
| EPever Discharging Status Raw | Sensor | `0x3202` | Raw LOAD/discharging status word |
| EPever Charging State | Template sensor | `0x3201` D3-D2 | Human-readable charging mode |
| EPever Load | Switch | Coil `2` | Manual LOAD ON/OFF control |

---

# 2. PV entities

## EPever PV Voltage

**Purpose:** Measures the voltage from the PV array at the charge-controller input.

```text
Register: 0x3100
Decimal address: 12544
Register type: Input register
Data type: uint16
Scale: 0.01
Unit: V
```

Example:

```text
Raw value: 3650
Displayed value: 36.50 V
```

Recommended Home Assistant metadata:

```yaml
device_class: voltage
state_class: measurement
unit_of_measurement: "V"
```

---

## EPever PV Current

**Purpose:** Measures PV current entering the controller.

```text
Register: 0x3101
Decimal address: 12545
Register type: Input register
Data type: uint16
Scale: 0.01
Unit: A
```

Example:

```text
Raw value: 148
Displayed value: 1.48 A
```

Recommended Home Assistant metadata:

```yaml
device_class: current
state_class: measurement
unit_of_measurement: "A"
```

---

## EPever PV Power

**Purpose:** Reports solar input power.

```text
Registers: 0x3102 + 0x3103
Decimal start address: 12546
Register type: Input register
Data type: uint32
Scale: 0.01
Unit: W
Word order: low/high register pair
```

The project configuration uses a 32-bit value and word swapping as required by the
EPEver register layout.

Recommended Home Assistant metadata:

```yaml
device_class: power
state_class: measurement
unit_of_measurement: "W"
```

---

# 3. Battery entities

## EPever Battery Voltage

**Purpose:** Battery-bank voltage measured by the charge controller.

```text
Register: 0x3104
Decimal address: 12548
Register type: Input register
Data type: uint16
Scale: 0.01
Unit: V
```

Example for a 24 V battery system:

```text
Raw value: 2705
Displayed value: 27.05 V
```

Recommended metadata:

```yaml
device_class: voltage
state_class: measurement
unit_of_measurement: "V"
```

---

## EPever Battery Charge Current

**Purpose:** Current being delivered by the EPEver controller to the battery.

```text
Register: 0x3105
Decimal address: 12549
Register type: Input register
Data type: uint16
Scale: 0.01
Unit: A
```

This value represents charge current from the EPEver controller.

It should not be assumed to represent total battery current if other chargers or loads
are directly connected to the battery.

Recommended metadata:

```yaml
device_class: current
state_class: measurement
unit_of_measurement: "A"
```

---

## EPever Battery Charge Power

**Purpose:** Charging power delivered by the controller to the battery.

```text
Registers: 0x3106 + 0x3107
Decimal start address: 12550
Register type: Input register
Data type: uint32
Scale: 0.01
Unit: W
```

Recommended metadata:

```yaml
device_class: power
state_class: measurement
unit_of_measurement: "W"
```

---

## EPever Battery SOC

**Purpose:** Battery state-of-charge estimate reported by the EPEver controller.

```text
Register: 0x311A
Decimal address: 12570
Register type: Input register
Data type: uint16
Scale: 1
Unit: %
```

Example:

```text
Raw value: 84
Displayed value: 84 %
```

### Important note

This is the **controller-reported SOC estimate**.

It should not automatically be treated as equivalent to SOC from a battery BMS or a
dedicated coulomb counter.

In systems with a separate Bluetooth BMS, compare the two values before using the
EPEver SOC for critical automations.

Recommended metadata:

```yaml
state_class: measurement
unit_of_measurement: "%"
```

---

# 4. LOAD entities

## EPever Load Voltage

**Purpose:** Voltage present on the controller's LOAD output.

```text
Register: 0x310C
Decimal address: 12556
Register type: Input register
Data type: uint16
Scale: 0.01
Unit: V
```

Recommended metadata:

```yaml
device_class: voltage
state_class: measurement
unit_of_measurement: "V"
```

---

## EPever Load Current

**Purpose:** Current flowing through the controller's LOAD output.

```text
Register: 0x310D
Decimal address: 12557
Register type: Input register
Data type: uint16
Scale: 0.01
Unit: A
```

Recommended metadata:

```yaml
device_class: current
state_class: measurement
unit_of_measurement: "A"
```

---

## EPever Load Power

**Purpose:** Power being consumed through the LOAD output.

```text
Registers: 0x310E + 0x310F
Decimal start address: 12558
Register type: Input register
Data type: uint32
Scale: 0.01
Unit: W
```

Recommended metadata:

```yaml
device_class: power
state_class: measurement
unit_of_measurement: "W"
```

---

# 5. Temperature entities

## EPever Battery Temperature

**Purpose:** Battery temperature reported through the EPEver controller.

```text
Register: 0x3110
Decimal address: 12560
Register type: Input register
Data type: int16
Scale: 0.01
Unit: °C
```

Recommended metadata:

```yaml
device_class: temperature
state_class: measurement
unit_of_measurement: "°C"
```

The validity of this value depends on the temperature-sensing hardware available to the
specific controller installation.

---

## EPever Controller Temperature

**Purpose:** Internal/controller temperature reported by EPEver.

```text
Register: 0x3111
Decimal address: 12561
Register type: Input register
Data type: int16
Scale: 0.01
Unit: °C
```

Recommended metadata:

```yaml
device_class: temperature
state_class: measurement
unit_of_measurement: "°C"
```

---

# 6. Raw status entities

The EPEver protocol exposes three important 16-bit status words.

These sensors are useful both for troubleshooting and for building template sensors.

---

## EPever Battery Status Raw

```text
Register: 0x3200
Decimal address: 12800
Register type: Input register
Data type: uint16
```

Contains battery-related status bits.

Possible information includes:

- battery voltage condition
- battery temperature condition
- internal resistance abnormality
- rated-voltage identification status

See:

[`STATUS_BITS.md`](STATUS_BITS.md)

for detailed bit decoding.

---

## EPever Charging Status Raw

```text
Register: 0x3201
Decimal address: 12801
Register type: Input register
Data type: uint16
```

This is one of the most useful status registers.

It contains information related to:

- charging state
- controller running/standby
- charging fault
- PV input condition
- MOSFET faults
- input over-current
- other charging-related status

Bits D3-D2 contain the charging mode.

---

## EPever Discharging Status Raw

```text
Register: 0x3202
Decimal address: 12802
Register type: Input register
Data type: uint16
```

Contains LOAD/discharging-related status information.

This can be used to decode:

- LOAD running / standby
- discharging fault
- output voltage state
- short-circuit condition
- load level / related status bits

See:

[`STATUS_BITS.md`](STATUS_BITS.md)

for the currently documented bit layout.

---

# 7. Human-readable charging-state entity

## EPever Charging State

This is a Home Assistant template sensor derived from:

```text
sensor.epever_charging_status_raw
```

The charging mode is stored in bits:

```text
D3-D2
```

Mapping:

| Extracted value | Charging state |
|---:|---|
| 0 | No charging |
| 1 | Float |
| 2 | Boost |
| 3 | Equalization |

Recommended template:

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

The expression:

```text
(raw // 4) % 4
```

extracts the D3-D2 bit pair.

---

# 8. LOAD switch

## EPever Load

**Type:** Home Assistant switch

```text
Modbus object: Coil
Coil address: 2
Purpose: Manual LOAD ON/OFF
```

Example configuration:

```yaml
switches:
  - name: EPever Load
    slave: 1
    address: 2
    write_type: coil
    command_on: 1
    command_off: 0
```

### Important

The controller must be configured for the appropriate manual LOAD-control mode.

This switch controls only the physical LOAD output.

It does **not**:

- change PV charging current
- change battery charging voltage
- enable or disable the MPPT algorithm
- control an external inverter unless that inverter is powered from the LOAD output

---

# 9. Suggested Home Assistant entity IDs

Home Assistant normally creates entity IDs automatically.

Expected IDs may look similar to:

```text
sensor.epever_pv_voltage
sensor.epever_pv_current
sensor.epever_pv_power

sensor.epever_battery_voltage
sensor.epever_battery_charge_current
sensor.epever_battery_charge_power
sensor.epever_battery_soc

sensor.epever_battery_temperature
sensor.epever_controller_temperature

sensor.epever_load_voltage
sensor.epever_load_current
sensor.epever_load_power

sensor.epever_battery_status_raw
sensor.epever_charging_status_raw
sensor.epever_discharging_status_raw

sensor.epever_charging_state

switch.epever_load
```

The actual entity IDs in your Home Assistant installation may differ.

Always check:

```text
Settings
→ Devices & services
→ Entities
```

before copying an entity ID into dashboards or automations.

---

# 10. Recommended dashboard grouping

For a simple Home Assistant dashboard, the entities can be grouped like this:

## Solar

```text
PV Voltage
PV Current
PV Power
Charging State
```

## Battery

```text
Battery Voltage
Battery Charge Current
Battery Charge Power
Battery SOC
Battery Temperature
```

## LOAD

```text
LOAD Voltage
LOAD Current
LOAD Power
LOAD switch
```

## Diagnostics

```text
Controller Temperature
Battery Status Raw
Charging Status Raw
Discharging Status Raw
```

This keeps raw diagnostic data away from the normal day-to-day dashboard while still
making it available for troubleshooting.

---

# 11. Recommended entity use

## Good candidates for normal monitoring

Use directly:

```text
PV Voltage
PV Current
PV Power
Battery Voltage
Battery Charge Current
Battery Charge Power
Battery SOC
LOAD Voltage
LOAD Current
LOAD Power
Battery Temperature
Controller Temperature
Charging State
```

## Good candidates for automations

Examples:

```text
Battery Voltage
Battery SOC
Charging State
PV Power
LOAD Power
LOAD switch
```

## Diagnostic-only entities

Keep available, but they normally do not need to be visible on the main dashboard:

```text
Battery Status Raw
Charging Status Raw
Discharging Status Raw
```

Instead, decode relevant bits into separate human-readable template entities as the
project develops.

---

# 12. Data interpretation notes

## PV Power vs Battery Charge Power

These values are not expected to be identical.

The MPPT controller converts PV voltage/current to a different battery voltage/current.

Controller conversion losses also exist.

Therefore:

```text
PV Power ≈ Battery Charge Power + controller losses
```

when no unusual operating condition is present.

---

## Battery charge current vs total battery current

`EPever Battery Charge Current` reports charge current from the EPEver controller.

If the battery also has:

- another charger
- DC loads connected directly to the battery
- an inverter connected directly to the battery
- another MPPT controller

then the EPEver value is not the same as the total net current entering or leaving the battery.

For true battery net current, use a BMS or shunt designed for coulomb/current measurement.

---

## Battery SOC

The EPEver SOC value is controller-derived.

For LiFePO4 systems, voltage-based SOC estimation can be less precise because LiFePO4 has
a relatively flat voltage curve over much of its usable capacity.

A battery BMS or dedicated shunt may provide a better SOC estimate depending on the system.

---

# 13. Status decoding roadmap

The repository currently exposes all three raw status words.

Planned improvements include human-readable entities for:

- charging fault
- charging running / standby
- PV connected / no input power
- battery voltage state
- battery temperature state
- LOAD running / standby
- LOAD fault
- LOAD short circuit
- other documented EPEver status bits

These should be added as template sensors or binary sensors only after the relevant bit
definitions have been verified against the protocol documentation and tested where practical.

---

# 14. Verified project functionality

| Entity / function | Status |
|---|---|
| PV Voltage | ✅ Verified |
| PV Current | ✅ Verified |
| PV Power | ✅ Verified |
| Battery Voltage | ✅ Verified |
| Battery Charge Current | ✅ Verified |
| Battery Charge Power | ✅ Verified |
| Battery SOC | ✅ Verified |
| Battery Temperature | ✅ Available |
| Controller Temperature | ✅ Available |
| LOAD Voltage | ✅ Verified |
| LOAD Current | ✅ Verified |
| LOAD Power | ✅ Verified |
| Battery Status Raw | ✅ Available |
| Charging Status Raw | ✅ Available |
| Discharging Status Raw | ✅ Available |
| Charging State | ✅ Configuration documented |
| LOAD ON/OFF switch | ✅ Verified |
| Detailed fault entities | 🚧 In progress |
| Remote charging-current control | ❌ Not implemented |

---

# 15. Related documentation

- [Complete hardware setup](HARDWARE_SETUP.md)
- [Home Assistant installation guide](INSTALLATION.md)
- [Modbus register map](REGISTER_MAP.md)
- [Status bit decoding](STATUS_BITS.md)
- [Test log](TEST_LOG.md)
- [Next steps](NEXT_STEPS.md)

---

# 16. Model compatibility

This entity list is based on the project configuration for:

**EPEver Tracer 4310A**

Other EPEver models may expose similar Modbus registers, but compatibility should not be
assumed unless it has been verified for the exact model and firmware.

---

# 17. Safety

Monitoring entities are read-only and normally low risk.

The `EPever Load` entity writes to a Modbus coil and changes the physical LOAD output state.

Before using control entities in automations:

- verify the controlled equipment
- verify the controller's LOAD operating mode
- test ON and OFF manually
- consider what happens after Home Assistant or controller restart
- do not use unverified control commands for safety-critical equipment

See the main repository README for the full disclaimer.
