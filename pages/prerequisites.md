---
layout: default
title: Prerequisites
---

# Prerequisites

1. **Home Assistant** up and running (Container, Supervised, or OS).
2. **Amber Electric integration** in Home Assistant.
   - If an official integration is available in your HA version, add it via **Settings → Devices & Services → Add Integration → Amber Electric**.
   - Otherwise install via **HACS**: search for *Amber Electric* and follow the repo instructions.
3. **Battery/charger/inverter entities** available in HA (power, state-of-charge, charge/discharge services).
4. Optional but recommended: **Solcast** or other PV forecast integration in HA.
5. A machine to run **EMHASS** (can be the same box). Docker is easiest.

> ⚠️ Entity names differ between setups. In the config examples we use placeholders like `sensor.amber_import_price`. Replace them with your actual entity IDs from **Developer Tools → States**.
