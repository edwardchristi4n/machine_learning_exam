# Checklist Spesifikasi 7.2 & 7.3 – KelpAdam

Dokumen ini memetakan setiap poin Spesifikasi Proyek UTS Bagian **7.2 (Data Cleaning dan Validasi)** dan **7.3 (Data Preparation)** ke bagian notebook `notebook/KelpAdam_UTS_DataPreparation_EDA.ipynb` yang menjawabnya, beserta hasilnya pada data kelompok.

**Ringkasan:** semua poin 7.2 dan 7.3 sudah terpenuhi.

| | |
|---|---|
| Data awal | 194 baris × 116 kolom (`dataset/raw/Survei_Gaya_Belajar_IPS_IPK_NO_PII.xlsx`, MD5 `390ab027d2a950f63a72684d795caa72`) |
| Data hasil | 192 baris × 160 kolom (`dataset/KelpAdam_survey_prepared.csv`) |
| Cleaning log | `dictionary/KelpAdam_cleaning_log.csv` (10 entri: C1–C9; C5 punya 2 entri) |
| Data dictionary | `dictionary/KelpAdam_data_dictionary.xlsx` (160 kolom terdokumentasi) |

**Kode bagian di notebook:** 1.x = data understanding, C1–C10 = cleaning, S1–S4 = scoring, P1–P3 = preparation.

---

## 7.2 Data Cleaning dan Validasi

### Poin 1 – Periksa missing value, respons kosong, nilai tidak valid, kategori tidak konsisten, dan duplikasi

| Yang diperiksa | Bagian | Hasil | Status |
|---|---|---|---|
| Missing value | 1.5a–d (deteksi); C2, C5, C6 (penanganan) | Kolom survei: 208 sel kosong, semuanya berasal dari 2 baris kosong total. IPS kosong karena angkatan (*missing struktural*), jadi **tidak diimputasi**. Tidak ada kolom yang kosong seluruhnya di data asli. | ✅ |
| Respons kosong | 1.5a, C2 | ID 11 & 133 kosong total (survei, IPS, IPK) → dihapus, 194 → 192 | ✅ |
| Nilai tidak valid | C3, C4, C7; C5, C6 | 0 jawaban Likert di luar 1–5 (C3), 0 opsi VARK asing (C4), 0 IPS/IPK di luar 0–4 (C7). Nilai 0 yang berarti "belum ada nilai": 173 sel IPS 2026 S1 (C5) dan 4 IPK (C6, ID 20, 29, 59, 82) → NaN | ✅ |
| Kategori tidak konsisten | 1.7, C4 | Setiap opsi VARK dicek terhadap daftar resmi (`VARK_MAP`): 0 typo, 0 beda kapitalisasi, 0 isian bebas | ✅ |
| Duplikasi | 1.6 (deteksi), C8 (cek ulang) | 0 ID ganda, 0 duplikat persis, 0 jawaban survei identik. 78 pasangan "sangat mirip" (≤ 5 dari 88 jawaban berbeda), semuanya melibatkan responden berpola seragam dan IPS/IPK-nya berbeda, jadi orangnya berbeda | ✅ |

### Poin 2 – Konversi tipe data & normalisasi teks/kategori secara hati-hati

| Yang dilakukan | Bagian | Hasil | Status |
|---|---|---|---|
| Konversi tipe | C3 | 88 item Likert float64 → Int64. Dibuktikan dengan tabel sebelum/sesudah: nilai jawaban tidak berubah | ✅ |
| Normalisasi teks | C4 | Jawaban VARK dirapikan dari `"opsi a;opsi b;"` menjadi `"opsi a; opsi b"` (`;` di akhir dan spasi berlebih dibuang). Isi opsi tidak berubah. Variasi kapitalisasi dicek: 0 | ✅ |

### Poin 3 – Validasi range & opsi jawaban sesuai instrumen; nilai di luar range ditandai & ditangani

