# EPEver Tracer → Home Assistant

Private repository for the EPEver Tracer Modbus integration used with Home Assistant Green.

**Status:** working telemetry and LOAD control; consolidated through 2026-08-17.

## Transport
- EPEver Tracer A-series project
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

This repository is intentionally separate from the SRNE project because EPEver uses RS485 and its own documented register map.
