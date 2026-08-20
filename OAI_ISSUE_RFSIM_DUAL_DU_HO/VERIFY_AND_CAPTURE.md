# Verify, capture, and share

End-to-end process: (A) reproduce, (B) capture, (C) package, (D) file on **GitLab**.  
All VM steps use **`~/` paths** (not empty hgfs).

---

## A. Reproduce

Preferred one-shot:

```bash
bash ~/oai-config/path-c/d3c-closeout.sh
# Expected: F1U_PATH_SWITCH + retry FAIL
```

Strongest measured proof (optional):

```bash
bash ~/oai-config/path-c/p1-instrumented-rerun.sh
# → ~/leo-evidence/d3c_p1_peer_mix_*/ with RFSIM_PEER_MIX lines
```

Full step list: [`REPRODUCE.md`](REPRODUCE.md).

---

## B. What to capture

| Evidence | Where | Why |
|---|---|---|
| CU log | `/tmp/pathc/cu.log` | `Handover triggered`, `Update tunnel …:2153`, `marking HO as complete` |
| DU0 / DU1 / UE logs | `/tmp/pathc/` + closeout copies | F1 Setup, CFRA, no PCI 1 / no in-sync |
| Closeout verdict | `~/leo-evidence/d3c_closeout_*/` | classified result + excerpts |
| Peer-mix counters | `~/leo-evidence/d3c_p1_peer_mix_*/` | measured root cause |
| Static matrices (in kit) | `evidence/` | D3 green + PRACH bisect |
| Logging patch | `patches/0001-rfsim-peer-mix-counters.patch` | re-measure |

Manual pcap (if needed — start **before** HO):

```bash
sudo tcpdump -i lo  -w ~/d3c_f1.pcap 'sctp or udp port 2153' &
sudo tcpdump -i any -w ~/d3c_core.pcap 'sctp port 38412' &
echo 'ci trigger_f1_ho' | nc -w 2 127.0.0.1 9099
# stop after ~60s
```

---

## C. Package

From this kit directory on the VM:

```bash
bash ~/OAI_ISSUE_RFSIM_DUAL_DU_HO/collect-evidence.sh
# → issue_evidence_<ts>.tar.gz in THIS folder
```

The packager:

- pulls newest `~/leo-evidence/{d3c_closeout,d3c_p1_peer_mix,diag_d3_green_dual_du,d3_develop,d3c_rca}_*` dirs  
- copies `/tmp/pathc/*.log` if present  
- includes local `evidence/`, `patches/`, `ISSUE.md`  
- scrubs `/home/$USER` → `/home/USER` and hgfs share paths  

**Skim the tarball contents once** before attaching.

---

## D. File on GitLab (you do this — kit does not auto-file)

1. Open [gitlab.eurecom.fr/oai/openairinterface5g/-/issues](https://gitlab.eurecom.fr/oai/openairinterface5g/-/issues) (not the GitHub mirror).
2. New issue → paste body from **`ISSUE.md`**.
3. Attach `issue_evidence_<ts>.tar.gz`.
4. Mention the logging-only patch / offer an MR if useful.

**Do not claim the issue is already filed.** Status of this kit: **ready if you choose**.

---

## Honesty checklist before you post

- [ ] Reproduced on **2026.w16** and (if claiming) develop `@ 31ffb21a82`.
- [ ] F1-U path-switch stated as **working**.
- [ ] Root cause stated as **measured** (peer-mix counters), not guessed.
- [ ] Software-only RFsim; no PHY air-interface root-cause claim.
- [ ] AWGN UE-as-server HO contrast stated as **different recipe**.
- [ ] Personal paths/usernames scrubbed from attachments.
- [ ] SGP4 → RFsim inject remains **OFF** (unrelated; do not imply live orbit inject in this bug).
