# RootPrint Verification Test Plan

## 1. Tujuan

Verification harus membuktikan fungsi digital, membedakan hasil simulasi dengan evidence fisik PUF, dan menjaga hubungan yang jelas antara claim, test, metric, dan pass criterion.

## 2. Verification Layers

### Layer A - Unit RTL

- reset dan initialization;
- valid/ready / command handshake;
- timeout;
- illegal state;
- counter wrap/overflow;
- CDC behavior;
- helper-data parsing;
- fuzzy-extractor error injection;
- AES known-answer vectors;
- CMAC known-answer vectors;
- KDF/reference-model comparison;
- zeroize precedence;
- lockout counter;
- host address decoding.

### Layer B - Integrated RTL

- PUF response model -> key reconstruction -> KDF -> CMAC;
- transaction atomicity;
- busy-write rejection;
- partial command rejection;
- replay model;
- alarm exactly at tag commit;
- reset at every sensitive state;
- secret-path assertions.

**Important:** synthetic PUF model hanya membuktikan digital control flow. Ia tidak membuktikan uniqueness, entropy, reliability, atau physical unclonability.

### Layer C - Quartus / Post-Fit

- RO chain tidak dieliminasi;
- placement PUF sesuai constraint;
- routing/partition review;
- SYS timing closure;
- CDC/timing exceptions terdokumentasi;
- production/debug build differentiation;
- resource usage per entity;
- no secret debug probe pada production image.

### Layer D - Board Test

Minimal 3, target 5 board DE10-Nano dengan region/layout PUF yang konsisten.

Test:

- RO alive/count repeatability;
- repeated PUF response;
- inter-board comparison;
- reconstruction campaign;
- end-to-end authentication;
- replay rejection;
- clone-copy campaign;
- zeroization / fail-closed response;
- optional Tier-2 logical fault injection.

## 3. Verification Matrix

| Requirement | RTL Test | Board Test | Metric | Pass Criterion |
|---|---|---|---|---|
| RO preserved | pulse/gate model | netlist + counter | alive RO | seluruh RO target terukur |
| Counter/CDC | freq/phase sweep, wrap | board counter test | count error / overflow | tidak ada overflow tak terdeteksi |
| PUF stability | injected flip/tie | repeated response | raw BER | sesuai ECC budget |
| PUF uniqueness | index/mask correctness | all board pairs | inter-HD, bit aliasing | useful variation; no identical response in campaign |
| Key reconstruction | error 0..t, burst, >t | repeated reconstruction | failure count | tidak ada unexplained failure dalam campaign yang dibekukan |
| Helper integrity | mutate/truncate fields | mutate capsule | pre-PUF rejection | semua mutation campaign ditolak |
| AES/CMAC/KDF | KAT / reference model | independent verifier | mismatch | 0 mismatch |
| Fresh nonce | session model | repeated sessions | nonce reuse | challenge dipakai sekali |
| Replay rejection | old/duplicate tag | record + replay | accepted replay | 0 accepted |
| Clone-copy | two-device model | directed board pairs | accepted clone | 0 accepted pada RootPrint campaign; baseline copy succeeds |
| Zeroization | secret inventory assertions | observation build | wipe cycles | seluruh secret location tercakup |
| Output suppression | alarm on commit cycle | fault/alarm input | invalid tag released | 0 tag valid setelah alarm |
| Lockout | fail 1/2/3 | repeated failed requests | lock state | attempt berikut ditolak sesuai policy |
| Host cannot read secret | address/alias scan | production register fuzzing | secret read path | 0 path |
| Timing closure | constraints audit | TimeQuest | slack / unconstrained path | 50 MHz target closure |
| Resource fit | per-entity compile | Quartus Fitter | ALM/FF/M10K/DSP | within declared budget or transparently revised |

## 4. Clone-Copy Campaign

Untuk setiap pasangan terarah board `i -> j`:

1. enroll Board `i` sebagai identity `i`;
2. copy seluruh aset digital yang memang masuk threat model ke Board `j`;
3. minta Board `j` authenticate sebagai `i`;
4. jalankan >= 100 fresh nonce/pair jika waktu memungkinkan;
5. lakukan control test bahwa Board `j` sehat dan dapat authenticate sebagai `j` setelah enrollment yang benar.

3 board -> 6 directed pairs. 5 board -> 20 directed pairs.

Campaign ini bukan estimasi false-accept probability seluruh populasi.

## 5. Replay Campaign

- record valid tag untuk `nonce_1`;
- kirim ulang terhadap `nonce_2`;
- kirim response dua kali pada session yang sama;
- kirim setelah timeout;
- restart verifier dan pastikan session state tidak ambigu.

Expected: 0 replay accepted pada campaign.

## 6. Zeroization Test

Audit inventory:

- PUF reconstruction workspace;
- corrected response / root material;
- extractor state;
- KDF intermediate;
- authentication key;
- AES key schedule;
- CMAC subkeys/state.

Test harus membuktikan bahwa security event memblok output terlebih dahulu, lalu melakukan wipe sesuai contract.

## 7. Reporting Discipline

Setiap result harus mencatat:

- Git commit;
- RTL/test seed;
- Quartus version;
- device/board ID;
- bitstream/config hash;
- constraint version;
- test condition;
- PASS / FAIL / SKIP.

`SKIP` tidak boleh dihitung sebagai `PASS`.
