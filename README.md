# QueueGuard — warnings that move with the queue

**FEIT Hackathon 2026 · RPM Hire challenge: End of Queue detection systems**

Roadworks queues grow upstream, often by several kilometres an hour. Warning signs are set out before the shift and stay put, so within minutes the queue tail sits behind them and drivers at 100 km/h meet stopped traffic unwarned. QueueGuard is the software layer that links radar, variable message signs (VMS), variable speed limit signs (VSL) and alerts so the warning follows the queue, plus a site guideline generator.

**Live demo:** [Open QueueGuard](https://shaohsuanw803-boop.github.io/queueguard/) · **Evaluation:** [View results](https://shaohsuanw803-boop.github.io/queueguard/eval.html) · **Site guideline generator:** [Plan a site](https://shaohsuanw803-boop.github.io/queueguard/guideline.html) · **Demo video:** [Watch the demo](https://shaohsuanw803-boop.github.io/queueguard/media/queueguard-demo.mp4) · **Source code:** [GitHub](https://github.com/shaohsuanw803-boop/queueguard)

**Final pitch (Team noname):** 23-slide [presenter deck](https://shaohsuanw803-boop.github.io/queueguard/pitch/) (four illustrated story slides after the cover; arrow keys; the 42 s demo clip plays on slide 11; N = notes, including the updated bilingual opening, T = timer, A = appendix) · [PDF](https://shaohsuanw803-boop.github.io/queueguard/docs/QueueGuard_final_pitch_Team_noname.pdf) · [clip](https://shaohsuanw803-boop.github.io/queueguard/media/queueguard-pitch-clip.mp4)

## Why this design

A 2025 Transport for NSW / Deakin University field trial of end-of-queue treatments (13 sites, 116 days) found that a commercial queue warning system and monochrome VMS had limited or no measurable effect, while colour VMS and flashing queue-warning signs reduced speeding. Manual UHF broadcasts helped; automatic ones did not. So detecting the queue is not the hard part. The warning has to be **in the right place, at the right time, and believable**. QueueGuard is built around that:

| Part | What it does |
|---|---|
| **Sense** | 20 s mean speeds from radar every 400 m. RPM Track, RPM's 4G platform for VMS, radar and CCTV data, is the natural source. The logic is sensor-agnostic. |
| **Predict** | Queued/free state with hysteresis; tail interpolated between detectors; tail velocity (shockwave) by least squares over 180 s, extrapolated 90 s |
| **Act** | First VMS upstream of *predicted tail − stopping distance* shows **STOPPED TRAFFIC / PREPARE TO STOP** (flashing); boards further upstream show **QUEUE AHEAD x KM**; VSL steps 100 → 80 → 60 (≤ 20 km/h per step); crew alert when a vehicle passes a radar within 450 m of the tail above 80 km/h (shown on the dashboard; phone delivery is pilot integration, and it is not counted in the results); UHF CB 40 message for trucks drafted automatically, **confirmed by the supervisor** |
| **Stay credible** | No queue message without a detected queue; escalate at once, relax only after 60 s; fail-safe: no data for 60 s → every board PREPARE TO STOP, VSL 80 |
| **Guide** | `guideline.html` turns site inputs into an end-of-queue plan: expected queue length and growth, device positions, message library, thresholds, alert roles, monitoring KPIs |

## What is new

Queue warning itself is proven, and commercial portable systems already exist, including for hire in Australia: on I-35 in Texas, an end-of-queue warning system cut work-zone crashes by up to 45% (TxDOT Waco District, via the Work Zone Safety Clearinghouse). RPM's own boards can already show live Mooven journey data. QueueGuard does not compete on detection. It is the control layer the RPM brief asks for, linking VMS, VSL and alerts, and it changes where, when and how believably the warning appears:

| | Typical queue warning (public descriptions) | QueueGuard |
|---|---|---|
| Warning position | fixed boards, switched when slow traffic is detected | board chosen from the tail predicted 90 s ahead |
| Warning timing | wherever a board happens to sit | into the 150–1,500 m alert window (not forgotten, not too late) |
| VMS, VSL and alerts | separate products | one controller, as the brief asks |
| Credibility | varies by system | no queue message without a detected queue; 60 s hold; fail-safe |
| Trucks | varies by system | UHF drafted, supervisor confirms (manual broadcasts worked in the NSW trial, automatic did not) |
| Before set-up | standard layout | site plan pre-tested with TfNSW metrics (`guideline.html` → *Run virtual trial*) |

In the simulation a fixed-position queue warning system gave only 6% fewer severe end-of-queue conflicts than static signs (not statistically different); QueueGuard gave 68% fewer (Results below).

## Viability

* **Business:** a per-site-week software add-on to VMS, VSL and radar kit RPM already hires. It lifts utilisation of those assets and gives Tier-1 clients safety data for their reporting.
* **Uptake:** no new hardware; VSL values only within the approved traffic management plan; a person confirms every UHF broadcast (the simulation assumes confirmation within 20 s); fail-safe to PREPARE TO STOP. Manual override of any sign is part of the pilot build.
* **Value:** the Texas system saved an estimated $1.4–1.8M in societal crash costs. A severe end-of-queue crash also stops the job.

## Results

Traffic microsimulation (Intelligent Driver Model, merging, driver distraction; see [docs/ASSUMPTIONS.md](docs/ASSUMPTIONS.md)). Every variant sees the same arrivals and the same drivers for a given seed. **Every sign has the same 60% chance of being noticed**, so QueueGuard gets no credit for colour or flashing, only for placement and timing. 40 seeds per variant (~43 simulated hours each); mean ± 95% CI.

| Variant | Severe EoQ conflicts /h | EoQ collisions /h | Arrival speed p85 (km/h) | Drivers warned in time | Queue time with no timely warning | Throughput (veh/h) |
|---|---|---|---|---|---|---|
| Static signs | 0.72 ± 0.33 | 0.23 | 67.5 | 78% | 32% | 1,113 |
| Fixed-position queue warning system | 0.67 ± 0.30 | 0.25 | 65.8 | 83% | 27% | 1,112 |
| **QueueGuard** | **0.23 ± 0.17** | **0.07** | **53.3** | **100%** | **0%** | **1,113** |
| QueueGuard without VSL | 0.46 ± 0.29 | 0.14 | 64.7 | 100% | 0% | 1,112 |
| QueueGuard without prediction | 0.37 ± 0.27 | 0.05 | 55.1 | 100% | 0% | 1,113 |

* Severe EoQ conflict = follower approaching a queued vehicle with TTC ≤ 1 s or DRAC ≥ 3.4 m/s², the serious-conflict thresholds used by TfNSW/Deakin.
* **−68% severe end-of-queue conflicts** and **−14 km/h** 85th-percentile arrival speed, with **no throughput or travel-time penalty** (715 s vs 714 s).
* **Sensitivity:** QueueGuard stays ahead of static signs when only 20% of drivers notice a warning, for recognition distances from 100–200 m to 250–450 m, and for peak demand of 1,600–2,000 veh/h (see `eval.html`).
* **Honest detail:** the 90 s tail prediction cuts "warning placed too late" from 10.9% to 6.7% of cases, but its point error (149 m) is no better than the current estimate (137 m), likely limited by the 400 m detector spacing and 20 s averaging (not varied in the experiments). Its value is moving warnings upstream while the queue grows.
* **Paired by seed** (every option sees the same drivers, `tools/paired.html`): QueueGuard − static = −0.48 severe conflicts per hour, 95% CI −0.85 to −0.12 (relative −68%, bootstrap 95% CI −29% to −90%); fixed-position − static = −0.05, 95% CI −0.39 to +0.30, no measurable difference. The two separate intervals in the table overlap slightly; the paired difference is the right test.
* **Known prototype limits:** only loss of all radar data at once is modelled (per-radar health checks come in the pilot); the site trial in `guideline.html` reuses the site's speed, demand and truck share on the standard closure, not its own geometry or sign layout.
* These are relative comparisons under stated assumptions, not a forecast of real crash reductions. That is what the pilot below is for.

## Path to deployment with RPM Hire

1. **Months 1–2, shadow mode:** run QueueGuard on RPM Track radar data from long-term sites with no control. Compare predicted tails with CCTV-trailer video. Tune thresholds.
2. **Months 3–4, site pilot:** live VMS/VSL control on 1–2 sites. Evaluate with the TfNSW/Deakin before–after method (radar speeds, video conflicts, driver survey).
3. **Then, hire add-on:** software on kit RPM already owns, plus the site guideline for the traffic management plan.

VSL values must stay within the approved traffic management plan and speed-zone authorisation. Manual override of every sign and limit by the site supervisor is part of the pilot build; the prototype already requires a person to confirm UHF broadcasts.

## Run it

No build step and no install. Open `index.html` in any modern browser, or use the [published site](https://shaohsuanw803-boop.github.io/queueguard/).

* `index.html` — side-by-side live simulation (static signs vs QueueGuard), roadside devices, time–space diagram, alerts, fail-safe toggle
* `eval.html` — full results; **Re-run in this browser** repeats the experiment
* `guideline.html` — site end-of-queue plan, printable to PDF
* `tools/evaluate.ps1` — headless batch evaluation (Microsoft Edge) → `results/`
* `tools/record2.ps1` — renders the demo video (Edge + ffmpeg); `tools/pitchclip.ps1` — the 42 s clip for the pitch
* `pitch/index.html` — the final pitch deck as a presenter page (built from the slide sources, clip from `media/`)

```
js/sim.js        traffic model, QueueGuard controller, metrics
js/render.js     canvas views
js/app.js        live dashboard
js/eval.js       experiments (variants, ablations, sensitivity)
js/guideline.js  site guideline generator
docs/ASSUMPTIONS.md
```

## References

* Transport for NSW / Deakin University, *Working Near Traffic – Work Zone End of Queue Study, Summary Report* (iMOVE project 1-064, April 2025).
* Austroads, *Guide to Temporary Traffic Management*, incorporated in Victoria's *Code of Practice for Worksite Safety – Traffic Management* (from 1 December 2023).
* Treiber, Hennecke & Helbing (2000), Congested traffic states in empirical observations and microscopic simulations, *Physical Review E* 62.
* Kesting, Treiber & Helbing (2007), General lane-changing model MOBIL for car-following models, *Transportation Research Record* 1999.
* Minderhoud & Bovy (2001), Extended time-to-collision measures for road traffic safety assessment, *Accident Analysis & Prevention* 33.

MIT licence.

Original project: [Morlan1hp/queueguard](https://github.com/Morlan1hp/queueguard). Copyright (c) 2026 QueueGuard team. This fork publishes the updated Team noname presentation.
