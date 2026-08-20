# D3c peer-mix counters — P1 instrumented rerun

**UTC:** 20260719T010726Z
**Scratch commit:** `31ffb21a82`
**Patch SHA256:** `4d56f97aeb583d5279bf0ac750652257e16cd722d0b327ec476c97801c5eb1eb`
**Relay state before apply:** absent_clean
**P1 result:** **FAIL**
**Path C restore:** **PASS**
**VM evidence:** `/home/USER/leo-evidence/d3c_p1_peer_mix_20260719T010726Z`
**Script:** `oai-config/path-c/p1-instrumented-rerun.sh`
**Patch:** `oai-config/patches/scratch-only/0001-rfsim-peer-mix-counters.patch`

---

## Setup

| Item | Value |
|------|-------|
| Topology | Green — DU0 = RFsim **server**; UE + DU1 = **clients** |
| Channel | AWGN (mode=`empty`) |
| Binaries | Scratch @ `31ffb21a82` + peer-mix counter patch |
| Instrumentation | LOGGING ONLY — `flushInput` / `rfsimulator_read_internal` / `rfsimulator_write_internal` |
| Rank-2 relay | **Reverted / absent** for honest stock P1 mix measurement |

## Counter tags

- `RFSIM_PEER_MIX` — per client_id: `rx_from_client`, `summed_into_host_rx`, `host_tx_out_to_client`
- `RFSIM_PEER_MIX_SUMMARY` — `host_rx=sum_of_peer_cirbufs` / `client_out=host_tx_only`

## Extracted lines (tail)

```
[HW]     RFSIM_PEER_MIX role=SERVER client_id=1 sock=86 model=- rx_from_client=3190786560 summed_into_host_rx=3190832640 host_tx_out_to_client=3190863360 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX_SUMMARY role=SERVER nconn=2 host_rx=sum_of_peer_cirbufs client_out=host_tx_only peer_cirbuf_never_forwarded_to_sibling_clients
[HW]     RFSIM_PEER_MIX role=SERVER client_id=0 sock=85 model=- rx_from_client=4887820701 summed_into_host_rx=4887828480 host_tx_out_to_client=4887882240 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX role=SERVER client_id=1 sock=86 model=- rx_from_client=3278223360 summed_into_host_rx=3278269440 host_tx_out_to_client=3278307840 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX_SUMMARY role=SERVER nconn=2 host_rx=sum_of_peer_cirbufs client_out=host_tx_only peer_cirbuf_never_forwarded_to_sibling_clients
[HW]     RFSIM_PEER_MIX role=SERVER client_id=0 sock=85 model=- rx_from_client=4979565981 summed_into_host_rx=4979573760 host_tx_out_to_client=4979619840 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX role=SERVER client_id=1 sock=86 model=- rx_from_client=3369968640 summed_into_host_rx=3370014720 host_tx_out_to_client=3370037760 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX_SUMMARY role=SERVER nconn=2 host_rx=sum_of_peer_cirbufs client_out=host_tx_only peer_cirbuf_never_forwarded_to_sibling_clients
[HW]     RFSIM_PEER_MIX role=SERVER client_id=0 sock=85 model=- rx_from_client=5066511261 summed_into_host_rx=5066511360 host_tx_out_to_client=5066565120 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX role=SERVER client_id=1 sock=86 model=- rx_from_client=3456906240 summed_into_host_rx=3456952320 host_tx_out_to_client=3456990720 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX_SUMMARY role=SERVER nconn=2 host_rx=sum_of_peer_cirbufs client_out=host_tx_only peer_cirbuf_never_forwarded_to_sibling_clients
[HW]     RFSIM_PEER_MIX role=SERVER client_id=0 sock=85 model=- rx_from_client=5154485661 summed_into_host_rx=5154493440 host_tx_out_to_client=5154554880 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX role=SERVER client_id=1 sock=86 model=- rx_from_client=3544888320 summed_into_host_rx=3544934400 host_tx_out_to_client=3544980480 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX_SUMMARY role=SERVER nconn=2 host_rx=sum_of_peer_cirbufs client_out=host_tx_only peer_cirbuf_never_forwarded_to_sibling_clients
[HW]     RFSIM_PEER_MIX role=SERVER client_id=0 sock=85 model=- rx_from_client=5239848861 summed_into_host_rx=5239856640 host_tx_out_to_client=5239910400 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX role=SERVER client_id=1 sock=86 model=- rx_from_client=3630251520 summed_into_host_rx=3630297600 host_tx_out_to_client=3630336000 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX_SUMMARY role=SERVER nconn=2 host_rx=sum_of_peer_cirbufs client_out=host_tx_only peer_cirbuf_never_forwarded_to_sibling_clients
[HW]     RFSIM_PEER_MIX role=SERVER client_id=0 sock=85 model=- rx_from_client=5325711261 summed_into_host_rx=5325719040 host_tx_out_to_client=5325772800 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX role=SERVER client_id=1 sock=86 model=- rx_from_client=3716113920 summed_into_host_rx=3716160000 host_tx_out_to_client=3716198400 note=stock_client_out_is_host_tx_only
[HW]     RFSIM_PEER_MIX_SUMMARY role=SERVER nconn=2 host_rx=sum_of_peer_cirbufs client_out=host_tx_only peer_cirbuf_never_forwarded_to_sibling_clients
```

Full extract: `/home/USER/leo-evidence/d3c_p1_peer_mix_20260719T010726Z/du0_peer_mix_lines.txt` (summary lines: 63; stock_note_seen=1; max_nconn≈2).

## Verdict — UE RX contains DU1-origin samples?

| Phase | DU1 samples in UE RX? | Basis |
|-------|----------------------|-------|
| **Before HO** | **N** | Stock `rfsimulator_write_internal` sends **host (DU0) TX only** to UE; peer CirBufs (incl. DU1 once connected) sum into **DU0 host RX** via `rfsimulator_read_internal`, not into UE client outbound |
| **After HO** | **N** | Same stock star topology; sibling CirBuf never forwarded (no Rank-2 relay). UE still receives only DU0 host TX on its socket |

### Interpretation

Under **DU0=server** multi-client RFsim, the stock multi-peer sum path mixes peers into the **server host RX** only.
The UE client RX stream is **host-TX-only** — it never contains DU1-origin samples before or after F1 HO.
This matches P1 FAIL (no `PCI: 1`) and motivates Rank-2 sibling mix / Rank 5–6 topology work.

## Honesty

- Counters are **logging only**; no IQ path behavior change.
- Measured against **stock** peer-sum (Rank-2 sibling relay reverted before this run).
- `summed_into_host_rx` on client_id for DU1 means samples entered **DU0 UL RX**, not UE DL.

