# Lab Progress & KPI Formulas

**Audience:** presentations, operators, newcomers  
**Purpose:** One keepable summary of (1) what this lab has delivered and (2) how Grafana numbers are calculated.  
**Sources of truth:** [`NTN_KPI_FORMULAS_IMPLEMENTED.md`](NTN_KPI_FORMULAS_IMPLEMENTED.md) · [`KPI_CATALOG.md`](KPI_CATALOG.md) · [`LAB_OVERVIEW_WHAT_HOW_GRAFANA_HONESTY.md`](LAB_OVERVIEW_WHAT_HOW_GRAFANA_HONESTY.md) · [`LAB_POLICY.md`](LAB_POLICY.md) · [`monitoring/ntn/config/ntn.yaml`](../monitoring/ntn/config/ntn.yaml)  
**Exporters (read-only reference):** `grafana/ntn_kpi_exporter.py` · `monitoring/orbit_csv_exporter.py`

**Lab one-liner:** Software 5G SA NTN (LEO-style) on OAI + RFsim + CN5G + Prometheus/Grafana. Proven Path C attach + ping. Not OTA, not SDR, not a real satellite ground station.

---

## Part A — What we have done so far

### Path C reliability
- Pinned OAI softmodems (~**2026.w16** / `38dc378`) with RFsim; Path C band **254**, CU + DU0, UE as RFsim client.
- One-script bring-up (`pathc-bringup-du0.sh` / `make leo`): CN5G → CU → DU0 → UE → `oaitun_ue1` → ping DN **192.168.70.135**.
- **Day-of capture:** `bash ~/demo/start-live-capture.sh` (monitoring + SGP4 → pathc ping gate → golive → restore SGP4 → background iperf → Sionna/Diona). See [`DEMO_PLAYBOOK.md`](DEMO_PLAYBOOK.md).
- Proven U-plane: **OK: PING SUCCESS**, typically **~25–80 ms** RTT (often ~25–45), **0% loss**. **Demo success = Ping UP.**
- AWGN dual-DU F1 HO is the honest HO demo; **SAT_LEO dual-DU F1 HO complete (D3c) = FAIL** — do not invent PASS. GEO attach on this pin = FAIL (errno 14).

### KPI physics / geometry / capacity
- Single geometry source: TR 38.821 spherical slant range (no competing linear demo formula).
- Single RF chain: elev/range → FSPL → L_extra → RSRP / C/N₀ → SINR → MCS / BLER / capacity.
- Named constants in `monitoring/ntn/config/ntn.yaml` (elev mask **10°**, carrier **2.4884 GHz**, EIRP/G/T, UE NYC lat/lon).
- Shared libs: `monitoring/ntn/link_budget.py`, `coverage.py`, `consistency.py`.
- TS 38.214-style **model capacity** @ **N_PRB=25** separate from **measured** MAC/oaitun/iperf throughput.
- Consistency gates: `budget_consistency_ok`, `consistency_ok`, forbidden rosy margin vs bad SINR.

### Honesty / OOS
- Coverage: `elev ≥ 10°` = in coverage; below = **OOS** (`rf_oos=1`).
- When OOS: live capacity/MCS/BLER gated to 0 (or BLER=1), `link_margin=0`; geometry + link-budget gauges stay as teaching models — **expected** with ISS TLE + NYC UE, not a bring-up failure.
- `service_state`: 0=OOS, 1=PREDICTION (U-plane down), 2=UP_OK, 3=ZOMBIE (**oaitun present while RF OOS**). Can show ZOMBIE while **ping still works** — distinguish from classic dead U-plane (oaitun + 100% ping loss).
- CQI/RSRQ and many pcap-derived latencies are **NOT EXPORTED** unless truly measured.
- Orbit inject into RFsim is **OFF** — map does not steer the radio.
- Diona = synthetic baseline (`diona_baseline.prom`, `source=example`). Live Topology = SGP4 model map, not OTA GPS.

### Grafana dashboards
- **Lab Demo** (`ntn-full-lab`) — primary teaching / demo tiles.
- **Ops / Super-ops** (`ntn-lab-super-ops`) — ops + NTN vs Diona / AIOps.
- **Live topology** (`ntn-live-topology`) — sat/UE map from SGP4 geometry.
- Runtime compose **only** from `~/monitoring` (never empty VMware hgfs under OneDrive).
- Datasource UID (Grafana 13): **`PBFA97CFB590B2093`**. Guest IP often `.128`, can flap to `.131`.

