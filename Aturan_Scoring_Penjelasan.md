# Aturan Scoring – KelpAdam (versi 3, final)

Dokumen ini menjelaskan **langkah scoring** yang dipakai di notebook `notebook/KelpAdam_UTS_DataPreparation_EDA.ipynb` (Bagian 3, sel S1–S5) beserta dasar teori dan referensinya. Ringkasan yang sama ada di lembar `aturan_scoring` pada `dictionary/KelpAdam_data_dictionary.xlsx`.

---

## 1. Keputusan dan dasarnya

| Sumber | Isi | Dampak |
|---|---|---|
| **Arahan dosen & asisten dosen (13 Nov 2026)** | Pak Sigit: ikuti panduan yang dipublikasikan Bu Devi. Asdos: "pakai yang di UAS saja", supaya satu jalur dan tidak perlu hitung ulang di UAS | Aturan label UTS = aturan label UAS |
| **Panduan UAS Klasifikasi (Bagian 3–6)** | Aturan skor dan satu label y untuk tiap framework | Diikuti persis |
| **Instrumen survei, *Catatan untuk Peneliti*** | Instrumen kelas bukan versi resmi; skor per dimensi dipakai sebagai profil; jangan memakai cut-off, norma, atau klasifikasi resmi VARK, ILS, Kolb LSI, LSQ | Stepping distance VARK, ambang ILS, tipe resmi LSI, dan pita LSQ **tidak dipakai** |
| **Spesifikasi UTS (Bagian 6 & 7.3)** | Label harus bisa ditelusuri ke aturan scoring; gunakan aturan yang diberikan atau disetujui dosen; jangan membuat cut-off sendiri | Semua aturan berasal dari panduan dosen dan terdokumentasi |

**Riwayat:** versi 1–2 memakai aturan resmi tiap framework (atas arahan 6 Nov "sesuai teori"). Setelah instrumen dan panduan UAS dibaca, serta dikonfirmasi ke dosen dan asdos, aturan diganti ke versi panduan dosen. Teori tetap dipakai untuk **struktur dimensi** dan **rumus selisih**.

**Prinsip:** skor = profil kecenderungan per dimensi; label = satu kategori dari aturan panduan dosen. Seri, Setara, dan Batas adalah kategori operasional tugas, bukan gaya belajar tambahan. Skor ditulis sebagai **jumlah**; rerata yang disarankan instrumen = jumlah ÷ banyak item, jadi urutan dan tanda selisihnya sama.

---

## 2. Langkah per framework

### VARK (S1) – Panduan Bagian 6 & Lampiran
- **Teori:** Fleming & Mills (1992). Ada empat modalitas (Visual, Aural, Read/Write, Kinesthetic), dan beberapa bisa sama kuat.
- **Item:** 16 skenario, kolom Excel B–Q (`VARK_Q01`–`VARK_Q16`).
1. Pecah teks jawaban per `;`, lalu cocokkan ke opsi resmi: A = V, B = A, C = R, D = K (`VARK_MAP`).
2. Buat 4 indikator 0/1 per soal, total **64 fitur** (`VARK_Q01_V` … `VARK_Q16_K`). Sel kosong tetap kosong, tidak diisi empat nol.
3. Skor `VARK_V/A/R/K` = jumlah 16 indikator per modalitas (0–16); `VARK_total` = jumlah keempatnya.
4. Label `VARK_label` = modalitas dengan skor tertinggi yang unik; kalau sama tinggi → **Seri**. Kalau ada soal kosong → label kosong.

### Felder-Silverman (S2) – Panduan Bagian 3
- **Teori:** Felder & Silverman (1988). Ada empat pasangan dimensi bipolar. Skor ILS adalah selisih kedua sisi, dan preferensi adalah kontinum (Felder & Spurlin, 2005).
- **Item:** 32 item Likert, kolom AX–CC, 4 item per sisi.
1. Skor tiap sisi `FS_<sisi>_sum` = jumlah 4 jawaban (4–20), untuk ke-8 sisi (Active, Reflective, Sensing, Intuitive, Visual, Verbal, Sequential, Global).
2. Selisih `FS_<a>_<b>_score` = sisi pertama − sisi kedua (−16…+16), untuk eksplorasi.
3. Label `FS_ACT_REF_label`: Active > Reflective → **Active**; lebih rendah → **Reflective**; sama → **Setara**.
4. Tiga pasangan lain disimpan sebagai profil, tidak dijadikan label (panduan: "profil lengkap tidak direduksi menjadi satu tipe").

### Honey-Mumford (S4) – Panduan Bagian 4
- **Teori:** Honey & Mumford (1992). Ada empat gaya (Activist, Reflector, Theorist, Pragmatist), yang diturunkan dari siklus Kolb.
- **Item:** 32 item Likert, kolom R–AW, 8 item per gaya.
1. Skor `HM_<gaya>_sum` = jumlah 8 jawaban (8–40).
2. Label `HM_label` = gaya dengan skor tertinggi yang unik; kalau dua atau lebih sama tinggi → **Seri**.

