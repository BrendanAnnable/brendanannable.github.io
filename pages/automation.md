---
layout: default
title: Automation & Scheduling
---

# Automated Battery Control

EMHASS by itself just creates the energy plan, but it won't actually know how to execute upon it. In order to execute the plan, we will need an automation.

Note: Make sure the Sigenergy integration is no longer in Read-Only Mode, as we will now be changing values automatically!

This automation switches the Sigenergy battery system between 4 operating modes depending on the energy plan:

1. Maximum Self Consumption
2. Command Discharging (PV First)
3. Command Charging (PV First)
4. Standby

Using just these 4 modes and modulating various charge and discharge limits we can achieve a great deal!

TODO: Explain how it works.

```yaml
{% raw %}alias: EMHASS Battery Management
description: ""
triggers:
  - trigger: state
    entity_id:
      - sensor.mpc_batt_power
      - sensor.mpc_grid_power
    from: null
    to: null
actions:
  - delay:
      hours: 0
      minutes: 0
      seconds: 1
      milliseconds: 0
  - variables:
      p_batt: "{{ states('sensor.mpc_batt_power') | int(0) }}"
      p_batt_kw: "{{ (p_batt / 1000) | round(3) }}"
      p_grid: "{{ states('sensor.mpc_grid_power') | int(0) }}"
      max_charge_rate: "{{ states('sensor.sigen_plant_ess_rated_charging_power') | float(0) }}"
      max_discharge_rate: "{{ states('sensor.sigen_plant_ess_rated_discharging_power') | float(0) }}"
      max_import_rate_kw: 100
  - choose:
      - conditions:
          - condition: template
            value_template: "{{ p_grid == 0 }}"
        sequence:
          - action: select.select_option
            metadata: {}
            data:
              option: Maximum Self Consumption
            target:
              entity_id: select.sigen_plant_remote_ems_control_mode
            alias: Maximum Self Consumption
          - action: number.set_value
            metadata: {}
            data:
              value: "{{ max_charge_rate }}"
            target:
              entity_id:
                - number.sigen_plant_ess_max_charging_limit
          - action: number.set_value
            metadata: {}
            data:
              value: "{{ max_discharge_rate }}"
            target:
              entity_id:
                - number.sigen_plant_ess_max_discharging_limit
        alias: Check self consume conditions
      - conditions:
          - condition: template
            value_template: "{{ p_batt > 0 or (p_grid < 0 and p_batt == 0) }}"
        sequence:
          - action: select.select_option
            metadata: {}
            data:
              option: Command Discharging (PV First)
            target:
              entity_id: select.sigen_plant_remote_ems_control_mode
          - action: number.set_value
            metadata: {}
            data:
              value: "{{ p_batt_kw }}"
            target:
              entity_id: number.sigen_plant_ess_max_discharging_limit
        alias: Check discharge conditions
      - conditions:
          - condition: template
            value_template: "{{ p_batt < 0 }}"
        sequence:
          - action: select.select_option
            metadata: {}
            data:
              option: Command Charging (PV First)
            target:
              entity_id: select.sigen_plant_remote_ems_control_mode
          - action: number.set_value
            metadata: {}
            data:
              value: "{{ p_batt_kw | abs }}"
            target:
              entity_id:
                - number.sigen_plant_ess_max_charging_limit
          - action: number.set_value
            metadata: {}
            data:
              value: "{{ max_import_rate_kw if p_grid > 0 else 0 }}"
            target:
              entity_id:
                - number.sigen_plant_pcs_import_limitation
        alias: Check charge conditions
      - conditions:
          - condition: template
            value_template: "{{ p_batt == 0 }}"
        sequence:
          - action: select.select_option
            metadata: {}
            data:
              option: Standby
            target:
              entity_id: select.sigen_plant_remote_ems_control_mode
          - action: number.set_value
            metadata: {}
            data:
              value: 0
            target:
              entity_id:
                - number.sigen_plant_pcs_import_limitation
        alias: Check standby conditions
mode: single
{% endraw %}
```

### Curtailment

You will also want an automation to automatically handle negative prices. This automation will prevent selling PV at a negative feed-in price, and will also prevent using solar when there is a negative general price (as you'll make more money drawing from the grid instead).

```
{% raw %}alias: Inverter Curtailment
description: ""
triggers:
  - trigger: numeric_state
    entity_id:
      - sensor.amber_5min_current_feed_in_price
    below: 0
    id: negative_fit
  - trigger: numeric_state
    entity_id:
      - sensor.amber_5min_current_feed_in_price
    id: positive_fit
    above: 0
  - trigger: numeric_state
    entity_id:
      - sensor.amber_5min_current_general_price
    above: 0
    id: positive_price
  - trigger: numeric_state
    entity_id:
      - sensor.amber_5min_current_general_price
    below: 0
    id: negative_price
  - trigger: state
    entity_id:
      - input_boolean.emhass_battery_control
conditions:
  - condition: state
    entity_id: input_boolean.emhass_battery_control
    state: "on"
actions:
  - alias: Curtail if feedin price is negative
    if:
      - condition: numeric_state
        entity_id: sensor.amber_5min_current_feed_in_price
        below: 0
    then:
      - action: number.set_value
        metadata: {}
        data:
          value: 0
        target:
          entity_id: number.sigen_plant_grid_export_limitation
        alias: Set export limit to 0
    else:
      - action: number.set_value
        metadata: {}
        data:
          value: 30
        target:
          entity_id: number.sigen_plant_grid_export_limitation
        alias: Set export limit to 30
  - alias: Ignore PV if usage price is negative
    if:
      - condition: numeric_state
        entity_id: sensor.amber_5min_current_general_price
        below: 0
    then:
      - action: number.set_value
        metadata: {}
        data:
          value: 0
        target:
          entity_id: number.sigen_plant_pv_max_power_limit
        alias: Set PV limit to 0
    else:
      - action: number.set_value
        metadata: {}
        data:
          value: 100
        target:
          entity_id: number.sigen_plant_pv_max_power_limit
        alias: Set PV limit to 100
mode: single
{% endraw %}
```
