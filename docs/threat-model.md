# RootPrint Threat Model

Dokumen ini mendefinisikan **batas keamanan prototype RootPrint pada DE10-Nano**. Threat model sengaja dibuat sempit dan dapat diuji; prototype FPGA ini bukan secure element bersertifikasi.

## 1. Security Objective

RootPrint menargetkan properti berikut:

> Menyalin aset digital yang dapat diakses attacker ke FPGA fisik lain tidak cukup untuk memindahkan identitas kriptografis device yang telah di-enroll.

Root secret diturunkan dari karakteristik fisik device dan hanya hidup secara volatil ketika diperlukan.

## 2. Trust Boundary

### Trusted

- FPGA fabric yang menjalankan **approved production bitstream**.
- Proses enrollment yang dikendalikan tim/issuer.
- Public anchor / capsule metadata yang dipin oleh konfigurasi tepercaya.
- Verifier pada laptop terpisah.
- Security FSM dan internal key path di fabric.

### Untrusted

- HPS sebagai host/transport.
- SD card / external media.
- Pesan masuk dari host.
- Helper data yang berada pada media non-secret.
- Software host yang tidak memerlukan akses terhadap root/key.

## 3. Assets to Protect

- Raw PUF response yang dipakai untuk reconstruction.
- Reconstructed root material.
- Derived device/authentication key.
- AES key schedule dan cryptographic internal state.
- Enrollment-specific sensitive characterization data.

## 4. In-Scope Adversary Capabilities

Prototype harus dievaluasi terhadap attacker yang dapat:

1. menyalin firmware, SD card, configuration data, helper data, dan host software;
2. menjalankan salinan digital tersebut pada DE10-Nano fisik lain;
3. merekam dan mengirim ulang authentication tag lama;
4. memodifikasi nonce, ID, context, command, atau public helper capsule;
5. melakukan repeated authentication attempts;
6. memicu logical/debug fault pada verification build;
7. mengirim bus transaction malformed, partial write, busy-write, timeout, atau urutan command ilegal.

## 5. Out-of-Scope Capabilities

Prototype ini **tidak** mengklaim perlindungan terhadap:

- attacker yang memiliki board asli dan memprogram ulang FPGA dengan malicious bitstream yang sengaja membaca PUF;
- invasive silicon probing atau decapsulation;
- laser fault injection;
- EM probing;
- side-channel resistance terhadap power/EM analysis;
- physical voltage glitch resistance;
- certified active tamper mesh;
- relay attack / distance bounding;
- ASIC-level physical security;
- secure boot / anti-rollback lengkap untuk configuration image.

## 6. Primary Security Properties

### 6.1 No production secret read path

Build produksi tidak boleh memetakan register/interface untuk:

- raw PUF response;
- root secret;
- device key;
- AES internal state;
- CMAC subkey;
- fuzzy-extractor secret workspace.

### 6.2 Fresh challenge authentication

Verifier menghasilkan nonce baru untuk setiap sesi. Authentication tag dihitung terhadap nonce + device ID + context. Response lama tidak valid untuk challenge baru.

### 6.3 Fail-closed state transition

Fault, timeout, illegal state, atau batas kegagalan yang ditetapkan harus menghasilkan:

```text
security event
    -> output suppression
    -> ZEROIZE
    -> LOCKED (untuk fatal event)
```

### 6.4 Zeroization

Material sensitif yang sedang hidup harus dihapus ketika transaksi selesai atau security event terjadi. Zeroization harus diaudit berdasarkan inventory lokasi secret, bukan hanya status bit `ZEROIZED`.

### 6.5 Device-bound copy experiment

Eksperimen utama menyalin aset digital Board A ke Board B. RootPrint dianggap memenuhi target prototype jika Board B tidak dapat autentikasi sebagai A pada campaign yang ditentukan, dengan kontrol bahwa Board B tetap berfungsi dan dapat autentikasi sebagai identitasnya sendiri setelah enrollment yang benar.

## 7. Baseline Scope

Stored-key baseline menyimpan root/authentication key pada storage yang memang dapat ikut tersalin bersama image/media.

Baseline ini **bukan** representasi:

- eFuse;
- OTP;
- TPM;
- secure element komersial;
- secure NVM dengan proteksi fisik.

Perbandingan dengan teknologi tersebut harus diperlakukan sebagai **architectural trade-off**, bukan hasil clone-copy experiment.

## 8. PUF-Specific Risks

RO-PUF pada FPGA dapat gagal memberikan uniqueness yang berguna jika delay didominasi oleh deterministic placement/routing. Risiko lain:

- fitter mengubah placement/routing;
- synthesis mengoptimasi oscillator chain;
- spatial gradient menyebabkan bit bias;
- adjacent oscillator coupling;
- temperatur dan self-heating mengubah frequency ordering;
- helper data membocorkan sebagian informasi tentang source response.

Karena itu klaim keamanan bergantung pada hasil characterization aktual.

## 9. Claims We Do Not Make

RootPrint tidak mengklaim:

- "unclonable";
- "tamper-proof";
- "secret cannot be extracted";
- "side-channel resistant";
- "first in Indonesia/world";
- "more secure than commercial secure element";
- "replacement for e-KTP/e-passport chip";
- "quantum-safe".

## 10. Security Acceptance Principle

Setiap claim harus memiliki hubungan langsung:

```text
claim -> experiment -> metric -> pass criterion
```

Jika measurement tidak mendukung claim, scope atau wording claim harus diturunkan.
