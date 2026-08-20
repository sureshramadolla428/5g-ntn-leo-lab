# D3 develop retest result — 20260718T200935Z

| Item | Value |
|------|-------|
| Host | USER-virtual-machine |
| Scratch tree | `/home/USER/oai-develop-scratch` |
| Build commit | `31ffb21a8204ae9706a88eb08606a80fc8eafb3e` (`develop`, merge `integration_2026_w29`) |
| Build command | `./build_oai --ninja --nrUE --gNB -w SIMU --build-lib telnetsrv` (binaries **reused**; `SKIP_BUILD=1`) |
| Build OK | 1 |
| Pinned lab | `/home/USER/openairinterface5g` @ `38dc378224d230a50a787f3d8e7d460314fcf770` (`2026.w16`) |
| Pinned untouched | **YES** (sha256 of `nr-softmodem` / `nr-uesoftmodem` unchanged) |
| Conf dir | `/home/USER/oai-config/path-c-devel` (not stock `path-c`) |
| Evidence | `/home/USER/leo-evidence/d3_develop_20260718T200935Z` |
| D3a (green baseline) | **PASS** |
| D3b (DU1 F1 Setup) | **PASS** |
| D3c (HO + PCI-1 sync + UP) | **FAIL** |
| Overall | **FAIL** |
| Path C restore (pinned binaries) | **PASS** |

## Verdict vs item 1

**Not fixed on develop.** Same D3c failure mode as pinned 2026.w16 / `38dc378` (and prior develop run `20260718T191417Z`):

| Signal | Develop @ `31ffb21a` (this run) |
|--------|------------------------------|
| HO triggered toward PCI 1 | Yes (`ho_trig=1`) |
| F1-U Update tunnel → `127.0.0.5:2153` | Yes (`gtp_du1=1`) |
| CU `marking HO as complete` | Yes (`ho_mark=1`) — **not** authentic success |
| Authentic `handover for UE … complete!` | **No** (`ho_ok=0`) |
| UE `PCI: 1` / target sync | **No** (`ue_pci1=0`; post-HO `synch Failed` loop) |
| Post-HO ping | **FAIL** (tunnel gone after UE segfault during HO; `SO_BINDTODEVICE oaitun_ue1: No such device`) |

## Honesty

- Scratch binaries reused @ `31ffb21a8204ae9706a88eb08606a80fc8eafb3e`; **no rebuild**; **does not modify** the pinned 2026.w16 lab / no `make leo`.
- D3c PASS requires UE **PCI 1 sync** evidence **and** post-HO ping — not CU `marking HO as complete` alone.
- UE NTN flags verified against scratch `doc/ntn-configuration.md` + `nr-uesoftmodem --help` (`flag_verify.txt`): `--prop_delay 20`, `chanmod`, `--time-sync-I 0.1`, `--ntn-initial-time-drift -46`, `--initial-fo 57340`, `--cont-fo-comp 2`.
- Upstream: **still present on develop @ 31ffb21a8204ae9706a88eb08606a80fc8eafb3e** — [`docs/UPSTREAM_ISSUE_DRAFT.md`](../UPSTREAM_ISSUE_DRAFT.md) ready to file.
- SUMMARY item 1 stays **terminal / upstream** (not FIXED).

## Notes

- D3c FAIL: `ue_pci1=0 ping_ok=0 ho_ok=0 ho_mark=1 gtp_du1=1` (CU mark alone ≠ PASS)
- During D3c the scratch `nr-uesoftmodem` hit **Segmentation fault** after HO trigger (additional radio failure signal; not counted as PASS).
- Prior FAIL evidence: `~/leo-evidence/d3_develop_20260718T191417Z/` — this run is the fresh authoritative retest.

## Log crumbs

See `run_D3*_snippet.txt`, `flag_verify.txt`, and `run_D3*_*.log` under `~/leo-evidence/d3_develop_20260718T200935Z`.

evidence_dir=/home/USER/leo-evidence/d3_develop_20260718T200935Z
