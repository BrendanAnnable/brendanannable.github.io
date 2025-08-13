---
layout: default
title: EMHASS Config Examples
---

# EMHASS Config Examples

Create `/config/emhass_conf.yaml` for EMHASS. Replace entity IDs with your own.

## Minimal config (Amber price-driven battery control)

```yaml
# /config/emhass_conf.yaml
hass:
  url: "http://homeassistant.local:8123"
  token: "REDACTED"

site:
  timezone: "Australia/Sydney"
  meter_import_price_entity: sensor.amber_import_price    # c/kWh
  meter_export_price_entity: sensor.amber_export_price    # c/kWh

battery:
  soc_entity: sensor.battery_soc
  charge_service: script.battery_charge_on                # or switch/inverter service
  discharge_service: script.battery_discharge_on
  stop_service: script.battery_stop
  soc_min: 10
  soc_max: 95
  max_charge_kw: 10
  max_discharge_kw: 10
  roundtrip_efficiency: 0.92

pv:
  pv_power_entity: sensor.pv_power                         # optional live power
  pv_forecast_entity: sensor.solcast_forecast_power_30min # optional forecast power series

load:
  load_power_entity: sensor.house_load_power               # optional
  # alternatively: let EMHASS estimate

optimization:
  horizon_hours: 24
  resolution_minutes: 15
  objective: cost                                          # minimize energy cost
  prefer_export_when_price_above_c_per_kwh: 25             # example threshold
  prefer_charge_when_price_below_c_per_kwh: 10             # example threshold
```

> 💡 Tip: Start simple. Once baseline works, add PV/load forecasts to improve accuracy.

## Using template sensors to normalize Amber units

Some Amber sensors expose **$/kWh**; EMHASS examples often use **c/kWh**. Create template sensors in Home Assistant converting to **c/kWh**:

```yaml
# configuration.yaml
template:
  - sensor:
      - name: "Amber Import Price (c/kWh)"
        unit_of_measurement: "c/kWh"
        state: >-
          {% set p = states('sensor.amber_import_price') | float(0) %}
          {{ (p * 100.0) | round(3) }}  # $/kWh -> c/kWh
        availability: >-
          {{ states('sensor.amber_import_price') not in ['unknown','unavailable','none'] }}
      - name: "Amber Export Price (c/kWh)"
        unit_of_measurement: "c/kWh"
        state: >-
          {% set p = states('sensor.amber_export_price') | float(0) %}
          {{ (p * 100.0) | round(3) }}
```

Then reference these new sensors in EMHASS.

## Exposing actions as HA Scripts

Create simple scripts that EMHASS can call (via `service` calls) to control your inverter/charger:

```yaml
# scripts.yaml
battery_charge_on:
  alias: Battery: Charge
  sequence:
    - service: inverter.set_mode
      target: { entity_id: inverter.your_device }
      data: { mode: "charge" }

battery_discharge_on:
  alias: Battery: Discharge
  sequence:
    - service: inverter.set_mode
      target: { entity_id: inverter.your_device }
      data: { mode: "discharge" }

battery_stop:
  alias: Battery: Stop
  sequence:
    - service: inverter.set_mode
      target: { entity_id: inverter.your_device }
      data: { mode: "auto" }
```

Adapt these to your device's actual services.
