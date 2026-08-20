# Path C excerpts (relevant to this issue)

Source lab doc: `docs/PATH_C_NTN_F1_HANDOVER.md` (full file in the main repo).

## Verified recipes vs this bug

| Verified | Recipe |
|----------|--------|
| LEO radio attach | **DU = RFsim server** — `pathc-bringup-du0.sh` / `make leo` + `SAT_LEO_TRANS` → ping OK |
| Dual-DU F1 HO (reliable) | **UE = server**, DUs = clients, **AWGN** — `pathc-du1-ho.sh` / `make path-c-ho` → `handover … complete!` |

| Known limit | Status |
|-------------|--------|
| D3c dual-DU F1 HO under live `SAT_LEO` with **DU0 = server** | **FAIL** — this issue (F1-U OK; no PCI 1) |
| UE-as-server + `SAT_LEO` SSB / dual-DU HO | **BLOCKED** (separate from peer-mix) |

## RFsim roles (why AWGN HO ≠ D3c)

| Mode | RFsim roles | Channel | Notes |
|------|-------------|---------|-------|
| LEO attach / D3c green | **DU = server**, UE (+ DU1) = client(s) | `SAT_LEO_TRANS` | Sibling client TX never reaches UE → D3c FAIL |
| Dual-DU F1 HO (lab demo) | **UE = server**, DUs = clients | **AWGN** | UE host sums DU clients → HO can complete |

## Ports

Prom **9090**, telnet HO **9099**, F1-U **2153**, N3 **2152**.

## Commands (VM)

```bash
# Green LEO attach
bash ~/oai-config/path-c/pathc-bringup-du0.sh

# Reliable F1 HO (different recipe — not the D3c failure)
bash ~/oai-config/path-c/pathc-du1-ho.sh
# expect: OK: PATH C F1 HO COMPLETE (channel=AWGN)

# D3c closeout (this issue)
bash ~/oai-config/path-c/d3c-closeout.sh
```