### SGP4 / orbit
- Orbit exporter `:9101` prefers `--sgp4` + `monitoring/tle/sample_leo.tle` (public **ISS ZARYA** sample — replace for your sat).
- `start-live-capture` / day-of: after golive (`--demo` orbit), **re-apply SGP4** via `restart_exporters.sh`.
- Publishes lat/lon/alt, elev, slant, delay, Doppler, pass timing; tagged `ntn_orbit_source{source="sgp4"}`.
- Geometry plane is **separate** from Path C RFsim channel.

### Sionna / Diona
- Optional offline Sionna NTN study → textfile `sionna_ntn_*` (predictions, not live softmodem RF).
- Diona panels = **CSV baseline** via `diona_baseline_exporter` → `diona_*` textfile — **no live Diona scrape**.

### Call-flow
- Live signalling ladder optional on `:8787` (`make callflow-live`); iframe on Lab Demo.
- Ordered UL/DL docs for Path C (NG → F1 → RACH → RRC → NAS → PDU → ping).

### Docs / runbooks
- Presentation overview, step-by-step bring-up/cleanup, live ping/iperf/Sionna, **demo playbook**, operator day-of guides, KPI catalog/policy/formulas, deep-dive pack, documentation index.

### Disk / NAT lessons (hard-won)
- **VMware NAT Service** on Windows must be Running — otherwise Docker pulls / guest DNS fail while the VM looks “up.”
- Guest DHCP can flip **`.128` ↔ `.131`** — wrong IP looks like a dead lab.
- Do **not** start Grafana compose from `/mnt/hgfs/...` (often empty under OneDrive) — wiped dashboards historically.
- Disk full kills bring-up; use cleanup runbook (safe Docker prune) before long campaigns.
- Prefer `~/oai-config`, `~/monitoring`, `~/demo` on the VM; Windows tree is an editing/scp mirror.
- Do **not** run long `bash -lc '…pkill -f ntn_kpi…'` one-liners (can self-kill). Background iperf from `start-live-capture` survives Ctrl+C — kill via `/tmp/iperf-client.pid` or `stop-all`.

### Open TODOs (honest)
| Item | Status |
|------|--------|
| Full **RFsim inject** from model geometry | Deferred — compare only; do not break Path C |
| Full **REST API** (Step 10) | Stub / skip |
| Full **RACH/RRC** collectors as always-on Grafana | Placeholder NOT-EXPORTED |
| Attach / PDU latency from **pcap** as live gauges | Deferred / offline evidence |
| Complete artifact pack runner | Optional skeleton later |
| SAT_LEO dual-DU F1 HO (D3c) | Known FAIL (AWGN HO works) |
| UE as RFsim server + SAT_LEO; GEO on pin | BLOCKED / FAIL on pin |
| CQI / RSRQ | NOT-EXPORTED |
| Replace sample ISS TLE with operator sat | Operator action |
| Long iperf can drop ping | Use TCP `-P 1`; avoid huge windows / UDP flood |

---

## Part B — How Grafana gets numbers

Short pipeline:

```
TLE/SGP4 + ntn.yaml UE lat/lon
        │
        ▼
  orbit exporter :9101  ──►  elev, range, delay, Doppler, sat lat/lon, pass timing
        │
        ▼  (same-cycle mirror)
  KPI exporter :9300   ──►  FSPL, RSRP, SINR, capacity, ping, honesty gauges
        │                    (+ softmodem logs, ICMP via oaitun)
        ▼
  Prometheus :9090
        │
        ▼
  Grafana :3000  (Lab Demo · Ops · Live topology)
```

Also scraped: node-exporter `:9100` (+ textfile: Sionna, Diona, lab_ops, pcap stamps), cAdvisor, blackbox, optional callflow `:8787`.

**Key constants** (`ntn.yaml`): `elev_min_deg=10`, `f_hz=2.4884e9`, `eirp_dbm=110`, `n_prb=25`, `scs_khz=15`, `nf_db=7`, `h_sat_km=600`, `initial_fo_hz=57340`, UE ≈ (40.71, −74.01).

