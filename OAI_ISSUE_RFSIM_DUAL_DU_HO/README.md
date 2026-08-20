# OAI issue kit — RFsim dual-DU F1 HO / peer-mix

**What this is:** one self-contained folder to **file (or re-verify before filing)** an OpenAirInterface issue about dual-DU F1 handover under multi-client RFsimulator: F1-U path-switch works, but the UE never syncs target PCI 1 because the RFsim **server never forwards sibling-client TX to other clients**.

**Filing status:** **Ready for GitLab if you choose.** Not filed. Do not claim already upstream.

**Where to file:** [gitlab.eurecom.fr/oai/openairinterface5g](https://gitlab.eurecom.fr/oai/openairinterface5g/-/issues)  
The GitHub repo is a **mirror** — do **not** file there by default.

**Supersedes:** `github-issue-rfsim-dual-du-ho/` (older partial kit). Prefer this folder.

---

## Start here checklist

1. [ ] Read [`HONESTY.md`](HONESTY.md) (scope + what not to claim).
2. [ ] Skim [`ISSUE.md`](ISSUE.md) — paste-ready body for GitLab.
3. [ ] On the lab VM: install scripts from the **private companion** `5g-ntn-leo-lab-scripts` (`OAI_ISSUE_RFSIM_DUAL_DU_HO/reproduce/` there). See [`reproduce/README.md`](reproduce/README.md).
4. [ ] Reproduce: [`REPRODUCE.md`](REPRODUCE.md) (`~/` paths; green Path C → `d3c-closeout` → optional `p1`).
5. [ ] Package: run `collect-evidence.sh` on the VM → `issue_evidence_<ts>.tar.gz` in this folder.
6. [ ] Follow [`VERIFY_AND_CAPTURE.md`](VERIFY_AND_CAPTURE.md) — scrub once, then create GitLab issue + attach tarball.
7. [ ] Offer the logging-only patch from the private companion (`OAI_ISSUE_RFSIM_DUAL_DU_HO/patches/`) — mention happy to open an MR if useful.

---

## Layout

```
OAI_ISSUE_RFSIM_DUAL_DU_HO/
  README.md                 # this file
  ISSUE.md                  # paste into GitLab
  REPRODUCE.md              # VM steps (~/ paths)
  VERIFY_AND_CAPTURE.md     # capture → pack → file
  HONESTY.md                # scope / limits
  .gitignore                # ignore *.tar.gz
  evidence/                 # static matrices + counter writeup
  reproduce/                # markdown excerpts; *.sh live in private companion
  # scripts + logging patch: sibling repo 5g-ntn-leo-lab-scripts
```

---

## Three commands (after kit is on the VM)

```bash
# 1) Copy public kit markdown + private scripts to VM — adjust host path / guest IP
scp -r OAI_ISSUE_RFSIM_DUAL_DU_HO user@VM:~/
# Scripts: from sibling 5g-ntn-leo-lab-scripts (same relative paths)

# 2) On VM: install reproduce scripts, then closeout (+ optional peer-mix proof)
cp -a ~/OAI_ISSUE_RFSIM_DUAL_DU_HO/reproduce/*.sh ~/oai-config/path-c/
bash ~/oai-config/path-c/d3c-closeout.sh
# optional strongest evidence:
# bash ~/oai-config/path-c/p1-instrumented-rerun.sh

# 3) On VM: pack evidence into this folder, then paste ISSUE.md on GitLab
bash ~/OAI_ISSUE_RFSIM_DUAL_DU_HO/collect-evidence.sh
# → ~/OAI_ISSUE_RFSIM_DUAL_DU_HO/issue_evidence_<ts>.tar.gz
```

Pin: OAI **2026.w16** (`38dc378`). Scratch develop used for counters: **`31ffb21a82`**.