| Yang divalidasi | Bagian | Hasil | Status |
|---|---|---|---|
| Range Likert | C3 | Semua ∈ {1, 2, 3, 4, 5} | ✅ |
| Opsi VARK | C4 | Semua opsi ada di daftar resmi; setiap soal punya tepat 1 opsi V, A, R, K | ✅ |
| Range IPS/IPK | C7 | Semua ∈ [0, 4]. Konsistensi IPK vs rata-rata IPS: selisih rata-rata 0,009, maksimum 0,21 | ✅ |
| Penanganan nilai tidak mungkin | C5, C6 | Tidak ada nilai di luar range. Nilai yang "tidak mungkin" secara makna (IPS 0 pada semester yang masih berjalan; IPK 0 tanpa IPS) → NaN dengan alasan tertulis di cleaning log | ✅ |

### Poin 4 – Hapus baris/kolom: catat jumlah sebelum-sesudah & alasan di cleaning log

| Penghapusan | Bagian | Dicatat | Status |
|---|---|---|---|
| 2 baris (ID 11, 133) | C2 | 194 → 192, alasan "respons kosong total" | ✅ |
| 1 kolom (`IPS_2026_S1`) | C5 | Setelah 0 → NaN kolom kosong seluruhnya → dibuang, alasan "tidak informatif; tetap ada di data asli". Kolom 116 → 115 | ✅ |
| Seluruh log | C10 | Ditampilkan di notebook dan disimpan (isinya sama persis) ke `dictionary/KelpAdam_cleaning_log.csv`, plus tabel ringkasan sebelum vs sesudah | ✅ |

### Poin 5 – Outlier tidak dihapus otomatis; bedakan nilai ekstrem valid vs nilai tidak mungkin

| Kasus | Bagian | Keputusan | Status |
|---|---|---|---|
| IPS/IPK sangat rendah (ID 4, 7, 30, 112, 128, 134, 175; IPK minimum 0,87) | C7 | **Dipertahankan**: nilai valid yang ekstrem | ✅ |
| 15 straightliner + 8 hampir straightline | C9 | **Tidak dihapus**, diberi flag (`flag_straightline`, `flag_near_straight`, `row_sd`, `longstring_max`) | ✅ |
| IPS 0 di semester berjalan, IPK 0 tanpa IPS | C5, C6 | Nilai yang "tidak mungkin" → NaN, bukan dihapus | ✅ |

---

## 7.3 Data Preparation

### Poin 1 – Penamaan kolom konsisten tanpa kehilangan keterlacakan

| Bagian | Hasil | Status |
|---|---|---|
| C1, P3 | 116 header panjang diganti kode pendek dengan pola konsisten (`VARK_Q01`, `HM_ACT_01`, `FS_SEN_03`, `KOLB_AE_06`, `IPS_2025_S2`). Teks asli disimpan di `kolom_map` dan kolom `arti` di data dictionary | ✅ |

### Poin 2 – Pisahkan dataset sumber & hasil preparation; transformasi lewat kode

| Bagian | Hasil | Status |
|---|---|---|
| Setup, sel awal Bagian 2 (`df = raw.copy()`), P2 | Sumber: `dataset/raw/…xlsx`, tidak pernah diubah (`raw` di notebook tidak disentuh). Hasil: `dataset/KelpAdam_survey_prepared.csv`. Semua langkah ada di kode dan notebook berjalan berurutan dari awal | ✅ |

### Poin 3 – Encoding/transformasi hanya jika perlu; belum preprocessing ML

| Bagian | Hasil | Status |
|---|---|---|
| Bagian 3–4 | Tidak ada one-hot encoding, scaling, atau imputasi. Transformasi yang dibuat hanya scoring (diarahkan dosen) dan perkiraan angkatan (variabel konteks untuk EDA) | ✅ |

### Poin 4 – Derived variables/scoring eksplisit & sumber aturan didokumentasikan