---

## Part C — Formula / source table for each major KPI

| KPI (dashboard name) | Metric name | Formula or collector | Source tag | Notes |
|----------------------|-------------|----------------------|------------|-------|
| Elevation | `ntn_ue_elevation_deg` / `fiveg_ntn_elev_deg` | SGP4 look angle (or demo/CSV); KPI mirrors orbit | **SGP4** / model | Mask **10°**; elev &lt; mask ⇒ OOS |
| Elev mask | `fiveg_ntn_elev_mask_deg` | Config `coverage.elev_min_deg` | **config** | Default 10.0 |
| Slant range | `ntn_range_km` / `fiveg_ntn_slant_range_model_km` | \(d=\sqrt{R_E^2\sin^2\varepsilon+h^2+2h R_E}-R_E\sin\varepsilon\) [km]; SGP4 uses geometric range | **SGP4** / model | Single geometry source; KPI mirrors orbit |
| One-way delay | `ntn_oneway_delay_ms` / `fiveg_ntn_model_oneway_delay_ms` | \(\tau=d/c\) (\(c=299792.458\) km/s); KPI mirrors orbit from **same scrape** as slant | **model** | Dashboard prefers KPI family with slant |
| Model RTT | `fiveg_ntn_model_rtt_ms` / `rtt_expected_ms` | \(2(d_\mathrm{svc}+d_\mathrm{feeder})/c + t_\mathrm{proc}\); feeder @ ε=35° | **model** | Transparent payload teaching |
| Doppler | `ntn_doppler_hz` / `fiveg_ntn_doppler_hz_model` / `*_khz` | \(f_d=(v_r/c)\,f_c\) | **model** / SGP4 | \(f_c=2.4884\) GHz Path C |
| Doppler rate | `fiveg_ntn_doppler_rate_hz_s` | \(\dot f_d=\Delta f_d/\Delta t\) (proxy if first sample) | **model** | Computed even when OOS |
| Residual FO | `fiveg_ntn_residual_fo_hz` | \(f_d - f_\mathrm{comp}\); residual ≈ −gain·rate (clamped ≈±900 Hz) | **model** | Not the static `doppler_minus_initial_fo` |
| Compensated / initial FO | `fiveg_ntn_compensated_fo_hz`, `*_doppler_minus_initial_fo_hz` | FO tracker vs `--initial-fo` (57340 Hz) | **model** / config | Do not call static offset “residual after compensation” |
| FSPL | `fiveg_ntn_fspl_db` | \(20\log_{10}(d_\mathrm{km})+20\log_{10}(f_\mathrm{GHz})+92.45\) | **model** | ITU / link-budget |
| EIRP | `fiveg_ntn_eirp_dbm` | Config `link_budget.eirp_dbm` (default 110) | **config** | Tuned for ≈−70…−100 dBm RSRP |
| G/T | `fiveg_ntn_gt_dbk` | \(G_\mathrm{rx}-10\log_{10}(T_\mathrm{sys})\); \(T_\mathrm{sys}=T_0\cdot\mathrm{NF_{lin}}\) | **model** / config | \(T_0=290\) K, NF=7 dB |
| C/N₀ | `fiveg_ntn_cn0_dbhz` (+ `cn0_recomputed`) | \(\mathrm{EIRP_{dBm}}-30-\mathrm{FSPL}-L_\mathrm{extra}+G/T+228.6\) | **model** | Same \(L_\mathrm{extra}\) as RSRP |
| L_extra | `fiveg_ntn_l_extra_db` | \(L_\mathrm{atm}+L_\mathrm{other}+L_\mathrm{pattern}\); pattern \((90-\varepsilon)\times0.25\) dB | **model** | Defaults 0.5 + 3.0 + pattern |
| RSRP | `fiveg_ntn_rsrp_dbm` | \(\mathrm{EIRP}-\mathrm{FSPL}-L_\mathrm{extra}+G_\mathrm{rx}-10\log_{10}(12 N_\mathrm{PRB})\) | **model** | Default ON from live elev |
| SINR / SNR | `fiveg_ntn_sinr_db` / `snr_db` | \(\mathrm{RSRP}-N_0\); \(N_0=-174+10\log_{10}(BW)+\mathrm{NF}\) | **model** | Display clamp [−5, 40] dB |
| Sync margin | `fiveg_ntn_sync_margin_db` | \(\mathrm{SINR}-\mathrm{SINR_{req}}\) (default req=0 dB) | **model** (heuristic) | Recomputed every scrape; not 3GPP KPI |
| Link margin | `fiveg_ntn_link_margin_db` | \(\mathrm{SINR_{raw}}-\mathrm{SINR_{req}}(\mathrm{MCS})\); **0 if OOS** | **model** | Forbidden: in_cov & margin&gt;10 & SINR&lt;0 |
| MCS DL/UL | `fiveg_ntn_serving_mcs_{dl,ul}` | Monotonic SINR→MCS map (lab / TS 38.214 spirit) | **model** | Zeroed when OOS |
| BLER | `fiveg_ntn_bler_*` | \(\approx 0.18\,e^{-\mathrm{SINR}/7}\) (clamped) | **model** | Gated when OOS |
| Capacity DL/UL (ceiling) | `fiveg_ntn_{dl,ul}_capacity_mbps` | \(\mathrm{RE/s}\cdot\mathrm{eff(MCS)}\cdot(1-0.14)/10^6\) at **config \(N_\mathrm{PRB}=25\)** | **model** | **0 when OOS**; MCS19→≈10.94 Mbps |
| Alloc DL/UL (optional) | `fiveg_ntn_{dl,ul}_alloc_mbps` | Same math at log `serving_nprb` | **model** | **NOT EXPORTED** if no serving NPRB; never replaces capacity tiles |
| Capacity prediction | `fiveg_ntn_dl_capacity_prediction_mbps` | Same TS 38.214 math @ 25 PRB, always computed | **model** | Use when U-plane down (`PREDICTION`) |
| Shannon (theoretical) | `fiveg_ntn_shannon_capacity_mbps` | \(BW\cdot\log_2(1+\mathrm{SINR_{lin}})\) | **model** | Labelled theoretical |
| Measured thr DL/UL | `fiveg_ntn_{dl,ul}_throughput_mbps` | MAC byte deltas; host `oaitun_*_mbps` when ping_up; optional iperf textfile | **measured** | Tiles prefer measured only if ping_up & thr&gt;0.05 |
| Spectral efficiency | `fiveg_ntn_spectral_efficiency_bps_hz` | \(\eta=\mathrm{capacity_{Mbps}}\cdot10^6/BW_\mathrm{Hz}(25\,\mathrm{PRB})\) | **model** | Same N_PRB as capacity ceiling |
| In coverage | `ntn_in_coverage` / `fiveg_ntn_in_coverage` | elev ≥ mask (and prefer ephemeris_valid) | **model** / policy | |
| RF OOS / in service | `fiveg_ntn_rf_oos` / `rf_in_service` | `rf_oos=1-in_coverage` | **model** / policy | |
| Time to LoS | `ntn_time_to_los_s` | Seconds until elev drops below mask; **0** if already OOS | **model** | Name = Loss of Signal, not “to Line of Sight” |
| Pass duration | `ntn_pass_duration_s` | Current above-mask window if in-pass; else **next** pass length | **model** | Must be &gt;0 when next_pass_eta&gt;0 |
| Next pass ETA | `ntn_next_pass_eta_s` | Seconds to next rise above mask (after current pass if in-pass) | **model** | Never “0 because already in-pass” |
| Ping UP / RTT / loss | `fiveg_ntn_ping_up`, `ping_rtt_ms`, `ping_loss_pct` | ICMP via `oaitun_ue1` → DN; reject RTT &lt; 5 ms | **measured** | Session-works truth |
| oaitun present | `fiveg_ntn_oaitun_*` / related | Host tunnel interface check | **measured** | Alone ≠ in-pass service (zombie trap) |
| Attach latency | `attach_latency_ms` | t(Reg Accept) − t(Reg Req) | **pcap** | **NOT EXPORTED** live by default |
| PDU setup | `pdu_setup_ms` | t(PDU Accept) − t(PDU Request) | **pcap** | **NOT EXPORTED** live by default |
| Ephemeris validity / age | `fiveg_ntn_ephemeris_validity_s`, `ephemeris_age_s`, `_valid` | Config `ntn-UlSyncValidityDuration-r17` else **180 s** default; valid=(age≤validity) | **config** / model | |
| Service state | `fiveg_ntn_service_state` | 0=OOS, 1=PREDICTION, 2=UP_OK, 3=ZOMBIE (oaitun while RF OOS) | **policy** | May be ZOMBIE while ping UP — ≠ dead U-plane |
| Model tput scope | `fiveg_ntn_model_tput_scope` | 0=live measured, 1=prediction only, 2=OOS zero | **policy** | |
| Session outside coverage | `fiveg_ntn_session_outside_coverage` | oaitun=1 while in_coverage=0 | **policy** | Zombie / RF-OOS flag |
| Sat lat / lon / alt | `ntn_satellite_lat/lon`, `ntn_satellite_alt_km` | SGP4/TLE subpoint | **SGP4** | Not OTA GPS |
| UE lat / lon / alt | `ntn_ue_lat/lon`, `ntn_ue_alt_km` | `ntn.yaml` geometry | **config** | Default NYC-ish |
| TLE age / orbit source | `ntn_tle_age_days`, `ntn_orbit_source` | Days since TLE epoch; label sgp4\|demo\|csv | **SGP4** / config | Refresh TLE when age grows |
| Carrier / PRB context | `fiveg_ntn_carrier_freq_hz` | Config `f_hz` | **config** | Must stay 2.4884e9 for Path C |
| Consistency OK | `fiveg_ntn_budget_consistency_ok`, `consistency_ok`, `provenance_epoch_align` | Identity recompute / hard checks / same-cycle mirror | **model** | Teaching integrity |
| Sionna predictions | `sionna_ntn_pred_rtt_ms`, `pred_snr_db`, `pred_bler`, `dry_run` | Offline Sionna study → node textfile | **model** | Optional; dry_run=1 = toy |
| Diona baseline | `diona_*` | CSV → `diona_baseline_exporter` textfile | **config** / offline | Orange panels; not live scrape |
| Softmodem MCS/BLER/RSRP (log) | various log-derived when present | Softmodem log scrape | **measured** (sim radio) | Useful demo; **not** field RF |
| CQI / RSRQ | — | — | **not exported** | Unless truly in logs |

