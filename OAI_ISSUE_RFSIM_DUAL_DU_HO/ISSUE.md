<!-- Paste this as the GitLab issue body.
     Tracker: https://gitlab.eurecom.fr/oai/openairinterface5g/-/issues
     Do NOT file on the GitHub mirror by default.
     Status: ready to file if the reporter chooses — not yet filed. -->

**Suggested title:**  
`[nrUE/rfsim] Dual-DU F1 HO: F1-U path-switch works but UE never syncs PCI 1 (multi-client hub-and-spoke; no sibling TX mix)`

---

## Summary

In a CU + two-DU F1 split over the RFsimulator, an F1 handover from DU0 (PCI 0) to DU1 (PCI 1)
signals correctly and the F1-U GTP tunnel is switched to DU1, but the **nrUE never acquires/syncs the
target cell (PCI 1)**, so the user plane dies (100% packet loss) even though the tunnel stays up.

Instrumented counters show the root cause: with the **gNB/DU acting as the RFsim server**, the
server mixes each client's uplink into the **host's** RX only and sends the **host's** TX to each
client — **a sibling client's downlink is never forwarded to another client**. The target DU's SSB
therefore never reaches the UE, so no amount of RRC/F1 signalling can complete the radio handover.

This reproduces on pinned tag **2026.w16 (`38dc378`)** and on **develop (`31ffb21a82`)**.

## Environment

| Item | Value |
|---|---|
| OAI RAN | **2026.w16** (`38dc378`) **and** develop (`31ffb21a82`) — both fail identically |
| Radio | RFsimulator only (no SDR) |
| Band / numerology | Band 254, µ=0 (15 kHz SCS), 25 PRB (NTN Path C recipe; peer-mix also shown under AWGN) |
| Green LEO topology | CU + **DU0 = RFsim server** + `SAT_LEO_TRANS` + **DU1 = RFsim client** + **nrUE = client** |
| Split | F1 (F1-C SCTP + F1-U GTP-U `:2153`) |
| HO trigger | CU telnet `ci trigger_f1_ho` on `:9099` |
| OS | Ubuntu 22.04 lab VM |

## Contrast: AWGN dual-DU HO that PASSES (different recipe)

| Recipe | RFsim roles | Channel | Result |
|---|---|---|---|
| **Green LEO / D3c (this issue)** | **DU0 = server**; UE + DU1 = clients | `SAT_LEO_TRANS` (also fails under AWGN with same roles) | F1-U OK; **no PCI 1 sync** |
| **Lab HO demo (works)** | **UE = server**; DU0 + DU1 = clients | **AWGN** | `handover … complete!` + ping |

UE-as-server AWGN HO works because the UE **host** sums all DU clients into its own RX (it can hear PCI 1). That is a **different topology**, not a fix for DU-as-server multi-client mixing. UE-as-server + live `SAT_LEO` has a separate PRACH/SSB blocker (bisect B0–B7) and is out of scope for this ask.

## What works (this is NOT an F1-U / GTP bug)

- **Baseline PASS**: CU + DU0 + UE → `oaitun_ue1` up, ping to DN OK.
- **DU1 joins PASS**: DU1 connects as an RFsim client and completes **F1 Setup**.
- **F1-U path-switch VERIFIED**: on HO, the CU updates the tunnel to DU1 (`127.0.0.5:2153`), DU1
  creates the matching TEID, and `ss` shows DU1 bound on `:2153`. → “F1-U stuck on DU0” is **refuted**.
- Same radio failure under AWGN when roles stay DU0=server / UE+DU1=clients (rules out “only the moving channel”).

## Symptom (after `ci trigger_f1_ho`)

