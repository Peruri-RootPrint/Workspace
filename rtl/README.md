# RTL

Folder ini disiapkan untuk implementasi RTL RootPrint.

**Status saat ini: belum mengklaim RTL selesai.** Nama modul di bawah adalah contract awal yang berasal dari proposal dan dapat berubah setelah characterization dan integration review.

## Planned Modules

| Module | Purpose |
|---|---|
| `ro_puf_array` | Array ring oscillator sebagai sumber variasi fisik device |
| `puf_pair_selector` | Memilih pasangan RO lokal; membatasi oscillator aktif |
| `ro_counter` | Mengukur frekuensi/relative count dan menangani CDC |
| `stability_filter` | Majority/reliability filtering dan tie rejection |
| `helper_data_controller` | Parsing/validasi helper capsule publik |
| `fuzzy_extractor` | Error correction / key reconstruction |
| `entropy_extractor` | Privacy amplification / extraction sesuai parameter final |
| `kdf_core` | Menurunkan `K_auth` dengan domain separation |
| `aes_core` | AES-128 iterative core |
| `cmac_controller` | AES-CMAC message handling dan tag generation |
| `key_register` | Penyimpanan key volatil tanpa read path ke host |
| `security_fsm` | Fail-closed lifecycle, timeout, failure policy, lockout |
| `zeroize_controller` | Menghapus seluruh secret workspace |
| `nonce_interface` | Snapshot nonce / ID / context secara atomik |
| `host_bridge` | Avalon-MM / lightweight host interface untuk data non-secret |
| `clock_monitor` | Tier-2 optional anomaly monitor |

## Interface Principles

1. Raw PUF response tidak diekspos di build produksi.
2. Reconstructed root dan `K_auth` tidak memiliki register baca host.
3. Security event memiliki prioritas terhadap publication tag.
4. `zeroize` harus masuk ke semua module yang menyimpan secret state.
5. Build characterization dan build production harus dipisahkan secara eksplisit.
6. Placement/constraint region PUF harus direview pasca-fit.

## Suggested Source Layout (when implementation begins)

```text
rtl/
├── puf/
│   ├── ro_puf_array.sv
│   ├── puf_pair_selector.sv
│   └── ro_counter.sv
├── key_reconstruction/
│   ├── stability_filter.sv
│   ├── helper_data_controller.sv
│   ├── fuzzy_extractor.sv
│   └── entropy_extractor.sv
├── crypto/
│   ├── aes_core.sv
│   ├── cmac_controller.sv
│   └── kdf_core.sv
├── security/
│   ├── key_register.sv
│   ├── zeroize_controller.sv
│   └── security_fsm.sv
└── interface/
    ├── nonce_interface.sv
    └── host_bridge.sv
```

Jangan commit secret verifier, operational raw PUF response, atau enrollment secret ke repository publik.
