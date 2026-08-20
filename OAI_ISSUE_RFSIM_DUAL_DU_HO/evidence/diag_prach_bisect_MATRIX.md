# PRACH bisect 20260718T033851Z (round 1 B1–B4 + round 2 B0/B5–B7)

Host: USER-virtual-machine
OAI build: /home/USER/openairinterface5g/cmake_targets/ran_build/build
RFSIMU defaults: empty/stock (HOST= ZERO= ZDOP=)
Honesty: round 1 isolates moving channel vs multi-client mixing; round 2 isolates NTN-SIB-vs-UE-flags vs static-delay. Does not prove PHY fix.

### Round 1 — channel / topology (B1–B4)

| ID | Label | SSB sync | prach_I0=0 cnt | Tunnel | Result | Notes |
|----|-------|----------|----------------|--------|--------|-------|
| B1 | static prop_delay, single DU0 | 1 | 954 | 0 | PARTIAL_SSB | SSB yes tunnel no prach_I0_0=954 |
| B2 | static prop_delay, dual DU | 1 | 1044 | 0 | PARTIAL_SSB | SSB yes tunnel no prach_I0_0=1044 |
| B3 | SAT_LEO+chanmod, single DU0 | 1 | 324 | 0 | PARTIAL_SSB | SSB yes tunnel no prach_I0_0=324 |
| B4 | SAT_LEO+chanmod, dual DU | 1 | 342 | 0 | PARTIAL_SSB | SSB yes tunnel no prach_I0_0=342 |

### Round 2 — NTN SIB / UE flags / static delay (B0, B5–B7)

Rationale: round 1 all PARTIAL_SSB → ruled out moving channel, dual-DU, topology.
Suspects: NTN SIB without UE flags; min_rxtxtime=4; prop_delay=20 itself.
Expected: B0 PASS=valid control; B0 FAIL=invalid run; B5 FAIL=static-delay UL bug; B7 PASS=missing UE NTN flags.

| ID | Label | SSB sync | prach_I0=0 cnt | Tunnel | Result | Notes |
|----|-------|----------|----------------|--------|--------|-------|
| B0 | AWGN no delay, SIB stripped, no UE NTN (control) | 1 | 00 | 0 | PARTIAL_SSB | SSB yes tunnel no prach_I0_0=00 |
| B5 | AWGN+pd20, SIB stripped, no UE NTN | 1 | 00 | 0 | PARTIAL_SSB | SSB yes tunnel no prach_I0_0=00 |
| B6 | SAT_LEO+chanmod+pd20, SIB stripped, no UE NTN | 1 | 324 | 0 | PARTIAL_SSB | SSB yes tunnel no prach_I0_0=324 |
| B7 | SAT_LEO+chanmod+pd20, SIB kept, UE NTN flags | 0 | 318 | 0 | FAIL_SSB | SSB no prach_I0_0=318 |
