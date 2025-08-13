---
layout: default
title: Step-by-step Setup
---

# Step-by-step Setup

## 1) Install Amber in Home Assistant
- Add the **Amber Electric** integration (official or HACS).  
- Note the entity IDs for **import price (c/kWh)** and **feed-in/export price (c/kWh)**. Example placeholders:
  - `sensor.amber_import_price`
  - `sensor.amber_export_price`  
  Some integrations expose multiple sensors (e.g., general, solar sponge, etc.). Use the **current** import/export price sensors for decisions.

## 2) Install EMHASS (Docker)
Create a folder for EMHASS, then run:

```bash
docker run -d --name emhass   -p 5000:5000   -v $PWD/config:/config   -e HASS_URL="http://homeassistant.local:8123"   -e HASS_TOKEN="LONG_LIVED_ACCESS_TOKEN"   -e TZ="Australia/Sydney"   --restart unless-stopped   pierdu/emhass:latest
```

- Generate an HA **Long-Lived Access Token** under **Profile → Long-Lived Tokens**.
- After it starts, EMHASS API will be at `http://<host>:5000`.

## 3) Map your HA entities to EMHASS
Create `/config/emhass_conf.yaml` inside the container volume and set key parameters. See [EMHASS Config Examples](/pages/emhass-config.html).

## 4) (Optional) Configure forecasts
- **PV forecast**: Use Solcast integration in HA; set `pv_power_forecast_entity` to your forecast entity (e.g., `sensor.solcast_pv_forecast_today` or a power-by-time series if available).
- **Load forecast**: You can start with a naive forecast (e.g., moving average) or use historical loads from HA; EMHASS can estimate if not provided.

## 5) Test EMHASS calls
EMHASS exposes endpoints like `/optimize`. Quick smoke test:

```bash
curl -X POST "http://localhost:5000/optimize"   -H "Content-Type: application/json"   -d '{"timedelta_hours": 24, "prediction_horizon": 24}'
```

You should receive a JSON schedule with optimal charge/discharge actions.

## 6) Schedule runs
Use cron on the Docker host to run every 15 minutes (Amber updates frequently):

```bash
*/15 * * * * curl -s -X POST http://localhost:5000/optimize -H "Content-Type: application/json" -d '{"timedelta_hours":24,"prediction_horizon":24}' > /tmp/emhass_last.json
```

Then apply actions in HA via an automation that reads the EMHASS output (see [Automation & Scheduling](/pages/automation.html)).
