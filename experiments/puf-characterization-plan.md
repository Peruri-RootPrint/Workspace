# RO-PUF Characterization Plan

## Objective

Menentukan apakah RO-PUF pada Cyclone V DE10-Nano memberikan **device-specific variation yang berguna** dan **reliability yang cukup** untuk key reconstruction.

Primary project risk bukan FPGA capacity, tetapi apakah measurement didominasi oleh process variation atau deterministic placement/routing artifact.

## Board Population

- **Minimum:** 3 DE10-Nano boards.
- **Preferred:** 5 boards.
- Lebih banyak board meningkatkan nilai characterization, tetapi proposal tidak mengklaim population-level statistics dari 3-5 board.

## Fixed Experimental Controls

Catat untuk setiap run:

- board ID;
- Cyclone V part;
- Quartus edition/version;
- fitter seed;
- Git commit;
- PUF constraint/partition version;
- bitstream hash;
- actual temperature probe reading jika tersedia;
- power-cycle index;
- measurement timestamp.

## PUF Architecture Under Test

Initial candidate:

- local paired RO comparison;
- hanya 2 RO aktif pada satu waktu;
- fixed measurement window;
- frequency/count comparison;
- placement/routing region dipertahankan semirip mungkin antar-build;
- raw response hanya tersedia pada characterization build.

Final number of RO pairs tidak dibekukan sebelum resource/latency characterization.

## Phase 1 - RO Preservation and Counter Sanity

Goal: pastikan desain benar-benar menghasilkan oscillator yang hidup dan dapat diukur.

Checklist:

- synthesis tidak mengeliminasi oscillator chain;
- post-fit location/routing dapat direview;
- selected RO menghasilkan count;
- pair selection tidak mengaktifkan terlalu banyak RO;
- counter wrap/CDC dapat dikontrol;
- repeated read pada board sama tidak menunjukkan artifact control path.

**Gate G1:** seluruh target RO terukur.

## Phase 2 - Canonical Multi-Board Response

Gunakan pair index yang sama pada semua board.

Untuk setiap board:

1. cold/power cycle;
2. warm-up yang sama;
3. ukur seluruh pair;
4. ulangi `Q` kali;
5. simpan count delta dan derived bit.

Jangan membandingkan response dengan mask personal yang menghasilkan index berbeda.

## Phase 3 - Uniqueness Screening

Untuk `B` boards dan canonical response length `n`:

```text
Inter-HD(i,j) = d_H(R_i, R_j) / n
```

Laporkan:

- all pair values;
- mean dan distribution;
- minimum/maximum;
- number of identical pair responses;
- per-bit aliasing.

Initial engineering screening target: mean HD berada kira-kira 40-60%, tetapi keputusan akhir harus berdasarkan distribusi dan bit behavior, bukan mean saja.

**Gate G2:** variation lintas board cukup berguna dan tidak didominasi constant layout bits.

## Phase 4 - Reliability / BER

Untuk board `i` dengan enrolled reference `R_i` dan repeated sample `R_i,q`:

```text
BER_i = sum_q d_H(R_i, R_i,q) / (Q * n)
```

Uji:

- repeated measurement dalam satu power epoch;
- multiple power cycles;
- room condition yang dicatat;
- moderate temperature variation hanya jika setup aman tersedia.

Laporkan BER per bit/pair, bukan hanya aggregate average.

## Phase 5 - Bit Selection / Stability Mask

Enrollment dataset dipisahkan dari holdout test.

Candidate bit selection dapat menggunakan:

- absolute frequency-difference margin;
- repeated stability;
- exclusion of tie/near-tie pair;
- bias/aliasing consideration.

Jangan memilih mask menggunakan holdout result.

## Phase 6 - ECC / Fuzzy Extractor Parameter Selection

ECC parameter harus diturunkan dari measurement, bukan dipilih lebih dulu.

Inputs:

- raw BER distribution;
- tail error behavior;
- target reconstruction failure;
- available ALM/M10K;
- helper-data leakage;
- latency target.

**Gate G3:** BER/tail mendukung ECC yang feasible dalam resource budget.

## Phase 7 - Holdout Reconstruction

Setelah mask/ECC/format dibekukan:

- gunakan dataset power cycle baru;
- jalankan repeated reconstruction;
- catat successful / failed reconstruction;
- jangan cache prior key untuk menutupi failure;
- lakukan authentication-reference check untuk memastikan reconstructed key benar.

**Gate G4:** reconstruction stabil pada holdout yang ditentukan.

## Temperature Characterization

Jika alat tersedia:

- room temperature;
- moderate heating dengan probe aktual;
- controlled cooling dalam safe board limit.

Do not claim temperature range yang tidak diuji.

## Failure Modes and Mitigation

| Failure | Interpretation | Response |
|---|---|---|
| semua board menghasilkan response hampir sama | routing/layout dominates | redesign pairing/placement/constraints |
| BER terlalu tinggi | pair terlalu dekat / environment sensitive | stronger margin filter, longer measurement, ECC review |
| banyak constant bits | spatial/systematic bias | local pair redesign, bit filtering, placement review |
| recompile mengubah identity besar | PUF region tidak stable terhadap fit | lock partition / preserve routing strategy / re-enroll with documented version |
| self-heating/coupling | terlalu banyak oscillator aktif / window tidak tepat | limit active RO, add settle time, review spacing |

## Data Handling

Operational raw PUF data dianggap sensitif.

Public repository sebaiknya hanya memuat:

- aggregated metrics;
- retired experimental board dataset jika disetujui;
- plots tanpa active operational root material.

Jangan commit verifier secret atau active enrolled response ke repository publik.
