Despite its daunting sophistication at first glance, EMHASS is quite simple at its core. You feed in 4 forecast values and it will spit out an optimal strategy for a given solar system, including when to charge and discharge your battery. Those 4 forecasts are:

1. A forecast of how much you can sell electricity for (i.e. feed-in price).
2. A forecast of how much you can buy electricity for.
3. A forecast of how much solar you will generate.
4. A forecast of how much electricity you will use.

<p align="center">
  <img src="/assets/img/emhass_diagram.png" alt="EMHASS Architecture Diagram" width="800">
</p>

EMHASS will take all this information and generate an energy plan. This plan is published to Home Assistant in the form of new sensor entities, which automations can then use to control your battery (e.g. via the Sigenergy modbus integration).

You can also visualize this energy plan in a dashboard and see what EMHASS has decided to do:

<p align="center">
  <img src="/assets/img/3_day_plan.png" alt="EMHASS Plan Example" width="800">
</p>

Here is a 3-day plan