### Kolb (S3) – Panduan Bagian 5
- **Teori:** Kolb (1984). Siklus belajar CE → RO → AC → AE. Sumbu AC−CE (cara memahami) dan AE−RO (cara mengolah) membentuk empat kuadran.
- **Item:** 24 item Likert, kolom CD–DA, 6 item per tahap.
1. Skor `KOLB_<tahap>_sum` = jumlah 6 jawaban (6–30).
2. d1 = `KOLB_ACCE` = AC − CE; d2 = `KOLB_AERO` = AE − RO (−24…+24).
3. Label `KOLB_label` dari tanda d1 dan d2:

| d1 | d2 | Label | Makna konsep (panduan) |
|---|---|---|---|
| − | − | CE_RO | Diverging |
| + | − | AC_RO | Assimilating |
| + | + | AC_AE | Converging |
| − | + | CE_AE | Accommodating |
| salah satu = 0 | apa pun | **Batas** | Tidak dipaksakan ke kuadran |

### Verifikasi (S5)
Beberapa responden dicek manual (skor → label), termasuk kasus Seri, Setara, dan Batas. Hitungan VARK dari indikator juga dicocokkan dengan hitungan langsung dari teks jawaban.

---

## 3. Straightliner
Ke-15 responden yang menjawab 88 item Likert dengan angka sama **tetap diberi skor dan label**, karena aturan panduan bisa menghitungnya. Hasilnya, semuanya menjadi Setara (Felder), Seri (HM), dan Batas (Kolb). Mereka ditandai `flag_straightline`, dikeluarkan dari analisis skor skala 1–5 di EDA, dan keputusan memakai atau mengecualikan mereka saat modelling diambil di UAS.

---

## 4. Hasil pada data (n = 192)

| Label | Distribusi | Seri/Setara/Batas |
|---|---|---|
| `VARK_label` | Kinesthetic 51, Aural 42, Visual 38, Read/Write 31, Seri 30 | 16% |
| `FS_ACT_REF_label` | Active 67, Reflective 64, Setara 61 | 32% |
| `HM_label` | Seri 64, Pragmatist 43, Reflector 36, Theorist 31, Activist 18 | 33% |
| `KOLB_label` | Batas 70, CE_RO 42, CE_AE 32, AC_RO 30, AC_AE 18 | 36% |

---

## 5. Referensi

**Aturan dan batasan**
- Panduan UAS Klasifikasi Preferensi Belajar, Pembelajaran Mesin Gasal 2026/2027 (Bagian 3–6 dan Lampiran opsi VARK).
- Instrumen Survei Penelitian *Learning Analytics* UAJY v1.0 (September 2026), bagian *Catatan untuk Peneliti*.

**Teori**
- Fleming, N. D., & Mills, C. (1992). Not another inventory, rather a catalyst for reflection. *To Improve the Academy, 11*, 137–155.
- Felder, R. M., & Silverman, L. K. (1988). Learning and teaching styles in engineering education. *Engineering Education, 78*(7), 674–681.
- Kolb, D. A. (1984). *Experiential Learning: Experience as the Source of Learning and Development.* Prentice-Hall.
- Honey, P., & Mumford, A. (1992). *The Manual of Learning Styles.* Peter Honey.

**Contoh penelitian dengan komponen metode serupa**
- Azzeh, M., Altamimi, A. M., & Albashayreh, M. (2022). *Predicting Students' Learning Styles Using Regression Techniques.* arXiv:2209.12691. Memakai 16 soal VARK yang dikodekan sebagai vektor biner, label = gaya dengan nilai tertinggi, dan model SVM, Random Forest, dan lainnya.
- Felder, R. M., & Spurlin, J. (2005). Applications, reliability and validity of the Index of Learning Styles. *International Journal of Engineering Education, 21*(1), 103–112. Skor dihitung dari jawaban "a" dikurangi "b", dan preferensi diperlakukan sebagai kontinum.
- Brower, K. A., Stemmans, C. L., Ingersoll, C. D., & Langley, D. J. (2001). An investigation of undergraduate athletic training students' learning styles and program admission success. *Journal of Athletic Training, 36*(2), 130–135. AC−CE dan AE−RO diplot pada dua sumbu dan empat kuadran.
- Czepula, A. I., dkk. (2016). Predominant learning styles among pharmacy students at the Federal University of Paraná, Brazil. *Pharmacy Practice, 14*(1), 650. Skor tiap gaya = jumlah respons (norma persentil di studi ini tidak dipakai).
- Rafi, A., Musdholifah, A., & Wardoyo, R. (2024). Identifying student learning styles using support vector machine in Felder-Silverman model. *Journal of Applied Data Sciences, 5*(3), 1495–1507.

Catatan: kombinasi persis "instrumen Likert kelas + kategori Seri/Setara/Batas" adalah aturan operasional dosen untuk tugas ini. Literatur di atas mendukung komponen metodenya, bukan kombinasi persisnya.
