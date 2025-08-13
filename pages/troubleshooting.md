---
layout: default
title: Troubleshooting
---

# Troubleshooting

## Prices look wrong (units / GST)
- Confirm whether your Amber sensors are in **$/kWh** or **c/kWh** and whether they include **GST**. Convert consistently (see template example).
- Cross-check with Amber's app at the same timestamp.

## Battery doesn't follow schedule
- Verify the control **services** (scripts) actually affect the inverter.
- Check that **SoC limits** and **power caps** are correct in `emhass_conf.yaml`.
- Ensure the optimization **resolution** matches how often you apply actions (e.g., 15 minutes).

## EMHASS returns empty or errors
- Look at container logs: `docker logs -f emhass`.
- Verify HA URL/token reachability from the container.
- Temporarily simplify config (remove forecasts) to isolate issues.

## Forecasts missing
- Confirm Solcast entities exist and update regularly.
- If forecasts are not available, EMHASS can still optimize with current prices and constraints, but results may be less accurate.

## Loops or flapping behavior
- Add **minimum dwell time** between mode changes in your HA automation.
- Add deadbands around thresholds (e.g., don't toggle around 10 c/kWh; use 9 and 11 as boundaries).
