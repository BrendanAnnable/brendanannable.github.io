---
layout: default
title: FAQ
---

# FAQ

**Q: Do I need direct Amber API keys?**  
Not if you already have Amber sensors in HA. EMHASS can read whichever import/export price sensors you expose.

**Q: How often should I optimize?**  
15 minutes is a good start (Amber prices can change frequently). Increase frequency during volatile periods.

**Q: Can I include demand charges / block tariffs?**  
Yes—extend the objective and constraints in EMHASS or pre-process price streams into an **effective price** entity that includes all components.

**Q: What if I have multiple batteries or phases?**  
Model aggregate constraints (sum power caps) or run separate optimizations with coordination logic in HA.
