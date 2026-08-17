# EPEver status bits

## `0x3201` charging equipment status
- D15–D14 input voltage status: 00 normal, 01 no power, 10 high voltage, 11 input error
- D13 charging MOSFET short
- D12 charging/anti-reverse MOSFET short
- D11 anti-reverse MOSFET short
- D10 input over-current
- D9 load over-current
- D8 load short
- D7 load MOSFET short
- D4 PV input short
- **D3–D2 charging state:** 00 no charge, 01 Float, 10 Boost, 11 Equalization
- D1 fault
- D0 running/standby

HA decoding concept: `(status_word >> 2) & 0x03`.

`0x3200` contains battery voltage/temp/fault flags. `0x3202` contains discharging/load/output status and faults.
