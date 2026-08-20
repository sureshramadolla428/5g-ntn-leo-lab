# Honesty — scope and limits for this issue

Read before filing. Keeps the report accurate and useful to OAI maintainers.

## In scope

- **Software-only RFsim** (no SDR / OTA claim).
- Dual-DU **F1** HO under multi-client RFsim with **DU0 as server**, UE + DU1 as clients.
- Symptom: CU/F1-U advance; **UE never syncs PCI 1**; post-HO ping dies.
- Root cause: **measured** star topology — server never forwards **client→sibling** TX; UE RX has **0** DU1-origin samples before and after HO.
- Logging-only instrumentation offered (`patches/0001-rfsim-peer-mix-counters.patch`).

## Explicitly out of scope / do not claim

| Topic | Honest statement |
|---|---|
| PHY air-interface bug | **Not claimed.** This is RFsim multi-peer sample mixing / topology. |
| F1-U / GTP broken | **False.** Path-switch to DU1 `:2153` is verified. |
| Already filed upstream | **Not filed.** Kit is ready for GitLab **if the user chooses**. |
| Live LEO dual-DU HO “almost works” | D3a/D3b PASS, **D3c FAIL** — do not soft-pedal. |
| AWGN dual-DU HO | **PASSES** only with **UE-as-server** recipe (`pathc-du1-ho.sh` / `make path-c-ho`). Different topology. |
| UE-as-server + `SAT_LEO` | Separate **PRACH/SSB** blocker (bisect B0–B7); not the peer-mix ask. |
| SGP4 / Grafana inject into RFsim | Lab inject is **OFF** — unrelated to this bug; do not imply orbit inject in the issue. |
| Rank-2 sibling relay experiments | Lab prototypes exist elsewhere; **stock** behavior (no relay) is what this issue reports. |
| Pin tree modifications | Peer-mix patch is **scratch-only** — never apply to the green LEO pin used for demos. |

## Filing status note

Older lab docs may say **DISCARDED** for a prior GitHub-filing attempt. That means “not pursued on GitHub,” not “invalid science.”  
**This kit:** content is current; status = **ready for GitLab if user chooses**. Do **not** claim already filed.

## Ask (one sentence)

Confirm whether multi-client RFsim is intentionally hub-and-spoke only, and whether first-class sibling DL mix is in scope for dual-DU F1 HO with DU-as-server.
