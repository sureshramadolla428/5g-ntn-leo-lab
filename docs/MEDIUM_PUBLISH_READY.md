# Medium article — Rel-17 5G NTN Emulation Lab

Ready-to-paste draft for Medium. Image markers use local paths under `4-DOCS-FINAL/medium-assets/`. Upload those files in Medium, then replace each marker with the uploaded image. Replace `[repo link]` before publishing.

---

## Title options

1. **How 5G Talks to Satellites: A Rel-17 NTN Emulation Lab** ← recommended
2. Emulating 5G Non-Terrestrial Networks End to End (Software-Only)
3. One State Vector, Every NTN Parameter: A Hands-On Rel-17 Lab

**Recommended title:** How 5G Talks to Satellites: A Rel-17 NTN Emulation Lab

**Subtitle:** A software-only OpenAirInterface lab: Rel-17 attach and data path measured over an emulated moving LEO channel, with a characterized RFsimulator multi-client topology constraint on dual-DU handover.

---

## Why I built this

5G Non-Terrestrial Networks (NTN) are among the more consequential additions to the standard in recent releases. With 3GPP Release 17, a UE can, in principle, attach to a base station hundreds of kilometres overhead and moving at roughly 7.5 km/s. I wanted to understand that from the stack, not from slides — by making a full attach and a real data packet work end to end.

