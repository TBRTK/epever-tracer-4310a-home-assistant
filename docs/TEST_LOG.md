# Test / integration log

Home Assistant successfully opened `/dev/serial/by-id/usb-FTDI_USB-RS485_Cable_AV0M39PA-if00-port0`. Early troubleshooting included no-response and transient missing-device-path errors. Later debug logs confirmed valid Modbus replies; `0x311A` returned SOC 100 during one test.

Project work subsequently established PV, battery and LOAD telemetry, status-register reads and manual LOAD switching using coil 2.
