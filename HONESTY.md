# Honesty — Rel-17 NTN LEO lab (this GitHub pack)

Read this before citing numbers, dashboards, or handover results.

## What this is

Software-only **5G SA NTN (LEO-style)** emulation: OAI CU/DU + nrUE + **RFsimulator** + OAI CN5G + Prometheus/Grafana.

- **Path C green recipe:** band **254**, DU0 = RFsim **server**, UE = RFsim **client**, channel `SAT_LEO_TRANS`.
- **Verified U-plane:** ping via `oaitun_ue1` to **192.168.70.135**, typically **~25–80 ms** RTT (often ~25–45 ms), **0% loss** on green runs.
- **Ping source:** SAT_LEO **RFsim** delay/Doppler — **not** SGP4-driven ping.
- **Orbit inject into RFsim:** **OFF**. Grafana map/model KPIs are a **teaching/telemetry plane**.

## Rel-17 radio (this Path C design)

| Item | Honest value |
|------|----------------|
| SIB19 | Used (ephemeris broadcast) |
| Common TA | **~18.8 ms** |
| K-offset | **40** |
| Initial FO | **~57 kHz** (~57340 Hz for this band-254 / ~600 km vector) |
| Band | **254** (~2.4884 GHz) |
| HARQ | **32** |

These numbers belong to **this** configuration. They are not universal NTN constants.

## Grafana

| Dashboard | Trust |
|-----------|--------|
| **Lab Demo** | Ping UP / RTT = **measured**. Many RF tiles = log/simulator/**model**. |
| **Live Topology** | SGP4/TLE geometry — **not** OTA GPS, **not** steering the radio. |
| **Super Ops** | Ops + residual AIOps. **Diona is static** (labelled example), not a live terrestrial feed. |

**RF OOS** when elevation **&lt; 10°** can **coexist with Ping UP** (tunnel still present; RF KPIs gated). That is policy, not a silent Path C failure.

## Dual-DU handover

| Recipe | Result | How to say it |
|--------|--------|----------------|
| AWGN, **UE-as-server**, DUs as clients | **PASS** | Honest HO demo for this RFsim topology |
| Green LEO, **DU0-as-server**, UE + DU1 as clients | **FAIL** | RFsim is **hub-and-spoke**: no client→sibling IQ. **Tool/scope limit, not a project failure.** |

F1-U/path-switch toward the target DU can still look healthy while the UE never syncs PCI 1. See `OAI_ISSUE_RFSIM_DUAL_DU_HO/`. The kit is **ready to file** if you choose; it is **not filed**.

## Explicitly not claimed

- Over-the-air / SDR / licensed carrier / real satellite ground station
- Live SGP4 writing softmodem delay/Doppler or measured ping
- Diona as live measured terrestrial NTN
- Green-LEO dual-DU F1 HO complete
- Onboard regenerative gNB, native ATSSS, or Rel-19 store-and-forward as running features

## Language to use

- **Measured** — host ICMP, tunnel presence, containers, iperf when actually run
- **Model / log / simulator** — SGP4, FSPL chain, softmodem-parsed RF
- **Tool limit** — RFsim multi-client mix under DU0-as-server

Personal research and education. Not affiliated with or endorsed by any operator or vendor.
