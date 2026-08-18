# EPEver Tracer → Home Assistant

![EPEver Tracer 4310A](images/Tracer%204310A.jpg)

Private repository for the EPEver Tracer Modbus integration used with Home Assistant Green.

**Status:** working telemetry and LOAD control; consolidated through 2026-08-17.

## Transport
- EPEver Tracer 4310A
- USB ↔ RS485 via FTDI adapter
- Modbus RTU, slave 1
- Working HA serial path: `/dev/serial/by-id/usb-FTDI_USB-RS485_Cable_AV0M39PA-if00-port0`
- 115200 baud, 8N1

## Current functionality
PV, battery and LOAD telemetry are available. Status words are available at `0x3200–0x3202`. LOAD manual control uses coil 2 when the controller is in manual output-control mode.

Charging state from `0x3201` bits D3–D2:
- 0 no charging
- 1 Float
- 2 Boost
- 3 Equalization

## Tested hardware

- Solar controller: EPEver Tracer 4310A
- Home automation controller: Home Assistant Green
- Communication adapter: FTDI USB to RS485 adapter
- Interface: RS485 / Modbus RTU
- USB device:
  `/dev/serial/by-id/usb-FTDI_USB-RS485_Cable_AV0M39PA-if00-port0`
- Battery system: 24 V nominal, 2 × 12 V LiFePO4 batteries in series

This repository is intentionally separate from the SRNE project because EPEver uses RS485 and its own documented register map.

## Documentation

- [Complete hardware setup](docs/HARDWARE_SETUP.md)
- [Modbus register map](docs/REGISTER_MAP.md)
- [Status bit decoding](docs/STATUS_BITS.md)
- [Test log](docs/TEST_LOG.md)
- [Next steps](docs/NEXT_STEPS.md)

## Disclaimer

This project is provided for informational and experimental purposes only.

The code, configuration examples, register mappings, wiring information, and other documentation in this repository are provided "as is", without warranty of any kind.

Use of this project is entirely at your own risk. You are responsible for verifying compatibility with your exact EPEver controller model, firmware, battery system, wiring, and Home Assistant setup before applying any configuration or making any changes.

Incorrect configuration, register writes, wiring, or charging parameters may cause malfunction, data loss, equipment damage, battery damage, or other unintended consequences.

I accept no responsibility or liability for any damage, loss, injury, malfunction, or other consequences resulting from the use, modification, or implementation of the information, code, or examples provided in this repository.

By using this project, you accept full responsibility for your own installation, testing, configuration, and use.

## Support

If this project has been useful to you and you would like to support further testing, documentation, and development, you can buy me a coffee:

[☕ Buy me a coffee](https://buymeacoffee.com/tb_rtk)

Support is completely optional and is greatly appreciated.
