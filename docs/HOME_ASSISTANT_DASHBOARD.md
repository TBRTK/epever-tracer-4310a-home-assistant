# Home Assistant dashboard examples

The screenshots below show a working Home Assistant dashboard used to compare the EPEver Tracer 4310A with an SRNE MT2410N10 / Biltema 25-5099 controller.

They demonstrate one possible layout for presenting the Modbus data exposed by this project in Home Assistant.

## PV

![PV comparison](../images/Skjermbilde%202026-08-23%20203825.png)

## Battery / charging

![Battery and charging comparison](../images/Skjermbilde%202026-08-23%20203914.png)

## LOAD

![LOAD comparison](../images/Skjermbilde%202026-08-23%20203950.png)

## Status and diagnostics

![Status and diagnostics](../images/Skjermbilde%202026-08-23%20204015.png)

## Layout shown

The example dashboard contains:

- PV power, current and voltage for EPEver and SRNE
- Gauge cards for PV power
- Battery / charging current and voltage
- Estimated SOC
- Charging status such as `Float` and `Bulk / MPPT`
- LOAD power, current, voltage and status
- Gauge cards for LOAD current
- Status overview
- Diagnostic/raw values used while verifying Modbus status decoding

The dashboard is included as an illustration of the resulting Home Assistant entities and one possible user-interface layout. Exact values shown in the screenshots are live measurements from the systems at the time the screenshots were taken and should not be interpreted as fixed expected values.

The EPEver and SRNE systems shown operate at different nominal battery voltages, so their voltage values are intentionally not directly comparable.

See `home-assistant/` for the Home Assistant configuration used by this project.