| Bagian | Hasil | Status |
|---|---|---|
| S1–S4, sel "Referensi aturan scoring", P1, P3 | Setiap scoring punya markdown yang menjelaskan aturannya dan kode yang eksplisit. Sumber teori dicantumkan di sel referensi (Fleming; Felder & Soloman; Kolb & Kolb 2005; Honey & Mumford) dan di kolom `sumber_aturan` data dictionary | ✅ |

### Poin 5 – Data dictionary: nama kolom, tipe data, arti, range/kategori valid, raw/derived

| Bagian | Hasil | Status |
|---|---|---|
| P3 | 160 kolom terdokumentasi dengan: `nama_kolom`, `jenis`, `tipe_data`, `arti`, `nilai_valid`, `nilai_teramati`, `terisi`, `raw_atau_derived`, `sumber_aturan` | ✅ |

### Tabel 5 jenis kolom

| Jenis | Ketentuan spesifikasi | Kolom di dataset prepared | Status |
|---|---|---|---|
| 1 Metadata non-identifying | Aman dianalisis | IPS per semester + IPK (10 kolom). Angkatan 2022 (1 orang) digabung ke "<=2023" agar tidak bisa dikenali | ✅ |
| 2 Raw/coded survey items | Dipertahankan untuk audit; makna jawaban asli tidak diubah | 16 VARK + 88 Likert (104 kolom); hanya format/tipe yang dirapikan (C3, C4) | ✅ |
| 3 Scores/dimensions | Dihitung dengan aturan yang disetujui dosen | 23 kolom skor. Arahan dosen (6 Nov 2026): scoring harus sesuai teori keempat model | ✅ |
| 4 Labels/profiles | Dapat ditelusuri ke aturan scoring; tidak dengan cut-off arbitrer | 16 kolom label. Batas dari teori: VARK = stepping distance Fleming; Felder = ambang ILS 1–3/5–7/9–11 diskalakan proporsional; Kolb = median sampel (cara LSI 3.1); Honey-Mumford = pita persentil 10/20/40/20/10% (cara LSQ) | ✅ |
| 5 Prepared variables | Transformasi dijelaskan di notebook & dictionary | 6 kolom: `angkatan_inferred`, `n_semester_ips`, `row_sd`, `longstring_max`, `flag_straightline`, `flag_near_straight` | ✅ |

### Mengapa dataset prepared berisi 160 kolom

| Jenis | Kolom | Alasan |
|---|---|---|
| Identifier | 1 | Kunci baris |
| Metadata | 10 | Konteks akademik (IPS_2026_S1 dibuang karena kosong seluruhnya) |
| Raw survey item | 104 | **Wajib** dipertahankan untuk audit (tabel 7.3 no. 2) |
| Score | 23 | Bahan EDA dan modelling |
| Label/profile | 16 | Calon target modelling |
| Prepared variable | 6 | Konteks dan bukti kualitas data |

Pemilihan fitur untuk melatih model dilakukan di tahap modelling, bukan di UTS (Spesifikasi 7.3 poin 3).

---

## Catatan tata kelola data (Spesifikasi Bagian 6)

- `.gitignore` sudah ditambah `dataset/raw/` agar data mentah tidak ikut ter-commit.
- File data mentah **sudah pernah ter-commit** sebelum `.gitignore` diperbarui. `.gitignore` tidak menghapus file yang sudah ter-commit, jadi jalankan di terminal folder UTS:
  ```
  git rm --cached "dataset/raw/Survei_Gaya_Belajar_IPS_IPK_NO_PII.xlsx"
  git commit -m "Keluarkan data mentah dari repository"
  git push
  ```
  File di laptop tetap ada; yang dihapus hanya salinannya di repository. Karena file ini masih tersimpan di riwayat commit lama, pastikan repository GitHub berstatus **Private**.
- `requirements.txt` berisi versi paket dari `.venv` kelompok, untuk mendukung reproducibility.
