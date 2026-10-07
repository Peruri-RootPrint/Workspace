# RootPrint

**PUF-Derived Hardware Root-of-Trust IP Core with Device-Bound Authentication on DE10-Nano**

RootPrint adalah proposal **Track: Secure Identity & Security Element Chip** untuk **PERURI Chip Hackathon 2026** oleh tim **Lodon Chip - Universitas Indonesia**.

RootPrint mengeksplorasi arsitektur hardware root-of-trust yang **tidak menyimpan device root key secara permanen pada media non-volatile perangkat**. Root material direkonstruksi saat dibutuhkan dari variasi fisik silikon melalui **Ring Oscillator Physical Unclonable Function (RO-PUF)**, distabilkan melalui key reconstruction/fuzzy extraction, diturunkan melalui KDF, digunakan untuk autentikasi **AES-CMAC**, lalu dihapus melalui mekanisme **zeroization**.

> **Core idea:** data digital dapat disalin, tetapi identitas kriptografis perangkat tidak seharusnya otomatis ikut berpindah ke FPGA fisik lain.

## Status

**Current stage:** proposal / pre-implementation.

Repository ini saat ini berfungsi sebagai **technical evidence hub** untuk arsitektur, threat model, verification plan, success metrics, dan rencana eksperimen. Folder `rtl/` belum mengklaim implementasi selesai. Artefak RTL, testbench, hasil synthesis, dan data board akan ditambahkan setelah implementasi dimulai.

## Arsitektur Singkat

```text
Verifier
   |
HPS / Host Bridge        (untrusted transport)
   |
-------------------- FPGA trusted boundary --------------------
   |
 RO-PUF
   |
 Reliability / Key Reconstruction
   |
 KDF
   |
 Volatile Device Key
   |
 AES-CMAC
   |
 Authentication Tag

 Security FSM -> zeroize / lockout / fail-closed response
```

Dokumen visual:

- [`docs/architecture/system-architecture.pdf`](docs/architecture/system-architecture.pdf)
- [`docs/architecture/authentication-flow.pdf`](docs/architecture/authentication-flow.pdf)
- [`docs/architecture/security-fsm.pdf`](docs/architecture/security-fsm.pdf)

## Masalah yang Diselesaikan

Secure embedded device membutuhkan **per-device root secret**. Pada desain yang menyimpan key sebagai data yang dapat disalin, duplikasi image/storage berpotensi ikut memindahkan material identitas. RootPrint memindahkan sumber root material ke karakteristik fisik perangkat dan menjaga material rahasia tetap berada di FPGA fabric selama runtime.

RootPrint **tidak** mengklaim PUF sebagai primitive baru dan **tidak** mengklaim menggantikan secure element komersial. Kontribusi yang ditargetkan adalah integrasi end-to-end yang terukur pada DE10-Nano:

1. RO-PUF untuk device-specific physical variation.
2. Reliability filtering dan key reconstruction.
3. KDF untuk domain-separated authentication key.
4. AES-CMAC untuk challenge authentication.
5. Security FSM dengan zeroization dan lockout.
6. HPS/FPGA trust boundary tanpa secret read path.
7. Multi-board characterization dan stored-key baseline comparison.

## Target Platform

- **Board:** Terasic DE10-Nano
- **FPGA/SoC:** Intel/Altera Cyclone V SoC
- **Target system clock:** 50 MHz
- **Working resource target:** < 30% ALM untuk prototype utama
- **Tooling:** Intel Quartus Prime, Platform Designer/Qsys, TimeQuest, SignalTap, SystemVerilog, ModelSim/Verilator atau simulator setara, Python/C untuk host dan automation

## Threat Model Ringkas

### In scope

- Menyalin firmware, SD card, helper data, configuration, dan software ke FPGA lain.
- Replay response autentikasi lama.
- Manipulasi command/helper data.
- Repeated authentication attempts.
- Logical fault yang didefinisikan untuk verification build.

### Out of scope untuk prototype FPGA

- Malicious reprogramming board asli dengan bitstream yang membaca PUF.
- Invasive probing / decapsulation.
- Laser/EM attack.
- Klaim side-channel resistance.
- Physical voltage glitch resistance.
- Relay attack / distance bounding.
- Certified tamper resistance.

Detail: [`docs/threat-model.md`](docs/threat-model.md).

## Bukti yang Ditargetkan

- Inter-device Hamming distance.
- Intra-device reliability / raw BER.
- Bit aliasing.
- Key reconstruction success.
- AES-CMAC correctness.
- Replay rejection.
- Clone-copy campaign.
- Authentication latency.
- FPGA resource utilization / timing closure.
- Zeroization latency.
- Secret-path audit.

Lihat [`verification/metrics.md`](verification/metrics.md).

## Demo yang Ditargetkan

1. **Board A** di-enroll dan berhasil autentikasi.
2. Aset digital A disalin ke **Board B**.
3. Board B mencoba menyamar sebagai A -> **REJECT** pada RootPrint.
4. Stored-key baseline dengan key yang berada pada storage copyable diuji sebagai pembanding.
5. Replay response lama -> **REJECT**.
6. Alarm/fault logis -> **ZEROIZE -> LOCKED**.

> Baseline stored-key hanya membandingkan terhadap **copyable external/software-accessible key storage**. Ini bukan klaim superioritas terhadap eFuse, OTP, TPM, atau secure element komersial.

## Struktur Repository

```text
RootPrint/
├── README.md
├── docs/
│   ├── RootPrint_Proposal.pdf
│   ├── architecture/
│   │   ├── system-architecture.pdf
│   │   ├── authentication-flow.pdf
│   │   └── security-fsm.pdf
│   └── threat-model.md
├── rtl/
│   └── README.md
├── verification/
│   ├── test-plan.md
│   └── metrics.md
├── experiments/
│   └── puf-characterization-plan.md
├── references/
│   └── references.md
└── LICENSE
```

## Tim

| Nama | Peran |
|---|---|
| **Simforianus Jonathan Flonas Harefa** | Hardware Security Architect & Team Lead |
| **Dicky Bayu Sadewo** | FPGA & PUF Engineer |
| **Farrel Athalla Muljawan** | Verification & System Integration Engineer |

Institusi: **Universitas Indonesia**.

## Prinsip Klaim

Repository ini membedakan dengan tegas antara:

- **target desain**,
- **hasil yang direncanakan untuk diukur**, dan
- **hasil eksperimen aktual**.

Tidak ada angka synthesis, uniqueness, BER, entropy, atau security claim yang dianggap hasil aktual sampai pengukuran board tersedia.

## Dokumen Proposal

Versi proposal submission-ready saat ini tersedia di:

[`docs/RootPrint_Proposal.pdf`](docs/RootPrint_Proposal.pdf)

## License

Kode dan materi repository ini dirilis di bawah **Apache License 2.0**, kecuali jika suatu file secara eksplisit menyatakan lisensi lain.
