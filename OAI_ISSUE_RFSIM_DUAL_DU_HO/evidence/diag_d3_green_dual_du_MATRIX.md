# D3 green dual-DU SAT_LEO diag 20260725T025734Z

Host: USER-virtual-machine
Recipe: pathc-bringup-du0.sh (DU0=RFsim server + SAT_LEO) then DU1 client + F1 HO
Honesty: tests whether rfsim server mixes second DU client under live SAT_LEO with proven green recipe — PASS = live SAT_LEO dual-DU HO with NO code changes; fail = evidence for upstream rfsim multi-client report.
Does not modify make leo / pathc-bringup-du0.sh / oai-config/patches/ / stock|_ref confs.

| ID | Label | Result | Notes |
|----|-------|--------|-------|
| D3a | green baseline (pathc-bringup-du0) | PASS | tunnel+ping |
| D3b | DU1 RFsim client F1 Setup | PASS | F1 Setup Response in du1.log |
| D3c | ci trigger_f1_ho + re-ping | FAIL | trig=1 mark=1 true_ho=0 ping=0 tun=1 gtp_du1=1 |
