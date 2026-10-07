# RootPrint Success Metrics

Semua angka pada tabel ini adalah **design targets / acceptance criteria**, bukan hasil aktual, sampai measurement dilakukan.

| Metric | Why it matters | Initial Target / Acceptance | Measurement |
|---|---|---|---|
| Inter-device Hamming Distance | screening device-specific variation | mean screening range 40-60%; no identical candidate response pair | compare canonical positions across all board pairs |
| Raw BER / reliability | menentukan stabilitas dan ECC requirement | after masking target <= 1% as initial gate; final criterion follows measured tail | >= 1,000 response/board/condition if feasible |
| Bit aliasing | mendeteksi layout-dominated constant bits / bias | report per-position proportion and constant-bit rate | canonical positions across boards |
| Key reconstruction success | root material harus reproducible | target >= 99.9%; initial campaign with 0 unexplained failure | repeated reconstruction after power cycle/holdout |
| Entropy / leakage budget | memastikan extraction claim tidak melebihi evidence | security claim only if model and helper-data leakage support it | source model + helper-data leakage analysis |
| AES-CMAC correctness | protokol harus cryptographically correct | 100% known-answer/reference tests pass | independent software reference |
| Replay rejection | challenge freshness | 0 accepted replay in defined campaign | old/duplicate/expired response tests |
| Clone-copy campaign | main device-bound experiment | 0 acceptance on RootPrint campaign; stored-key baseline copy succeeds | directed board-pair tests with fresh nonce |
| Authentication latency | usability / architecture cost | p95 fabric latency <= 100 ms initial target | on-chip timestamps; separate transport RTT |
| Zeroization latency | lifecycle enforcement | target <= 1,024 active SYS cycles unless architecture changes | hardware counter + secret inventory audit |
| FPGA utilization | feasibility on DE10-Nano | < 30% ALM working target; memory within actual fit | Quartus Fitter report |
| System timing | deterministic implementation | SYS 50 MHz with non-negative slack | TimeQuest post-fit |
| Secret-path audit | HPS must not read internal secret | 0 production register/debug path exposing secret | RTL/netlist review + register fuzzing |
| Fault Tier-2 | suppress invalid output under tested fault model | 0 incorrect authentication tag released in defined campaign | debug fault injection build |

## Interpretation Notes

### Hamming distance is not proof of 128-bit security

Data dari 3-5 boards hanya merupakan **engineering screening**, bukan population-level entropy proof.

### Zero failure is not lifetime reliability

Jika tidak ada failure dalam beberapa ribu trial, hasil itu hanya mendukung confidence pada kondisi campaign yang diuji. Drift, aging, dan correlated measurements tetap membatasi interpretasi.

### Resource estimate vs measured result

Gunakan label:

- `Target`
- `Pre-synthesis estimate`
- `Measured / post-fit`

secara terpisah. Jangan mengubah estimate menjadi result tanpa Quartus report.

## Planned Result Table

| Build | ALM / FF / M10K / DSP | Fmax / SYS Slack | Status |
|---|---|---|---|
| Characterization | TBD | TBD | Pending implementation |
| Production RootPrint | TBD | TBD | Pending implementation |
| Stored-key baseline | TBD | TBD | Pending implementation |
| Production + Tier-2 | TBD | TBD | Optional / pending |
