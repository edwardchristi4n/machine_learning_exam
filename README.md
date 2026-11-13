# Preferensi Gaya Belajar: Data Preparation & EDA (UTS Pembelajaran Mesin)

Proyek UTS mata kuliah **Pembelajaran Mesin**, Gasal 2026/2027, Universitas Atma Jaya Yogyakarta. Pengerjaan oleh **Kelompok Adam (KelpAdam)**.
Dosen pengampu: Yohanes Sigit Purnomo, Ph.D. dan Theresia Devi Indriasari, Ph.D.

Proyek ini mengolah data survei preferensi gaya belajar dari empat framework (**VARK, Honey-Mumford, Felder-Silverman, Kolb**) beserta IPS/IPK mahasiswa. Tujuannya membuat dataset yang bersih, terdokumentasi, dan siap untuk tahap modelling (UAS). Isinya meliputi data understanding, cleaning, scoring sesuai panduan dosen, data preparation, dan exploratory data analysis (EDA).

> **Data mentah tidak ada di repository ini.** File survei dari dosen hanya untuk keperluan mata kuliah, sehingga dikecualikan lewat `.gitignore` (lihat [Data & privasi](#data--privasi)).

---

## Ringkasan hasil

| | |
|---|---|
| Data awal | 194 respons × 116 kolom |
| Setelah cleaning | **192 respons** (2 respons kosong total dibuang) |
| Dataset prepared | 192 baris × **216 kolom** (104 item mentah, 27 skor, 4 label, 64 indikator VARK) |
| Data dictionary | 216 kolom terdokumentasi + lembar aturan scoring, bisa ditelusuri sampai kolom Excel asli |
| Cleaning log | 10 entri (C1–C9), lengkap dengan jumlah sebelum/sesudah dan alasan |
| Grafik EDA | 13 gambar di `figures/` |
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
│   ├── KelpAdam_data_dictionary.xlsx            # data dictionary (6 lembar)
│   └── KelpAdam_cleaning_log.csv                # cleaning log
├── figures/                                     # 13 grafik EDA (PNG)
├── report/                                      # laporan UTS
├── recruitment/                                 # dokumentasi rekrutmen responden
├── Aturan_Scoring_Penjelasan.md                 # langkah scoring + dasar teori + referensi
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
   - 13 grafik di `figures/`

Semua path bersifat relatif terhadap folder proyek, dan notebook sudah diuji berjalan dari awal sampai akhir tanpa error.

---

## Alur notebook

| Bagian | Isi | Kode sel |
|---|---|---|
| 0 – Setup | Path, aturan yang dipegang (data asli tidak diubah, outlier tidak dihapus otomatis) | – |
| 1 – Data Understanding | Ukuran, provenance (MD5), kelompok kolom, tipe data, missing, duplikat, format VARK, statistik Likert, IPS/IPK | 1.1–1.11 |
| 2 – Data Cleaning | Ganti nama kolom, buang respons kosong, validasi Likert & opsi VARK, IPS/IPK 0 → kosong, cek rentang & duplikat, flag straightliner, cleaning log | C1–C10 |
| 3 – Scoring | 64 indikator VARK, skor per dimensi, dan satu label per framework sesuai panduan dosen; verifikasi manual | S1–S5 |
| 4 – Data Preparation | Perkiraan angkatan, penyusunan dataset prepared, data dictionary | P1–P3 |
| 5 – EDA | Alur data, konteks, missing & pola jawaban, distribusi skor & label (kelas minoritas), korelasi, IPK & fitur item untuk regresi, kesesuaian antar-framework, ringkasan temuan | E1–E8 |

---

## Aturan scoring

Sesuai arahan dosen dan asisten dosen (13 November 2026), aturan scoring UTS **mengikuti Panduan UAS Klasifikasi**, sehingga UTS dan UAS memakai satu jalur aturan. Struktur dimensi mengikuti teori tiap framework. Cut-off, norma, atau klasifikasi resmi tidak dipakai, karena instrumen kelas bukan versi resmi (*Catatan untuk Peneliti* di instrumen). Detail langkah dan referensinya ada di [`Aturan_Scoring_Penjelasan.md`](Aturan_Scoring_Penjelasan.md) dan lembar `aturan_scoring` di data dictionary.

| Framework (teori) | Skor | Label (satu y) |
|---|---|---|
| **VARK** (Fleming & Mills, 1992) | 64 indikator 0/1 → jumlah V/A/R/K (0–16) | Modalitas tertinggi yang unik; kalau sama tinggi → Seri |
| **Felder-Silverman** (Felder & Silverman, 1988) | Jumlah 4 item tiap sisi (4–20) untuk 4 pasangan; selisih untuk eksplorasi | Active > Reflective → Active; < → Reflective; sama → Setara |
| **Honey-Mumford** (Honey & Mumford, 1992) | Jumlah 8 item per gaya (8–40) | Gaya tertinggi yang unik; kalau sama tinggi → Seri |
| **Kolb** (Kolb, 1984) | Jumlah 6 item per tahap (6–30); d1 = AC − CE, d2 = AE − RO | Tanda d1, d2 → CE_RO / AC_RO / AC_AE / CE_AE; salah satu 0 → Batas |

Seri, Setara, dan Batas adalah kategori operasional tugas, bukan gaya belajar tambahan.

---

## Data dictionary

`dictionary/KelpAdam_data_dictionary.xlsx` berisi 6 lembar:

| Lembar | Isi |
|---|---|
| `cara_baca` | Panduan membaca dictionary dan contoh cara menelusuri kolom |
| `data_dictionary` | 216 kolom: jenis, arti, tipe data, nilai valid & teramati, terisi/kosong, raw/derived, **kolom Excel asli**, **dibentuk dari**, **langkah notebook**, **dasar teori**, sumber aturan |
| `aturan_scoring` | Per framework: dasar teori, item & kolom Excel, langkah pengodean, rumus skor, aturan label, catatan, referensi |
| `ringkasan_jenis` | Jumlah kolom per jenis (Identifier, Metadata, Raw item, Score, Label, Prepared variable) |
| `langkah_notebook` | Arti kode langkah C1–C9, S1–S5, P1 |
| `cleaning_log` | Salinan cleaning log |

Contoh penelusuran: `FS_ACT_REF_label` (S2) ← `FS_ACT_sum`, `FS_REF_sum` (S2) ← `FS_ACT_01`–`FS_ACT_04` ← kolom Excel AX–BA.

---

## Temuan utama EDA

- Kekosongan data hanya **struktural** (IPS mengikuti angkatan, IPK mahasiswa baru); semua item, skor, dan label lengkap. Tidak ada imputasi.
- Ada kecenderungan menjawab "setuju": 68,9% seluruh jawaban Likert bernilai 4/5. Ini alasan label dibentuk dengan membandingkan skor di dalam diri responden sendiri.
- Konsistensi item tiap subskala memadai (Cronbach's alpha 0,70–0,86).
- Label **VARK** dan **Felder** relatif seimbang. Label **Honey-Mumford** didominasi Seri (33%), dan **Kolb** didominasi Batas (36%); semua 15 straightliner jatuh ke Setara/Seri/Batas.
- IPK (n = 173) miring ke kiri (median 3,42). Skor maupun setiap item **tidak berkorelasi signifikan dengan IPK** setelah koreksi FDR (|rho| ≤ 0,23). Dari label, hanya VARK yang berbeda (kelompok Seri ber-IPK lebih tinggi, ε² = 0,09), dan ini bukan hubungan sebab-akibat.
- Asosiasi antar-framework lemah (Cramér's V ≤ 0,24), jadi framework tidak saling menggantikan.

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