| Signal | Seen? | Meaning |
|---|---|---|
| CU `Handover triggered … PCI 1` | Yes | CU starts F1 HO |
| F1-U GTP re-pointed to DU1 `:2153` | Yes | path-switch works |
| UE `Configuring CRNTI …` + RRCReconfigurationComplete | Yes | UE accepted HO reconfig |
| DU1 `Added new CFRA process` | Yes | target armed CFRA |
| `handover for UE … complete!` | **No** | true HO success never logged |
| UE log `PCI: 1` / target sync | **No** | **UE never acquires PCI 1** |
| DU1 sees UE `in-sync` | **No** | target radio never hears the UE |
| Post-HO ping | **No** | 100% loss — tunnel up, radio dead |

Closeout classification: **F1U_PATH_SWITCH**; retry **FAIL**.

## Root cause — measured, not inferred

A **logging-only** patch on the scratch develop tree instrumented the RFsim multi-peer path
(`rfsimulator_read_internal` / `rfsimulator_write_internal`). During HO, counters show:

```
RFSIM_PEER_MIX_SUMMARY role=SERVER nconn=2
    host_rx = sum_of_peer_cirbufs
    client_out = host_tx_only
    peer_cirbuf_never_forwarded_to_sibling_clients
```

With DU as server:

- each client's UL is summed into the **server host's** RX (DU0 hears the UE) ✔  
- the server sends only the **host's** TX to each client (UE hears DU0) ✔  
- **client→client is never mixed**, so the UE's RX contains **zero** samples originating from DU1,
  **before and after** the HO.

Measured: `UE RX has DU1-origin samples = No` (before HO) / `No` (after HO).

## Reproduction (green Path C roles)

```bash
# OAI 2026.w16 (or develop) built with rfsim + telnetsrv; CN5G up.
# Prefer ~/ paths (do not rely on empty hgfs mounts).

# 1) Green baseline: CU + DU0 (RFsim server + SAT_LEO) + UE (client)
bash ~/oai-config/path-c/pathc-bringup-du0.sh   # expect oaitun_ue1 + ping

# 2) Start DU1 as an RFsim CLIENT to DU0's server (PCI 1)
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E env LD_LIBRARY_PATH="$PWD" ./nr-softmodem --rfsim \
  -O ~/oai-config/path-c/gnb-du1.band254.ntn.pci1.conf \
  --rfsimulator.[0].serveraddr 127.0.0.1
# wait for "F1 Setup Response" in du1 log

# 3) Trigger F1 handover
echo 'ci trigger_f1_ho' | nc -w 2 127.0.0.1 9099

# 4) Observe: F1-U → DU1, no "PCI: 1" sync, no true HO complete, post-HO ping = 100% loss.

# One-shot closeout (optional):
bash ~/oai-config/path-c/d3c-closeout.sh
# Peer-mix counter proof (scratch develop + logging patch):
# bash ~/oai-config/path-c/p1-instrumented-rerun.sh
```

## Questions for maintainers

1. Is the RFsimulator server intended to be strictly **star / hub-and-spoke** (host↔client only,
   no client↔client mixing)? The counters suggest yes; confirming would make this a documented
   limitation rather than an unexpected bug.
2. If so, is a **first-class multi-peer downlink mix** (so a client can receive a sibling client's
   TX) in scope? That is what dual-DU F1 HO needs on the DU-as-server topology.
3. Is there a supported way to do dual-cell F1 HO under RFsim today that we have missed
   (besides UE-as-server + AWGN)?

## Attachments

- Peer-mix counter writeup + `RFSIM_PEER_MIX` / `_SUMMARY` lines  
- Green dual-DU matrix (D3a/D3b PASS, D3c FAIL) + closeout verdict  
- PRACH bisect matrix (B0–B7) — separate UE-as-server LEO PRACH context  
- Logging-only patch: `0001-rfsim-peer-mix-counters.patch` (scratch develop; SHA256
  `4d56f97aeb583d5279bf0ac750652257e16cd722d0b327ec476c97801c5eb1eb`)  
- CU / DU0 / DU1 / UE logs from a failing run (in evidence tarball)

Happy to share the logging patch as an MR if useful for maintainers re-measuring.

---

*Scope: software-only RFsim reproduction. We do not claim a PHY bug in the air interface — this is a measured characterization of multi-client sample mixing under hub-and-spoke RFsim.*
