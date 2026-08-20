# reproduce/ — scripts live in the private companion

This **public** folder keeps markdown only ([`PATH_C_EXCERPTS.md`](PATH_C_EXCERPTS.md)).

Bring-up and closeout **scripts** (`pathc-bringup-du0.sh`, `d3c-closeout.sh`, `p1-instrumented-rerun.sh`), `collect-evidence.sh`, and the logging-only patch are in the sibling private repo:

`../5g-ntn-leo-lab-scripts/OAI_ISSUE_RFSIM_DUAL_DU_HO/`

Copy those onto the Ubuntu OAI VM under **`~/oai-config/path-c/`**. Prefer `~/` over hgfs.

```bash
mkdir -p ~/oai-config/path-c ~/oai-config/patches/scratch-only
cp -a ~/OAI_ISSUE_RFSIM_DUAL_DU_HO/reproduce/*.sh ~/oai-config/path-c/
cp -a ~/OAI_ISSUE_RFSIM_DUAL_DU_HO/patches/*.patch ~/oai-config/patches/scratch-only/
chmod +x ~/oai-config/path-c/*.sh
```

| Script | Role |
|---|---|
| `pathc-bringup-du0.sh` | Green LEO baseline: CU + DU0 (RFsim **server** + `SAT_LEO`) + UE client |
| `d3c-closeout.sh` | One-shot D3a→D3b→D3c closeout + verdict under `~/leo-evidence/d3c_closeout_*` |
| `p1-instrumented-rerun.sh` | Scratch develop + **logging-only** peer-mix counters (P1); restores pin afterward |

**Notes**

- Scripts may still reference a lab share root (`SHARE` / `SHARE_ROOT`). If hgfs is empty, either set `SHARE_ROOT` / `PATCH_DIR` to kit or `~/oai-config` paths, or rely on the `$HOME/oai-config/path-c` copies after install.
- `p1-instrumented-rerun.sh` must **never** patch `~/openairinterface5g` (the pin). Scratch only: `~/oai-develop-scratch`.
- Full Path C context (AWGN HO vs LEO attach): see [`PATH_C_EXCERPTS.md`](PATH_C_EXCERPTS.md) in this folder.

**Run order for filing** (after private scripts are on the VM)

```bash
bash ~/oai-config/path-c/d3c-closeout.sh
# optional:
bash ~/oai-config/path-c/p1-instrumented-rerun.sh
bash ~/OAI_ISSUE_RFSIM_DUAL_DU_HO/collect-evidence.sh
```
