# Demo playbook — live Grafana capture (day-of)

**Audience:** operator presenting Lab Demo / Ops / Live Topology  
**One command (Ubuntu VM):** `bash ~/demo/start-live-capture.sh`  
**Repo twin:** `demo/start-live-capture.sh`  
**SSH:** `sureshramadolla@192.168.122.128` (DHCP may flap to **`.131`**) · key `id_ed25519_ntn`  
**Grafana:** `admin` / `admin` · Prometheus datasource UID (Grafana 13): **`PBFA97CFB590B2093`**

Companion: bring-up [`LAB_BRINGUP_STEP_BY_STEP.md`](LAB_BRINGUP_STEP_BY_STEP.md) · honesty [`LAB_OVERVIEW_WHAT_HOW_GRAFANA_HONESTY.md`](LAB_OVERVIEW_WHAT_HOW_GRAFANA_HONESTY.md) · open gaps [`NTN_OPEN_ITEMS_STATUS.md`](NTN_OPEN_ITEMS_STATUS.md)

---

## 0) Preflight (Windows + Ubuntu)

| Check | Pass when |
|-------|-----------|
| VMware **NAT Service** Running | Docker / DNS work on guest |
| VM IP | `arp -a \| findstr 192.168.122` — use live address for browser/SSH |
| Disk | `df -h /` — prefer ≥5G free (ideal ~13G) |
| Runtime trees | Use **`~/monitoring`**, **`~/oai-config`**, **`~/demo`** — not empty hgfs |

```powershell
# Windows — discover IP, then SSH
arp -a | findstr 192.168.122
ssh -o IdentitiesOnly=yes -i "$env:USERPROFILE\.ssh\id_ed25519_ntn" sureshramadolla@192.168.122.128
```

**Do not** run long `bash -lc '…pkill -f ntn_kpi…'` one-liners — the pattern can match/kill the shell itself. Prefer `~/demo/stop-all.sh` or name-only softmodem kills.

---

## 1) Start (one command)

```bash
bash ~/demo/start-live-capture.sh
```

Skip clean stop if RAN/core already good:

```bash
SKIP_STOP=1 bash ~/demo/start-live-capture.sh
```

### What the script does (in order)

1. Optional **STOP** (`STOP_RAN=1 STOP_CORE=1` + softmodem/iperf kill) unless `SKIP_STOP=1`
2. **Monitoring + SGP4** — `ensure-monitoring-up` + `restart_exporters` (`:9101`, TLE `~/monitoring/tle/sample_leo.tle`)
3. **Path C** — `~/oai-config/path-c/pathc-bringup-du0.sh` — **gates on `OK: PING SUCCESS`** (ping DN `192.168.70.135` via `oaitun_ue1`)
4. **golive-and-verify** — then **re-applies SGP4** (golive starts orbit in `--demo`; script restores SGP4 for Live Topology)
5. **Background iperf** ~300 s (`IPERF_T`, default 300) — TCP `-P 1`, DL `-R`
6. **Sionna** textfile + **Diona** baseline (`source=example`)
7. Prints Grafana / call-flow URLs

**Ctrl+C** stops *this* script only — **background iperf keeps running**. Kill later:

```bash
kill "$(cat /tmp/iperf-client.pid)" 2>/dev/null || true
# or: bash ~/demo/stop-all.sh
```

---

## 2) What to open (say aloud)

| Dashboard | URL (replace VM IP) | Speech |
|-----------|---------------------|--------|
| **Lab Demo** | `http://<VM>:3000/d/ntn-full-lab` | End-to-end story; **Ping UP** is the proof |
| **Ops / NTN vs Diona** | `http://<VM>:3000/d/ntn-lab-super-ops` | Ops + orange Diona = **example CSV**, not live NTN RF |
| **Live Topology** | `http://<VM>:3000/d/ntn-live-topology` | **SGP4/TLE model map** (ISS sample + UE from `ntn.yaml` ~NYC) — not OTA GPS |
| Call-flow | `http://<VM>:8787/` | Optional ladder |
| Prometheus | `http://<VM>:9090/targets` | Scrape health |

---

## 3) Demo success speech (critical)

| Criterion | Truth |
|-----------|--------|
| **Demo success** | **Ping UP** to `192.168.70.135` via `oaitun_ue1` (~**25–80 ms**, often ~25–45) |
| **RF green** (RSRP/SINR/MCS/capacity live tiles) | Only when **elev ≥ ~10°** (in coverage). Below mask → `rf_oos=1`, MCS/capacity gated to OOS zeros — **expected**, not bring-up failure |
| **ISS + NYC** | Sample TLE + NYC UE often **below 10°** → OOS panels while ping still works |
| **Capacity vs measured** | Capacity ceiling @ **N_PRB=25** (model); measured thr needs iperf |
| **Diona** | Synthetic baseline (`diona_baseline.prom`, `source=example`) |
| **Map** | Model geometry — inject into RFsim is **OFF** |

**Stakeholder sentence:** *“Green ping proves the emulated LEO 5G data path. Grafana mixes measured session health with SGP4 + link-budget models. RF tiles go dark when the model sat is below the elevation mask — that is honesty, not a dead lab.”*

---

## 4) Mid-demo recovery

| Symptom | Fix |
|---------|-----|
| Softmodems + `oaitun` exist, **100% ping loss** | Classic **zombie** tunnel — `STOP_RAN=1 STOP_CORE=1 bash ~/demo/stop-all.sh` then `bash ~/oai-config/path-c/pathc-bringup-du0.sh` (or full `start-live-capture`) |
| Grafana `service_state=ZOMBIE` but ping works | Model label: **oaitun present while RF OOS** (elev &lt; mask). Distinct from dead U-plane — trust **measured ping** for “session works” |
| Long iperf drops ping | Stop iperf (`/tmp/iperf-client.pid`); keep TCP, `-P 1`; avoid huge windows / UDP flood; rerun pathc if needed |
| Blank panels | `bash ~/monitoring/ensure-monitoring-up.sh`; hard refresh; check `:9090` / `:9300` / `:9101` |
| Wrong browser IP | DHCP `.128`↔`.131` — re-check `hostname -I` / arp |

---

## 5) Do **not** claim in this demo

- D3c dual-DU F1 HO under **SAT_LEO** (AWGN HO works — separate recipe)
- UE as RFsim server + SAT_LEO; GEO air attach on pin **2026.w16**
- RFsim inject from SGP4; full REST KPI API; always-on RACH/RRC collectors
- Live CQI/RSRQ (NOT-EXPORTED)
- Diona / Live Topology as OTA truth

---

## Related

- Full day-of capture detail: [`LAB_LIVE_TEST_PING_IPERF_SIONNA.md`](LAB_LIVE_TEST_PING_IPERF_SIONNA.md)
- Ops stop/start variants: [`../demo/OPERATIONS_RUNBOOK.md`](../demo/OPERATIONS_RUNBOOK.md)
- Legacy multi-terminal softmodem stages (obsolete for day-of): archived narrative in git history of `DEMO_RUNBOOK.md` — prefer this playbook