---

## Part D — True vs model one-liner for presentations

**Green ping proves the emulated LEO 5G data path on OAI + RFsim + CN5G. Grafana mixes measured session health (ping, tunnel, host/CN) with honest physics models (SGP4 geometry + link budget + TS 38.214 capacity). The map does not steer the radio; this is not an OTA satellite campaign.**

| Say “true / measured” | Say “model / teaching” |
|-----------------------|-------------------------|
| Ping UP, ping RTT/loss via oaitun | Sat lat/lon/alt, elev, slant, delay, Doppler |
| oaitun / CN5G / host CPU | FSPL, EIRP, G/T, C/N₀, RSRP, SINR, margins |
| iperf when you ran it | DL/UL capacity prediction, spectral eff, MCS/BLER from SINR map |
| Softmodem process alive | Residual FO / compensated FO chain |
| | Sionna preds + Diona CSV baseline |

---

## Quick pointers

| Need | Doc |
|------|-----|
| Full formula derivations | [`NTN_KPI_FORMULAS_IMPLEMENTED.md`](NTN_KPI_FORMULAS_IMPLEMENTED.md) |
| Metric catalog | [`KPI_CATALOG.md`](KPI_CATALOG.md) |
| OOS / inject / TODO policy | [`LAB_POLICY.md`](LAB_POLICY.md) |
| Stakeholder overview | [`LAB_OVERVIEW_WHAT_HOW_GRAFANA_HONESTY.md`](LAB_OVERVIEW_WHAT_HOW_GRAFANA_HONESTY.md) |
| Bring-up commands | [`LAB_BRINGUP_STEP_BY_STEP.md`](LAB_BRINGUP_STEP_BY_STEP.md) · [`DEMO_PLAYBOOK.md`](DEMO_PLAYBOOK.md) |
| Consistency check | `bash demo/verify-consistency.sh` |

*Docs only — exporters unchanged. Synced to lab policy / formula docs as of 2026-07-30.*