I do not have a satellite, an SDR fleet, or a licensed S-band carrier. What I do have is [OpenAirInterface](https://openairinterface.org/) (OAI) and its **RFsimulator**: a channel emulator that carries IQ samples over TCP and can apply delay, Doppler, and drift. That is enough to stand up a complete 5G system and push traffic across an emulated moving-LEO link.

**Scope, stated plainly:** this is *emulation*. There is no over-the-air signal, no SDR, and no real orbit steering the radio. The useful claim is not “I built a satellite network.” It is: I built the Rel-17 NTN control loop in software, measured it with labelled provenance, and instrumented an RFsimulator multi-client topology constraint that places dual-DU handover under the green LEO role assignment out of scope for this RFsim topology.

[IMAGE: Rel-17 Space Network Emulation Lab infographic — 4-DOCS-FINAL/medium-assets/rel17-space-network-emulation-lab-hires.png]

---

## The stack

The lab mirrors a real CU/DU deployment:

- **OAI CN5G** — AMF, SMF, UPF, NRF, UDM/UDR in Docker. In this lab the core is essentially radio-agnostic: it is not given orbit, beam, or radio-timing state. (Real 3GPP NTN is not a pure-RAN story — the core does carry coarse NTN location and access awareness. More on that below.)
- **gNB with CU/DU split** — F1-C for signalling, F1-U for user data. The DU owns the radio.
- **nrUE** — the software UE.
- **RFsimulator** — between DU and UE, applying a moving-LEO channel (`SAT_LEO_TRANS`) with propagation delay and Doppler.

On the green Path C bring-up, the DU is the RFsim *server* and the UE is the *client*. Bring-up order matters (core → CU → DU → UE). Launcher scripts keep the NTN flags and log handling reproducible.

Configured radio for the Path C numbers below: **band 254 (~2.4884 GHz, S-band MSS), ~600 km LEO, elevation mask ~10°, 25 PRB at 15 kHz SCS (µ=0)**. Timing and Doppler figures are derived for *this* configuration — not universal NTN constants.

When the session is up, the UE registers, establishes a PDU session, gets an IP on `oaitun_ue1`, and ICMP across the emulated satellite lands in the **~25–80 ms** RTT class (often ~25–45 ms; verified runs around ~32 ms average with 0% loss). That latency comes from RFsim’s channel model — roughly `prop_delay` ~20 ms plus the motion term — **not** from live SGP4 driving the softmodem. Orbit-inject into RFsim is **off**. The Grafana map and model KPIs share SGP4 geometry as a teaching/telemetry plane; that geometry does not steer measured ping.

[IMAGE: End-to-end architecture — CN5G → CU/DU → RFsim → nrUE — 4-DOCS-FINAL/medium-assets/architecture.png]

[IMAGE: Session / green data path evidence — 4-DOCS-FINAL/medium-assets/session.png]

---

## One state vector, every parameter

A terrestrial 5G stack assumes a nearby static cell: small timing advance, negligible Doppler, short HARQ round-trips, tight RRC timers. A ~600 km moving LEO link breaks all of those assumptions. Rel-17 NTN is largely the set of fixes:

- **SIB19** broadcasts satellite **ephemeris** (position and velocity).
- **Common Timing Advance** (`ta-Common-r17`, plus drift) pre-loads the large, changing path delay — about **18.8 ms** for Path C.
- **Cell-specific K-offset** (`cellSpecificKoffset_r17`) = **40** slots (~40 ms at 15 kHz SCS).
- **Doppler pre-compensation** — about **+57.3 kHz** for this band-254 / ~600 km geometry (`--initial-fo 57340`), with continuous compensation through the pass.
- **Extended HARQ** — **32** processes.
- **Stretched RRC timers** (t300/t301/t319) so the UE does not abandon the attempt before the satellite round-trip returns.

These are not independent knobs. From the Path C *design* state vector (configured ECEF position/velocity for a ~600 km LEO), slant range → delay → common TA, and radial velocity → Doppler → initial FO. The numbers check out arithmetically against that configured vector. That is the Path C design plane — not a claim that the day-of Grafana SGP4 map is writing softmodem parameters.

Architecturally: the **RAN and UE own the radio physics** (timing, Doppler, ephemeris). In this lab the core runs without orbit knowledge. Per 3GPP (TS 23.501 and related NTN material), the 5GC is not a total blackout either — earth-fixed tracking areas, NTN user-location information, AMF location verification, access identification, and related restrictions exist. Clear statement: radio intelligence lives in RAN/UE; the core stays radio-agnostic while keeping coarse location and access awareness.

[IMAGE: RF pass / LEO KPI panels — drop your day-of Grafana RSRP·SINR·ping snap here if sharper than stock; stock: 4-DOCS-FINAL/medium-assets/rf-pass.png]

---

## Observability with provenance labels

A lab you cannot see into is a lab you cannot trust. Custom exporters turn OAI logs and host state into Prometheus metrics, with Grafana panels for RSRP, SINR, BLER, MCS, throughput, delay, and Doppler.

The rule that matters more than the dashboard polish: **label every metric by provenance.**

- **Measured** — host ICMP ping, container/interface state.
- **Simulator / log** — RF quantities parsed from softmodem logs (simulator-domain, not OTA).
- **Orbit model** — SGP4 elevation, range, delay, Doppler geometry.
- **Model / keyword / config** — offline models, static baselines, log-keyword counters.

Without those labels, a clean RSRP curve reads like a field measurement. With them, the demonstration stays credible.

**Coverage honesty for the day-of map (ISS TLE + NYC UE):** when elevation falls below ~10°, the lab marks **RF OOS**. That can coexist with **Ping UP** and a stale service state (tunnel still present while RF KPIs are gated). Treat RF zeros below the mask as expected policy, not a silent bring-up fault — and do not conflate OOS RF with a dead U-plane.

[IMAGE: Grafana NTN KPI board — 4-DOCS-FINAL/medium-assets/grafana-ntn-kpi.png]

[IMAGE: Grafana orbit / topology plane — 4-DOCS-FINAL/medium-assets/grafana-ntn-orbit.png]

---

## AIOps: physics residual, not magic thresholds

Static latency thresholds fit terrestrial cells poorly under LEO geometry. The lab’s AIOps layer computes an *expected* RTT from orbit geometry (roughly twice one-way delay plus a protocol floor) and watches the **residual** against measured host ping.

A growing residual is a useful anomaly signal *when both planes are healthy*. When the orbit plane and the measured Path C ping diverge — or when RF is OOS — treat the residual as a hedged diagnostic, not ground truth that “SGP4 is driving the radio.” Orbit-inject is still off; expected and measured come from different planes by design.

There is also a closed-loop remediator path (chaos inject via `tc netem`, anomaly classification, optional CU CI `trigger_f1_ho`). **Default is dry-run:** it logs intent and does not actuate. Live actuation requires an explicit `--arm` flag, with debounce/cooldown. That path has been exercised in documented sync-loss fullpass evidence under AWGN-style lab conditions. It is **lab CI inject**, not 3GPP measurement-report-based mobility, and it does not remove the SAT_LEO dual-DU RFsimulator topology constraint described below.

[IMAGE: AIOps residual / ops panels — 4-DOCS-FINAL/medium-assets/aiops.png]

[IMAGE: Super-ops / NTN vs baseline board — 4-DOCS-FINAL/medium-assets/grafana-ntn-lab-super-ops.png]

---

## Seeing the orbit — and keeping it honest

An SGP4 map shows the sub-satellite point, ground UE, and a UE → satellite → gNB → core diagram. Elevation, slant range, one-way delay, and Doppler on that view are teaching/telemetry. Same geometry family as the model KPIs; **not** the driver of softmodem or measured ping.

Beside the live NTN numbers sits a terrestrial comparison column. That baseline is **Diona: a static, labelled CSV/example reference**, not a live Diona scrape. It frames the gap. It is not measured NTN data wearing a different colour.

[IMAGE: Live topology / geomap teaching plane — 4-DOCS-FINAL/medium-assets/topology.png]

[IMAGE: Topology dashboard mockup (optional alternate) — 4-DOCS-FINAL/medium-assets/ntn-live-topology-dashboard.png]

---

## Fault injection: proving the parameters matter

Configuring SIB19, TA, K-offset, and FO is necessary but not sufficient as evidence. I ran break-tests that deliberately poison one Rel-17 parameter at a time and check that the observed outcome matches the procedure the specs describe (mapped through a local clause index):

- **Wrong common TA / K-offset** → RACH fails / no RAR (observable against TS 38.321 §5.1; TA/K-offset via SIB19 in TS 38.331 and timing in TS 38.213).
- **Wrong initial FO** → SSB/PBCH does not decode; cell search fails (TS 38.211 / 38.213; ephemeris in TS 38.331).
- **Disable continuous FO mid-pass** → sync degrades as Doppler walks away.
- **GEO-class delay during registration** → NAS timer stress (TS 24.501 §10.2). This is a stress demonstration, not a claim that I reproduced an exact T3510 expiry via a core-side timer change I did not make.

These are RFsimulator measurements — sync/RACH detail lives in softmodem logs, not air captures. Cases the software-only lab cannot force stay labelled BLOCKED.

Evidence pack: `4-DOCS-FINAL/medium/fault-injection-test-cases/`

---

## Characterized limit: dual-DU HO under RFsim multi-client topology

A dual-DU F1 handover — UE moving from one cell to another as one satellite sets and another rises — was a natural next measurement after Path C attach and ping.

Under the reliable recipe — **UE as RFsim server, DUs as clients, AWGN** — F1 handover completed cleanly (`handover … complete` in the logs).

Under the green LEO Path C topology — **DU0 as RFsim server; UE and DU1 as clients** — over the live moving-LEO channel, handover **did not complete**. The core triggered HO and F1-U moved toward the target DU, but the UE never acquired the target cell and the data path dropped.

A logging-only instrumentation patch in RFsim’s sample-mixing path (counters only; no behaviour change) showed why: the RFsimulator server is **hub-and-spoke**. It sums each client’s uplink into the *host* receive path and returns only the *host* transmit to each client. It **never forwards one client’s samples to a sibling client**. Peer-mix counters confirm the target cell’s IQ never reaches the UE. Correct F1/RRC cannot compensate for a missing radio sample.

That is a **characterized RFsimulator multi-client topology constraint** — a tool/scope finding, not a claim that F1 handover itself is broken or that the Path C lab failed. Dual-DU HO under these green LEO roles is **out of scope for this RFsim topology**. The reproducible package — environment, steps, counter evidence, logging patch — lives in the share kit as `OAI_ISSUE_RFSIM_DUAL_DU_HO`. It is **prepared for upstream** (paste-ready for OAI’s tracker if/when filed). It is **not filed** yet; do not treat it as an open upstream ticket.

Publishing the instrumented finding with counters is more useful than a handover demonstration that only succeeds when the topology avoids the multi-client case.

---

## What I deliberately did not claim

Out of scope — and not claimed:

- **Over-the-air / SDR** — RFsimulator only.
- **Onboard regenerative gNB** — transparent / bent-pipe model.
- **Native ATSSS / MA-PDU** — approximated only.
- **Live store-and-forward** — Rel-19-style geometry model, not a running OAI feature.
- **Map “handover” pulses** — geometry markers, labelled as such, not radio handovers.
- **SGP4 driving measured ping** — inject off; planes are separate.
- **Diona as live terrestrial feed** — static labelled baseline only.

---

## What I would tell someone starting the same work

- Emulation is a legitimate way to learn a standard deeply — if you are precise about what is and is not real.
- NTN is mostly a timing and frequency problem: geometry → delay → TA; radial velocity → Doppler → FO.
- Instrument before you theorise. The dual-DU finding came from sample counters, not from guessing at RRC alone.
- Provenance labels on every metric are practical insurance against the one demonstration mistake that costs credibility.
- Separate teaching planes (SGP4 map, model KPIs) from the measured U-plane — and say so clearly when RF goes OOS under the elevation mask while ping is still up.

The code, configs, dashboards, and the dual-DU RFsim issue kit are in the project share/repo: 👉 [repo link]

If you work on NTN, 5G RAN, or satellite integration and want to compare notes on Rel-17 timing, RFsim topology, or labelled observability, I am glad to talk.

---

### Publishing notes (not for Medium body)

- Prefer the hires infographic as the hero image; use `rel17-space-network-emulation-lab.png` if Medium size limits prefer the smaller file.
- If you have a fresher day-of Grafana ping/RSRP capture than `rf-pass.png` / `session.png`, drop it into `medium-assets/` and point the markers there.
- Optional extras already copied: `grafana-ntn-lab-ops.png`, `ntn-stats.png`, `leo-vs-geo.png`, `ntn-canvas.png`, `ntn-geomap.png`.

---

*Built on OpenAirInterface (open-source). Personal research and education project — software emulation, not an over-the-air system — not affiliated with or endorsed by any operator or vendor. Comparison-baseline figures in screenshots are illustrative unless explicitly labelled measured.*
