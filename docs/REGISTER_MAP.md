# EPEver Tracer register map

## Real-time input registers (0x04)
| Address | Meaning | Scale |
|---|---|---|
| `0x3100` | PV voltage | ×0.01 V |
| `0x3101` | PV current | ×0.01 A |
| `0x3102/03` | PV power L/H | ×0.01 W |
| `0x3104` | Battery voltage | ×0.01 V |
| `0x3105` | Battery charge current | ×0.01 A |
| `0x3106/07` | Battery charge power L/H | ×0.01 W |
| `0x310C` | LOAD voltage | ×0.01 V |
| `0x310D` | LOAD current | ×0.01 A |
| `0x310E/0F` | LOAD power L/H | ×0.01 W |
| `0x3110` | Battery temperature | ×0.01 °C |
| `0x3111` | Controller/case temperature | ×0.01 °C |
| `0x311A` | Battery SOC | 1 % |

## Status input registers
- `0x3200` Battery status
- `0x3201` Charging equipment status
- `0x3202` Discharging/LOAD status

## Holding registers (selected)
- `0x9000` Battery type
- `0x9001` Battery capacity
- `0x9003` High-voltage disconnect
- `0x9004` Charge-limit voltage
- `0x9006` Equalization voltage
- `0x9007` Boost voltage
- `0x9008` Float voltage
- `0x9009` Boost reconnect voltage
- `0x903D` Load control mode used in this project; verify against model/firmware before changing elsewhere

## Coils
0 charging device; 1 manual/automatic output control; 2 manual LOAD; 3 default LOAD; 5 test mode; 6 force LOAD temporary test.
