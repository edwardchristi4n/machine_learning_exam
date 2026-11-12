# Preferensi Gaya Belajar: Data Preparation & EDA (UTS Pembelajaran Mesin)

Proyek UTS mata kuliah **Pembelajaran Mesin**, Gasal 2026/2027, Universitas Atma Jaya Yogyakarta. Pengerjaan oleh **Kelompok Adam (KelpAdam)**.
Dosen pengampu: Yohanes Sigit Purnomo, Ph.D. dan Theresia Devi Indriasari, Ph.D.

Proyek ini mengolah data survei preferensi gaya belajar dari empat framework (**VARK, Honey-Mumford, Felder-Silverman, Kolb**) beserta IPS/IPK mahasiswa. Tujuannya membuat dataset yang bersih, terdokumentasi, dan siap untuk tahap modelling (UAS). Isinya meliputi data understanding, cleaning, scoring sesuai teori, data preparation, dan exploratory data analysis (EDA).

> **Data mentah tidak ada di repository ini.** File survei dari dosen hanya untuk keperluan mata kuliah, sehingga dikecualikan lewat `.gitignore` (lihat [Data & privasi](#data--privasi)).

---

## Ringkasan hasil

| | |
|---|---|
| Data awal | 194 respons × 116 kolom |
| Setelah cleaning | **192 respons** (2 respons kosong total dibuang) |
| Dataset prepared | 192 baris × **228 kolom** |
| Data dictionary | 228 kolom terdokumentasi, bisa ditelusuri sampai kolom Excel asli |
| Cleaning log | 10 entri (C1–C9), lengkap dengan jumlah sebelum/sesudah dan alasan |
| Grafik EDA | 14 gambar di `figures/` |
| Responden dengan IPK valid | 173 |
| Straightliner (88 jawaban Likert identik) | 15, diberi flag dan **tidak dihapus** |

---

## Struktur folder

```
machine_learning_exam/
├── notebook/
│   └── KelpAdam_UTS_DataPreparation_EDA.ipynb   # seluruh proses, dijalankan berurutan
├── dataset/
│   ├── raw/                                     # data mentah dari dosen (TIDAK di-commit)
│   └── KelpAdam_survey_prepared.csv             # dataset hasil preparation
├── dictionary/
│   ├── KelpAdam_data_dictionary.xlsx            # data dictionary (5 lembar)
│   └── KelpAdam_cleaning_log.csv                # cleaning log
├── figures/                                     # 14 grafik EDA (PNG)
├── report/                                      # laporan UTS
├── recruitment/                                 # dokumentasi rekrutmen responden
├── Aturan_Scoring_Penjelasan.md                 # aturan scoring + referensi teori
├── Checklist_Spesifikasi_7.2_7.3.md             # pemetaan spesifikasi -> bagian notebook
├── Analisis_Dataset_dan_Strategi_Cleaning_EDA.md
├── KelpAdam_anggota.txt                         # anggota & pembagian tugas
├── requirements.txt
└── README.md
```

---

## Cara menjalankan

1. **Siapkan Python 3.12** dan virtual environment:
   ```bash
   python -m venv .venv
   .venv\Scripts\activate          # Windows
   # source .venv/bin/activate     # macOS / Linux
   pip install -r requirements.txt
   ```
2. **Letakkan data mentah** dari dosen (tidak disertakan di repo) di:
   ```
   dataset/raw/Survei_Gaya_Belajar_IPS_IPK_NO_PII.xlsx
   ```
   Bagian 1.1 notebook menampilkan hash MD5 file. Cocokkan dengan `390ab027d2a950f63a72684d795caa72` untuk memastikan file-nya sama.
3. **Buka notebook** `notebook/KelpAdam_UTS_DataPreparation_EDA.ipynb` di VS Code atau Jupyter, pilih kernel `.venv`, lalu jalankan **Restart → Run All**.
4. Notebook otomatis membuat ulang:
   - `dataset/KelpAdam_survey_prepared.csv`
   - `dictionary/KelpAdam_data_dictionary.xlsx` dan `dictionary/KelpAdam_cleaning_log.csv`
   - 14 grafik di `figures/`

Semua path bersifat relatif terhadap folder proyek, dan notebook sudah diuji berjalan dari awal sampai akhir tanpa error.

---

## Alur notebook

| Bagian | Isi | Kode sel |
|---|---|---|
| 0 – Setup | Path, aturan yang dipegang (data asli tidak diubah, outlier tidak dihapus otomatis) | – |
| 1 – Data Understanding | Ukuran, provenance (MD5), kelompok kolom, tipe data, missing, duplikat, format VARK, statistik Likert, IPS/IPK | 1.1–1.11 |
| 2 – Data Cleaning | Ganti nama kolom, buang respons kosong, validasi Likert & opsi VARK, IPS/IPK 0 → kosong, cek rentang & duplikat, flag straightliner, cleaning log | C1–C10 |
| 3 – Scoring | Skor & label sesuai teori tiap framework, plus fitur & label versi panduan UAS | S1–S5 |
| 4 – Data Preparation | Perkiraan angkatan, penyusunan dataset prepared, data dictionary | P1–P3 |
| 5 – EDA | Alur data, konteks, missing & pola jawaban, distribusi skor & label, korelasi, kesesuaian antar-framework, ringkasan temuan | E1–E8 |

---

## Aturan scoring

Scoring mengikuti teori asli tiap framework, sesuai arahan dosen. Detail lengkap dan referensinya ada di [`Aturan_Scoring_Penjelasan.md`](Aturan_Scoring_Penjelasan.md).

| Framework | Skor | Label teori (S1–S4) | Label versi panduan UAS (S5) |
|---|---|---|---|
| **VARK** (Fleming) | Jumlah pilihan V/A/R/K (0–16) | Profil multimodal dengan *stepping distance* | Modalitas tertinggi yang unik, atau Seri |
| **Felder-Silverman** (ILS) | Jumlah item kutub a − kutub b (−16…16) | Arah + kekuatan (ambang ILS diskalakan) | Active / Reflective / Setara |
| **Kolb** (LSI) | AC−CE dan AE−RO | 4 tipe, batas = median sampel | CE_RO / AC_RO / AC_AE / CE_AE / Batas |
| **Honey-Mumford** (LSQ) | Jumlah 8 item per gaya (8–40) | Pita persentil 10/20/40/20/10% | Gaya tertinggi yang unik, atau Seri |

Untuk UAS juga disiapkan **64 indikator VARK 0/1** (`VARK_Q01_V` … `VARK_Q16_K`). Label teori dan label UAS disimpan di kolom terpisah, sehingga keduanya bisa ditelusuri.

---

## Data dictionary

`dictionary/KelpAdam_data_dictionary.xlsx` berisi 5 lembar:

| Lembar | Isi |
|---|---|
| `cara_baca` | Panduan membaca dictionary dan contoh cara menelusuri kolom |
| `data_dictionary` | 228 kolom: jenis, arti, tipe data, nilai valid & teramati, terisi/kosong, raw/derived, **kolom Excel asli**, **dibentuk dari**, **langkah notebook**, sumber aturan |
| `ringkasan_jenis` | Jumlah kolom per jenis (Identifier, Metadata, Raw item, Score, Label, Prepared variable) |
| `langkah_notebook` | Arti kode langkah C1–C9, S1–S5, P1 |
| `cleaning_log` | Salinan cleaning log |

Contoh penelusuran: `HM_label_uas` (S5) ← `HM_ACT_sum` (S4) ← `HM_ACT_01`–`HM_ACT_08` ← kolom Excel R–Y.

---

## Temuan utama EDA

- Kekosongan data hanya **struktural** (IPS mengikuti angkatan, IPK mahasiswa baru) atau **disengaja** (label teori straightliner). Tidak ada imputasi.
- Ada kecenderungan menjawab "setuju": 68,9% seluruh jawaban Likert bernilai 4/5. Ini alasan skor berbasis selisih dan norma relatif dipakai.
- Konsistensi item tiap subskala memadai (Cronbach's alpha 0,70–0,86).
- Dengan aturan panduan UAS, label **VARK** dan **Felder** relatif seimbang. Label **Honey-Mumford** didominasi Seri (33%), dan **Kolb** didominasi Batas (36%).
- Skor gaya belajar **tidak berkorelasi signifikan dengan IPK** setelah koreksi FDR. Dari label, hanya tipe VARK yang berbeda dengan efek kecil.

---

## Data & privasi

- Sumber data adalah survei resmi mata kuliah yang dibagikan dosen setelah periode pengisian ditutup. File sudah tanpa identitas pribadi (*NO_PII*).
- Kolom `ID` hanya nomor urut untuk menelusuri baris, **bukan variabel analisis**.
- File mentah (`dataset/raw/`) tidak di-commit dan tidak boleh dipublikasikan. Dataset prepared tidak berisi identitas langsung.
- AI generatif dipakai sebagai alat bantu. Kelompok bertanggung jawab penuh atas kode, metode, dan interpretasi.

---

## Anggota kelompok

| Nama | Peran utama |
|---|---|
| Edward Christian Imanuel Tambunan | Kode data understanding, cleaning, scoring, EDA; penyebaran kuesioner |
| Andreas Dwitara Sidabariba | Analisis metode scoring berdasarkan teori asli dan kesesuaian dengan UAS; referensi jurnal; penyebaran kuesioner |
| Viriya Anatta Djasawibawa | Penyusunan laporan; referensi; penyebaran kuesioner |

Rincian pembagian tugas ada di `KelpAdam_anggota.txt`.
