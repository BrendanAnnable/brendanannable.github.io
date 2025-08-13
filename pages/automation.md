---
layout: default
title: Automation & Scheduling
---

# Automation & Scheduling

## Cron triggers EMHASS → Home Assistant applies actions

One robust pattern is:
1. Cron calls EMHASS `/optimize` every 15 minutes and writes JSON to a file (or posts to a HA webhook).
2. A Home Assistant automation parses the schedule and sets targets for the next interval (e.g., charge limit, power setpoint).

### Option A: Pull JSON from file via Command Line Sensor

```yaml
# configuration.yaml
sensor:
  - platform: command_line
    name: emhass_next_action
    command: "cat /share/emhass_last.json | jq -r '.schedule[0].action'"
    scan_interval: 60
```

### Option B: EMHASS posts to a Home Assistant webhook

Create a webhook-triggered automation in HA:

```yaml
{% raw %}
# automations.yaml
- alias: "Apply EMHASS action"
  trigger:
    - platform: webhook
      webhook_id: emhass_apply
  action:
    - variables:
        payload: "{{ trigger.json }}"
    - choose:
        - conditions: "{{ payload.action == 'charge' }}"
          sequence:
            - service: script.battery_charge_on
        - conditions: "{{ payload.action == 'discharge' }}"
          sequence:
            - service: script.battery_discharge_on
      default:
        - service: script.battery_stop
{% endraw %}
```

Then have EMHASS (or your cron wrapper) `POST` the next step to the webhook URL.

## Scheduling windows (Amber-aware)

Amber prices swing throughout the day. You may prefer tighter scheduling during volatile periods (e.g., 6am–10am, 4pm–9pm). Create time-based automations that increase EMHASS frequency or adjust constraints when `import_price < X` or `export_price > Y`.
