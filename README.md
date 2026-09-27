# The Feeder as a Power Plant

**Track 8 — Avartan: Sustainability Ideathon**
Team **Idea_buzz** — Jenil Charadva, Dhruv Jodhani, Jay Patel

> A concept-stage feeder-level Virtual Power Plant (VPP) that coordinates the batteries and hybrid inverters homeowners already own — turning an 11 kV Gujarat feeder of 200–500 homes into a dispatchable, revenue-sharing grid asset, with no new hardware.

---

## 🔗 Live artifacts

| Artifact | Link |
|---|---|
| 🖥️ Pitch deck (8 slides) | [Open deck](https://claude.ai/artifact/BTYzemZbvN9fxsM5hmRWLq) |
| 📊 Interactive concept demo | [Open demo](https://claude.ai/artifact/RgBEd5piaqxqARW12PEEpk) |

The demo has four tabs: a duck-curve simulator, an animated process walkthrough of the full coordination pipeline, a dispatch/settlement calculator with live formula substitution, and a DISCOM value-case explorer.

---

## The problem

Rooftop solar penetration on Gujarat's low-voltage feeders is approaching the point where midday generation reverses power flow back into the grid, while evening demand still peaks the usual way. Thousands of homes on these feeders already own batteries and hybrid inverters — but each is tuned only for its own household's backup needs, invisible to the DISCOM, and uncoordinated with its neighbours.

**This is a coordination gap, not a hardware gap.**

We identify five concrete gaps and treat each as a design constraint with an actual mechanism, not a problem to defer:

| Gap | Description |
|---|---|
| **G1 — Visibility** | No feeder-level view of individual battery SoC, reserve setting, or generation |
| **G2 — Interoperability** | No common protocol across inverter brands |
| **G3 — Settlement** | No structure that pays homeowners without threatening their own backup power |
| **G4 — Trust** | No mechanism that makes homeowners comfortable ceding any control |
| **G5 — DISCOM value** | No quantified link between battery coordination and money saved |

---

## The concept

A closed-loop coordination architecture for a single 11 kV feeder:

```
Home batteries & inverters  →  Comms gateway (protocol translation → MQTT)
        │
        ▼
Feeder monitoring  ──▶  VPP orchestration engine (reserve-aware dispatch)
        │                         │
        ▼                         ▼
Dispatch to homes         DISCOM value tracking
        │                         │
        └───────────┬─────────────┘
                     ▼
          Settlement & metering
                     │
        (results feed the next optimisation cycle)
```

**Color legend:** Teal = home layer · Purple = coordination platform · Amber = DISCOM & settlement.

No new hardware is required — hybrid inverters already expose SoC, reserve level, and generation over a local interface. An edge gateway translates each vendor's native protocol into an open standard (IEEE 2030.5 / SunSpec Modbus) and republishes it over MQTT.

---

## The math

Every mechanism is a formula. Every formula has stated inputs — either measured, forecast from a named real source, or explicitly logged as a stated assumption.

**Reserve-aware dispatch floor** *(closes G3's core technical problem)*
```
ReserveFloor(i, t) = max( HardMin(i), BaseFloor(i) − k(i) × SolarConfidence(t+1) )
```
`HardMin(i)` is the homeowner's absolute minimum reserve and is **never** overridden — this is the trust guarantee (G4) made mechanical, not promised. `SolarConfidence(t+1)` is a next-day generation forecast confidence score from a real public source (IMD or Open-Meteo).

**Dispatchable capacity**
```
Dispatchable(i, t) = max( 0, SoC(i, t) − ReserveFloor(i, t) )
```
Recomputed every cycle from live SoC — never assumed static.

**Orchestration objective** *(Phase 1: rule-based — charge 10:00–16:00, discharge 18:00–22:00; Phase 2: replaced by a real LP/MPC solver)*
```
minimize   PeakNetLoad(feeder) over the 24-hour window
subject to  Dispatch(i,t) ≤ Dispatchable(i,t)  for all homes i, all hours t
```

**Settlement formula** *(closes G3's economic problem)*
```
Payout(i) = Σ_t [ kWh_dispatched(i, t) × BaseRate × StressMultiplier(t) ]
```
`BaseRate` = ₹4.15/kWh (stated placeholder, pending DISCOM/GERC negotiation). `StressMultiplier` is a stepped, explainable tier — 1.0× normal, 1.5× evening peak, 2.0× feeder stress — not a continuous market, so it stays auditable by a regulator and explainable to a homeowner in one sentence.

**Participation tiers** *(closes G4)*: Conservative / Balanced / Aggressive set `k(i)`, controlling how far the reserve floor can drop on a high-confidence forecast — always bounded so it never falls below `HardMin(i)`.

---

## DISCOM value case — two arguments, kept honestly separate

- **Loss reduction — real, but modest.** Gujarat DISCOMs already run ~8.86% aggregate technical & commercial loss (2019–20) vs. a 19.01% national average; Torrent Power reported 6.46% by FY21. Because the baseline is already efficient, this argument should not be oversold.
- **Deferred capex — the strongest ₹ case.** Reference transformer procurement rates (MSEDCL schedule, explicitly flagged as Maharashtra-sourced, illustrative only):

| Transformer size | Approx. cost |
|---|---|
| 315 kVA | ₹5.6–5.7 lakh |
| 630 kVA | ₹8.0–8.1 lakh |
| 995 kVA | ₹11.3 lakh |

If coordinated dispatch defers a specific transformer upgrade by N years, the avoided/deferred capex — discounted appropriately — is the order-of-magnitude ₹ value per feeder.

---

## Data & assumptions ledger

Every number used is tagged by status — nothing is presented without knowing where it came from:

| Item | Status |
|---|---|
| Gujarat T&D loss % vs. national average | Real, cited (published DISCOM/regulatory data, 2019–20) |
| Torrent Power FY21 distribution loss | Real, cited |
| Transformer costs (315/630/995 kVA) | Real, cited, but cross-state (MSEDCL, Maharashtra) |
| IEEE 2030.5 / SunSpec as interoperability standard | Real, existing standards (used in California DER programs) |
| GERC ties solar capacity to transformer capacity | Real, cited (GERC Fourth Amendment Regulations, 2024) |
| Reverse power flow shortens transformer life beyond a threshold | Real, cited (*Energies*, Vol. 15, Issue 23, 2022) |
| `solarGen(h)` / `houseLoad(h)` demo curves | Stated assumption, stylized (half-sine + three-Gaussian shapes) |
| Battery size (5 kWh usable), rooftop size (3 kWp) per home | Stated assumption |
| `BaseRate` ₹4.15/kWh | Stated placeholder |
| Charge/discharge windows (10:00–16:00 / 18:00–22:00) | Stated assumption, grounded in real solar/demand shape logic |

---

## Prior art — why this is still an open space

| Reference | What it does | Why it doesn't cover this case |
|---|---|---|
| Tata Power + AutoGrid, Mumbai | Behavioural + automated demand response, 75 MW peak reduction | Curtails load; does not dispatch stored energy |
| DEWA VPP pilot, Dubai | AI orchestration across mixed assets, ~3.3 MW | Utility-led, utility-owned assets |
| Next Kraftwerke, Germany | 5,000 units, >4,000 MW dispatchable | Mature market with existing flexibility-market infrastructure |
| CSTEP sector diagnosis, India | Identifies regulatory + comms-cost hurdles | Diagnosis, not a deployed solution |

None of the above targets a single 11 kV feeder, consumer-owned mixed-brand hardware, at 200–500 home scale — that's the space this project occupies.

---

## Roadmap

- **Phase 1 — Pilot (50–100 homes, single feeder):** comms gateways, rule-based dispatch, manual settlement, no DISCOM system integration yet.
- **Phase 2 — Feeder-scale (200–500 homes):** automated end-to-end settlement, SCADA/RTU integration, real LP/MPC optimizer replaces rule-based dispatch.
- **Phase 3 — Multi-feeder / district scale:** standardized replication, aggregate DISCOM-level capex planning.
- **Phase 4 — Regulatory formalization:** propose a GERC VPP framework using Phase 1–3 pilot data.

---

## Known risks (stated honestly)

- Cybersecurity of dispatch commands to home inverters — open, standard DER security practice not yet designed in detail.
- Reserve-confidence model is unvalidated against real feeder telemetry — Phase 1 pilot exists to test this.
- Homeowner trust/behavioral response to tiers is unproven — needs real user testing.
- Phase 1 dispatch logic is a deliberate simplification of the true optimizer.
- Transformer cost figures are cross-state (Maharashtra, not Gujarat) — a real pilot needs a Gujarat-specific quote.

---

## Repo contents

```
├── README.md                          — this file
├── Idea_buzz_Avartan_Track8_Progress_Report.pdf   — full written specification
├── feeder-vpp-pitch-deck.html         — pitch deck source (8 slides)
└── feeder-vpp-demo.html               — interactive concept demo source
```

---

## References

1. Tata Power, "Tata Power announces Demand Response Program with AutoGrid," 2023. tatapower.com
2. Dubai Electricity and Water Authority, "DEWA completes its pilot Virtual Power Plant project," 2024. mediaoffice.ae
3. CSTEP, "The Future of Virtual Power Plants in India, A Perspective," 2023. cstep.in
4. Gujarat Electricity Regulatory Commission, "Net Metering Rooftop Solar PV Grid Interactive Systems (Fourth Amendment) Regulations," 2024. gercin.org
5. "Impact of Reverse Power Flow on Distributed Transformers in a Solar Photovoltaic Integrated Low Voltage Network," *Energies*, Vol. 15, Issue 23, 2022.

---

*This is a concept-stage submission for an ideation hackathon. Every number here is either a cited public source or a labeled stated assumption — nothing is decorative.*
