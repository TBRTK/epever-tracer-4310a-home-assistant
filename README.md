# EPEver Tracer → Home Assistant

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

## Support

If this project has been useful to you and you would like to support further testing, documentation, and development, you can buy me a coffee:

[☕ Buy me a coffee](https://buymeacoffee.com/tb_rtk)

Support is completely optional and is greatly appreciated.
