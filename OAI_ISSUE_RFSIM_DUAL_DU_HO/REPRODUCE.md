# Reproduce — dual-DU F1 HO peer-mix failure

All steps run on the **Ubuntu OAI lab VM**. Use **`~/` paths**. Do **not** depend on `/mnt/hgfs/...` (often empty).

**Prereqs:** OAI **2026.w16** (`38dc378`) built with RFsim + `telnetsrv`; CN5G up; Path C confs in `~/oai-config/path-c/`.

---

## 0) Install reproduce scripts from this kit

```bash
mkdir -p ~/oai-config/path-c ~/leo-evidence
cp -a ~/OAI_ISSUE_RFSIM_DUAL_DU_HO/reproduce/*.sh ~/oai-config/path-c/
# logging patch stays in the kit (or copy next to oai-config/patches/scratch-only/):
mkdir -p ~/oai-config/patches/scratch-only
cp -a ~/OAI_ISSUE_RFSIM_DUAL_DU_HO/patches/*.patch ~/oai-config/patches/scratch-only/
chmod +x ~/oai-config/path-c/*.sh
```

See also [`reproduce/README.md`](reproduce/README.md).

---

## 1) Clean + loopback aliases

```bash
sudo pkill -9 nr-softmodem nr-uesoftmodem 2>/dev/null; sleep 3
sudo ip addr add 127.0.0.3/8 dev lo 2>/dev/null
sudo ip addr add 127.0.0.4/8 dev lo 2>/dev/null
sudo ip addr add 127.0.0.5/8 dev lo 2>/dev/null
```

---

## 2) Green Path C baseline (DU0 = RFsim server + SAT_LEO)

```bash
bash ~/oai-config/path-c/pathc-bringup-du0.sh
# expect: OK / PING SUCCESS — oaitun_ue1 up, ping to DN OK
```

---

## 3a) One-shot closeout (preferred for filing evidence)

Runs D3a → D3b (DU1 client + F1 Setup) → D3c (`ci trigger_f1_ho` + re-ping):

```bash
bash ~/oai-config/path-c/d3c-closeout.sh
# → ~/leo-evidence/d3c_closeout_<ts>/
```

**Expected verdict:** class **`F1U_PATH_SWITCH`**, retry **`FAIL`**  
(F1-U switches to DU1; radio never syncs PCI 1; post-HO ping fails.)

---

## 3b) Manual HO (if not using closeout)

```bash
cd ~/openairinterface5g/cmake_targets/ran_build/build
sudo -E env LD_LIBRARY_PATH="$PWD" ./nr-softmodem --rfsim \
  -O ~/oai-config/path-c/gnb-du1.band254.ntn.pci1.conf \
  --rfsimulator.[0].serveraddr 127.0.0.1
# wait for F1 Setup Response in /tmp/pathc/du1.log (or your du1 log)

echo 'ci trigger_f1_ho' | nc -w 2 127.0.0.1 9099
# wait ≥60s: F1-U → DU1; no PCI:1; ping 100% loss
```

---

## 4) Optional — peer-mix counter proof (strongest evidence)

Requires develop scratch at `~/oai-develop-scratch` @ **`31ffb21a82`** (or current develop with the same RFsim symbols). Patch is **logging only**; never apply to the pinned LEO tree.

```bash
# Ensure kit patch is visible to the script (SHARE_ROOT or default paths):
export SHARE_ROOT="$HOME/OAI_ISSUE_RFSIM_DUAL_DU_HO"
# OR keep patch under ~/oai-config/patches/scratch-only/ and set:
# export PATCH_DIR="$HOME/oai-config/patches/scratch-only"

bash ~/oai-config/path-c/p1-instrumented-rerun.sh
# → ~/leo-evidence/d3c_p1_peer_mix_<ts>/ with RFSIM_PEER_MIX / _SUMMARY
# Expect: UE RX has DU1-origin samples = N before and after HO
```

If `p1-instrumented-rerun.sh` still looks for the lab share tree, set:

```bash
export SHARE_ROOT="$HOME"   # after installing patches under ~/oai-config/patches/scratch-only
export PATCH_DIR="$HOME/oai-config/patches/scratch-only"
export DOCS_EV="$HOME/leo-evidence"   # or skip writing back to repo docs
```

---

## 5) Contrast (optional) — AWGN HO that PASSES

Different recipe — **UE = RFsim server**, DUs = clients, **AWGN**:

```bash
bash ~/oai-config/path-c/pathc-du1-ho.sh
# expect: OK: PATH C F1 HO COMPLETE (channel=AWGN)
```

This does **not** contradict the D3c failure; it uses host-side mixing at the UE.

---

## 6) Package for GitLab

```bash
bash ~/OAI_ISSUE_RFSIM_DUAL_DU_HO/collect-evidence.sh
# → ~/OAI_ISSUE_RFSIM_DUAL_DU_HO/issue_evidence_<ts>.tar.gz
```

Then paste [`ISSUE.md`](ISSUE.md) on GitLab and attach the tarball. See [`VERIFY_AND_CAPTURE.md`](VERIFY_AND_CAPTURE.md).
