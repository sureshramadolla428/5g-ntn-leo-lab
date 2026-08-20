# D3c closeout verdict — 20260718T081005Z

| Stage | Result | Notes |
|-------|--------|-------|
| Part A evidence dir | `/home/USER/leo-evidence/diag_d3_green_dual_du_20260718T042659Z` | newest ~/leo-evidence/*d3* |
| Part A classification | **F1U_PATH_SWITCH** | DU1 heard UE and CU shows F1-U toward DU1, but D3c still failed ping/HO-complete — user-plane after path-switch. |
| Part B retry | FAIL | ran=1; no post-HO ping within 60s; ho_ok=0 mark=1 gtp_du1=1 tun=1 |
| **Verdict** | **BLOCKED** | FIXED only if retry PASS; else terminal + upstream draft |
| Upstream draft | yes | `UPSTREAM_ISSUE_DRAFT.md` |

## Part A excerpts

See `part_a_excerpts.txt` and `part_a_flags.txt`.

## Honesty

Closes the dual-DU SAT_LEO HO blocker as either **FIXED** (retry evidence) or **terminally documented** (classification + GitLab issue draft). Does **not** modify `make leo`, `pathc-bringup-du0.sh`, `oai-config/patches/`, or stock|_ref confs.

evidence_dir=/home/USER/leo-evidence/d3c_closeout_20260718T081005Z
