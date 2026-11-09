# Analisis Dataset & Strategi Cleaning–EDA
**Proyek UTS Pembelajaran Mesin – Preferensi Gaya Belajar (Gasal 2026/2027)**
File sumber: `Survei_Gaya_Belajar_IPS_IPK_NO_PII.xlsx` · Dianalisis: 6 November 2026 · Tenggat pengumpulan: Kamis, 12 November 2026, 23.59 WIB

> Dokumen ini adalah **audit awal dan rencana kerja**, bukan hasil EDA final. Angka skor subskala di Bagian 4 hanya pratinjau eksploratif. Spesifikasi (Bagian 6 & 7.3) melarang membuat aturan scoring/cut-off sendiri, jadi skor resmi baru boleh dihitung setelah aturan dari dosen dikonfirmasi.

---

## 0. Ringkasan temuan utama

1. **Dataset berisi item mentah, belum ada skor atau label.** Ada 194 baris × 116 kolom dalam 1 sheet: 16 item VARK (pilihan ganda multi-jawab), 88 item Likert 1–5 (Honey-Mumford, Felder-Silverman, Kolb), 10 kolom IPS, dan 1 kolom IPK. Tidak ada kolom prodi, angkatan, semester, atau timestamp.
2. **Tidak ada PII langsung.** Kolom `ID` hanya nomor urut 1–194 (unik, tanpa celah).
3. **Ada 2 respons kosong total (ID 11 dan 133).** Seluruh 104 item survei, IPS, dan IPK kosong. Hanya ini sumber missing di kolom survei, jadi setelah keduanya dihapus, **192 responden memiliki jawaban survei lengkap**.
4. **Isian survei secara format sangat bersih.** Semua Likert bernilai bulat 1–5 tanpa nilai di luar rentang. Semua jawaban VARK hanya memakai 4 opsi baku per item, tanpa isian bebas.
5. **15 responden (7,8%) melakukan straightlining.** Mereka menjawab ke-88 item Likert dengan angka yang sama persis (5 orang semua "3", 5 orang semua "4", 5 orang semua "5"). Ada **8 responden lagi** yang hampir sama (84–87 dari 88 jawaban identik). Ini isu kualitas data terbesar.
6. **IPS 2026 Semester 1 bernilai 0,0 untuk seluruh 173 baris yang terisi.** Semester ini masih berjalan, jadi angka 0 berarti "belum ada nilai", bukan IPS nol. Hal yang sama berlaku untuk **4 IPK = 0** (ID 20, 29, 59, 82), yaitu mahasiswa baru tanpa IPS sebelumnya.
7. **Kutub yang "berlawanan" justru berkorelasi positif** (ρ = 0,34–0,71). Contohnya FS Active vs Reflective, atau Kolb CE vs AC. Ini menandakan **kecenderungan setuju umum (acquiescence)**: 68,9% seluruh jawaban adalah 4/5. Selain itu, item disusun sebagai Likert independen, bukan pilihan paksa seperti instrumen aslinya. Konsekuensinya, label "gaya dominan" sangat sensitif terhadap aturan scoring dan **tidak boleh dibuat tanpa aturan dosen**.
8. **Hubungan gaya belajar dengan IPK lemah.** Semua |ρ| ≤ 0,20. Tujuh dari 20 korelasi punya p < 0,05, tetapi **tidak ada yang bertahan setelah koreksi uji berganda** (Benjamini-Hochberg, q min = 0,134).

---

## 1. Profil dataset

| Aspek | Temuan |
|---|---|
| Ukuran | 194 baris × 116 kolom, 1 sheet (`Sheet1`) |
| Identifier | `ID` integer 1–194, unik, berurutan, tanpa celah. Bukan PII (nomor urut). |
| Header | Teks pertanyaan lengkap, panjang sampai >100 karakter. Perlu di-rename. |
| Kolom 1–16 | VARK: 16 item multi-jawab. Teks opsi digabung dengan pemisah `;` (selalu diakhiri `;`). |
| Kolom 17–104 | 88 item Likert 1–5 (float di Excel, isinya bulat) |
| Kolom 105–114 | IPS per semester: 2022 S1 s.d. 2026 S1, termasuk "2025 Semester 3" |
| Kolom 115 | IPK |
| Metadata konteks | **Tidak ada** (prodi, angkatan, semester, waktu pengisian) |

### 1.1 Struktur instrumen (hasil penelusuran isi item)

Pengelompokan di bawah **disimpulkan dari urutan kolom dan isi pernyataan**, bukan dari dokumen resmi. Kesimpulan ini didukung data: untuk ke-16 kelompok, korelasi rata-rata antar-item di dalam kelompok selalu lebih tinggi daripada korelasi ke item di luar kelompok. Meski begitu, pemetaan resmi tetap perlu dikonfirmasi ke dosen (lihat Lampiran A).

| Framework | Kolom | Subskala (dugaan) | Item/subskala | Cronbach α (n=192) | α tanpa straightliner (n=177) |
|---|---|---|---|---|---|
| VARK | 1–16 | V, A, R, K (1 opsi per modalitas di tiap item) | 16 item × 4 opsi | – | – |
| Honey-Mumford | 17–48 | Activist, Reflector, Theorist, Pragmatist | 8 | 0,79–0,87 | 0,75–0,85 |
| Felder-Silverman | 49–80 | Active/Reflective, Sensing/Intuitive, Visual/Verbal, Sequential/Global (8 kutub) | 4 | 0,74–0,83 | 0,70–0,81 |
| Kolb | 81–104 | CE, RO, AC, AE | 6 | 0,80–0,88 | 0,77–0,86 |

Semua α berada di sekitar 0,70 ke atas (terendah 0,699 untuk FS_Global tanpa straightliner), jadi konsistensi internal memadai. α turun sedikit (sekitar 0,02–0,04) setelah straightliner dikeluarkan. Artinya straightliner ikut menggelembungkan α secara artifisial.

---

## 2. Temuan per kelompok kolom

### 2.1 VARK (16 item multi-jawab)
- Format seragam: semua sel diakhiri `;`. Setelah di-split, setiap item hanya punya **4 opsi unik**, dan masing-masing jelas mewakili satu modalitas V/A/R/K. Tidak ada typo, variasi kapitalisasi, atau opsi "Lainnya".
- Jumlah opsi dipilih per item 1–4. Tidak ada item yang dijawab kosong (selain 2 baris kosong total).
- Total pilihan per responden 16–64, median 31. Ada 19 responden yang selalu memilih tepat 1 opsi. ID 55 memilih **keempat opsi di semua 16 item**, dan ID ini juga straightliner Likert.
- Hitungan per modalitas (0–16): rata-rata V 7,66, A 8,14, R 7,28, K 8,07. Sebarannya relatif seimbang, tanpa satu modalitas yang mendominasi kelas.
- Opsi paling populer: Q12 "menjelaskan secara lisan" (140), Q3 "berlatih soal/mencoba contoh" (131), Q14 "ditunjukkan secara visual/markup" (130).
- Pemetaan otomatis berbasis kata kunci perlu **dikoreksi manual pada 3 opsi**:
  - Q4 "contoh verbal dan tanya jawab" → **A** (bukan K)
  - Q5 "contoh hasil akhir atau struktur visual" → **V**
  - Q10 "membicarakan masalah dengan orang lain" → **A**

  Pemetaan final ada di Lampiran A.1.

### 2.2 Likert (88 item)
- Semua nilai ∈ {1, 2, 3, 4, 5}, tanpa desimal. Missing hanya pada 2 baris kosong.
- Distribusi seluruh sel (192 × 88 = 16.896): 1 = 166 (1,0%), 2 = 710 (4,2%), 3 = 4.387 (26,0%), 4 = 6.862 (40,6%), 5 = 4.771 (28,2%). **Jawaban sangat condong ke "setuju"** (skew item −0,97 s.d. 0,03).
- Rata-rata item 3,26–4,24. Paling rendah: "membuat catatan/refleksi setelah belajar" (3,26) dan "nyaman dengan gagasan teoritis tanpa contoh" (3,32). Paling tinggi: "argumen konsisten dan beralasan jelas" (4,24).
- Persentase setuju (4/5) per item 38,5%–82,3%. Tidak ada item yang ≥ 85% (tidak ada ceiling ekstrem).
- Semua item ditulis **positif** (tidak ada item terbalik/reverse-coded). Akibatnya bias setuju tidak bisa dikoreksi lewat item terbalik.

### 2.3 IPS & IPK

| Kolom | n terisi | Rentang | Catatan |
|---|---|---|---|
| IPS 2022 S1–S2 | 1 | 3,03–3,13 | Hanya 1 mahasiswa (angkatan 2022) |
| IPS 2023 S1–S2 | 18 | 2,68–4,00 | |
| IPS 2024 S1–S2 | 125 | 0,95–4,00 | |
| IPS 2025 S1 / S2 | 173 / 172 | 0,87–4,00 | |
| IPS 2025 S3 | 16 | 3,00–4,00 | Kemungkinan **semester pendek/antara**. Perlu konfirmasi. |
| IPS 2026 S1 | 173 | **semua 0,0** | Semester berjalan, jadi bukan nilai valid |
| IPK | 177 | 0,00–4,00 | 4 baris = 0 (mahasiswa baru), 17 kosong |

- Semua nilai berada dalam rentang [0, 4]. IPS memakai sampai 6 desimal (hasil pembagian bobot SKS), IPK 2 desimal.
- **IPK konsisten dengan IPS.** Selisih IPK − rata-rata IPS: rata-rata 0,009, maksimum |0,21|. Selisih ini wajar karena IPK berbobot SKS. Jadi IPK dapat dipercaya sebagai data sistem.
- **Nilai rendah tetapi valid:** IPS < 2 pada ID 4, 7, 30, 112, 128, 134, 175, dan IPK minimum 0,87 (ID 30). Ini nilai ekstrem yang mungkin, bukan error. Sesuai spesifikasi 7.2, **jangan dihapus**.
- **Missing IPK/IPS bersifat struktural, bukan acak:**
  - 8 responden tanpa IPS dan IPK sama sekali (ID 17, 83, 127, 144, 169, 192, 193, 194)
  - 11 mahasiswa baru yang hanya punya IPS 2026 S1 = 0: 4 di antaranya IPK = 0, 7 lainnya IPK kosong. Pengodean tidak konsisten untuk kondisi yang sama.
  - 11 mahasiswa lama tanpa entri IPS 2026 S1 (mungkin cuti atau sudah lulus)
- **Angkatan dapat diinferensikan** dari tahun IPS pertama yang tercatat: 2022 = 1, 2023 = 17, 2024 = 107, 2025 = 48, 2026 = 11, tidak diketahui = 8. Ini *inferensi*, bukan data resmi (mahasiswa transfer bisa terklasifikasi salah), dan harus ditandai sebagai derived variable.

---

## 3. Temuan kualitas data (dasar keputusan cleaning)

| # | Isu | Jumlah | ID | Rekomendasi | Dasar spesifikasi |
|---|---|---|---|---|---|
| Q1 | Respons kosong total | 2 | 11, 133 | **Hapus**, catat di cleaning log (194 → 192) | 7.2 "respons kosong" |
| Q2 | Straightlining 88 item Likert | 15 | 9, 38, 55, 57, 64, 68, 115, 125, 132, 136, 159, 167, 171, 174, 194 | **Jangan hapus.** Beri flag `flag_straightline`, lalu lakukan analisis sensitivitas | 7.2 "tidak boleh otomatis dihapus"; keputusan harus berdasar alasan |
| Q3 | Hampir-straightline (84–87 dari 88 identik) | 8 | 42, 44, 78, 126, 142, 166, 187, 188 | Flag `flag_near_straight`, laporkan saja | idem |
| Q4 | IPS 2026 S1 = 0 (semester berjalan) | 173 | – | Recode 0 → NaN dan dokumentasikan; kolom bisa dikeluarkan dari analisis | 7.2 nilai tidak valid / tidak mungkin |
| Q5 | IPK = 0 pada mahasiswa tanpa IPS sebelumnya | 4 | 20, 29, 59, 82 | Recode → NaN agar konsisten dengan 7 kasus serupa | 7.2 konsistensi |
| Q6 | IPS/IPK sangat rendah | 7 | 4, 7, 30, 112, 128, 134, 175 | **Pertahankan.** Ini nilai valid yang ekstrem; tandai di EDA | 7.2 bedakan ekstrem vs tidak mungkin |
| Q7 | Duplikat persis | 0 | – | Tidak ada setelah Q1. Dokumentasikan bahwa sudah dicek | 7.2 |
| Q8 | Kemiripan tinggi antar-baris | 2 pasang | 78–188, 125–132 | **Bukan duplikat**: IPS/IPK dan jawaban VARK berbeda, jadi kemungkinan orang berbeda yang sama-sama menjawab seragam. Catat saja | 7.2 |
| Q9 | Header panjang | 116 | – | Rename ke kode pendek; teks asli disimpan di data dictionary | 7.3 |
| Q10 | Missing IPS/IPK struktural | lihat 2.3 | – | Jangan diimputasi. Analisis IPK memakai n = 173 yang valid | 7.2, 8 |
| Q11 | Risiko re-identifikasi (kuasi-identifier) | 1 | angkatan 2022 (n = 1) | Gabungkan angkatan ≤ 2023 saat pelaporan; jangan tampilkan baris individual dengan urutan IPS lengkap | 3.6, 6 |

### 3.1 Mengapa straightliner di-flag, bukan otomatis dibuang
- Alasannya **berbasis pola respons, bukan karena preferensinya "tidak biasa"** (larangan di spesifikasi Bagian 7). Menjawab "5" pada semua 88 item berarti setuju penuh pada pernyataan yang saling bertolak belakang, misalnya *"suka langsung mencoba tanpa menunggu"* sekaligus *"cenderung mengamati dan menganalisis sebelum mencoba"*.
- Ke-15 straightliner tetap menjawab VARK dengan variasi (16–64 pilihan). Artinya **data VARK mereka mungkin tetap informatif**, sehingga eksklusi cukup diterapkan pada analisis Likert.
- Rekomendasi: pertahankan semua baris di dataset prepared dengan kolom flag. Lalu:
  - **Analisis utama Likert:** n = 177 (tanpa straightliner)
  - **Analisis VARK & IPK:** n = 192 (semua responden)
  - **Sensitivitas:** tampilkan tabel ringkas yang membandingkan hasil kunci (α, korelasi) dengan dan tanpa straightliner

  Keputusan ini dicatat di cleaning log. Sebaiknya dikonfirmasi juga ke dosen.

---

## 4. Pratinjau pola (eksploratif, skor belum resmi)

> Skor di bawah memakai **rata-rata item per subskala** dan **hitungan pilihan per modalitas VARK** sebagai ringkasan eksploratif. Ini bukan aturan scoring resmi. Pakai di laporan hanya jika disetujui dosen.

**Rata-rata subskala (n = 192):** semua berada di 3,60–4,04. Paling rendah FS_Intuitive (3,60), paling tinggi FS_Sensing (4,04). Perbedaan antar-subskala kecil dibanding variasi antar-orang (SD sekitar 0,6–0,75).

**Kutub berlawanan berkorelasi positif** (Spearman, n = 177):

| Pasangan | ρ |
|---|---|
| FS Active – Reflective | 0,45 |
| FS Sensing – Intuitive | 0,39 |
| FS Visual – Verbal | 0,39 |
| FS Sequential – Global | 0,34 |
| Kolb CE – AC | 0,63 |
| Kolb RO – AE | 0,69 |
| HM Activist – Reflector | 0,48 |
| HM Theorist – Pragmatist | 0,71 |

Interpretasi: responden yang "suka setuju" cenderung tinggi di semua subskala. Selain itu, instrumen asli Felder (ILS) dan Kolb (LSI) memakai pilihan paksa/ranking, sedangkan di sini setiap pernyataan dinilai terpisah. **Jangan berasumsi kedua kutub saling meniadakan.**

**Validitas konvergen antar-framework** (n = 177):
- **Kuat untuk konstruk yang mirip.** HM Activist ↔ FS Active ρ = 0,67; HM Theorist ↔ Kolb AC 0,66; HM Pragmatist ↔ Kolb AE 0,61; HM Reflector ↔ Kolb RO 0,52.
- **Lemah antara VARK dan Likert.** VARK_V ↔ FS_Visual ρ = 0,26; VARK_R ↔ FS_Verbal 0,14 (tidak signifikan); VARK_K ↔ Kolb_CE 0,05. Kemungkinan karena format jawabannya berbeda (multi-pilih vs Likert) dan konstruknya memang berbeda.

  Bahan diskusi yang menarik: **framework yang tampak serupa belum tentu mengukur hal yang sama** (sesuai spesifikasi Bagian 8).

**Gaya belajar vs IPK** (n = 160, tanpa straightliner dan IPK tidak valid):
- Semua |ρ| ≤ 0,20. Tertinggi FS_Reflective +0,20, VARK_A +0,19, dan HM_Activist +0,18.
- Tujuh korelasi p < 0,05 tanpa koreksi, tetapi **0 yang signifikan setelah koreksi FDR**.
- Kesimpulan yang aman: *tidak ada bukti hubungan yang berarti di sampel ini*. Selain itu, korelasi bukan kausalitas, dan IPK sangat menceng kiri (skew −1,64), jadi gunakan Spearman.

**IPK per angkatan (inferensi):** 2023 rata-rata 3,60 (n = 17), 2024 3,36 (n = 107), 2025 3,29 (n = 48). Ukuran grup tidak seimbang, jadi perbandingan ini hanya deskriptif.

---

## 5. Hal yang harus dikonfirmasi ke dosen (segera, hari ini)

1. **Aturan scoring resmi** untuk tiap framework:
   - VARK: hitungan per modalitas? aturan multimodal?
   - Felder: selisih kutub?
   - Kolb: AC−CE dan AE−RO, lalu kuadran?
   - Honey-Mumford: norma?

   Ditanyakan juga apakah label "gaya dominan" boleh dibuat dan dengan aturan apa.
2. **Pemetaan resmi item → subskala** (Lampiran A), terutama pemetaan opsi VARK.
3. **Makna "IPS 2025 Semester 3"** (semester pendek?) dan IPS 2026 S1 = 0 (semester berjalan?).
4. **IPK = 0 vs kosong** untuk mahasiswa baru: apakah keduanya berarti "belum ada IPK"?
5. **Penanganan straightliner:** apakah flag + analisis sensitivitas sudah sesuai harapan?
6. Apakah IPS/IPK boleh disertakan di `KelpX_survey_prepared.csv`, dan format penamaan ZIP di LMS.

> Jika aturan scoring belum keluar sampai ±9 Nov, jalankan **Jalur B** (Bagian 7): EDA di level item dan frekuensi opsi, ditambah ringkasan subskala yang diberi label jelas "eksploratif, belum skor resmi". Tidak boleh ada label/kategori gaya buatan sendiri.

---

## 6. Strategi data cleaning (berurutan, semua lewat kode)

Prinsip dari spesifikasi:
- File asli read-only.
- Setiap langkah tercatat di `cleaning_log`: langkah, alasan, jumlah sebelum/sesudah, dan ID terdampak.
- Tidak ada edit manual di Excel.

| Langkah | Aksi | Output/validasi |
|---|---|---|
| C0 | Salin file asli ke `dataset/raw/` (read-only), catat hash MD5, lalu baca dengan pandas | Bukti provenance |
| C1 | **Rename kolom** sesuai skema Lampiran A (`VARK_Q01`, `HM_ACT_01`, `FS_SEN_03`, `KOLB_AE_06`, `IPS_2024_S1`, `IPK`). Simpan tabel mapping nama baru ↔ teks asli | Tabel mapping → data dictionary |
| C2 | Hapus baris kosong total (ID 11, 133) | 194 → 192; log |
| C3 | Validasi Likert: cek nilai ∈ {1..5}, cast ke `Int64` | Assert 0 pelanggaran |
| C4 | VARK: `strip`, split `;`, buang string kosong, validasi setiap opsi ada di daftar 4 opsi resmi item tersebut | Assert 0 opsi asing |
| C5 | IPS 2026 S1: recode 0,0 → NaN, beri catatan "semester berjalan" | 173 sel diubah; log |
| C6 | IPK = 0 tanpa IPS sebelumnya → NaN | 4 sel (ID 20, 29, 59, 82); log |
| C7 | Validasi rentang IPS/IPK ∈ [0, 4] dan konsistensi IPK ≈ rata-rata IPS (toleransi ±0,3) | Assert; daftar outlier valid (tidak dihapus) |
| C8 | Cek duplikat: (a) persis, (b) semua item survei, (c) kemiripan Likert ≤ 5 beda, dikombinasi dengan VARK dan IPS | Laporkan 0 duplikat; 2 pasang mirip = orang berbeda |
| C9 | Flag kualitas: `flag_straightline` (SD = 0 pada 88 item), `flag_near_straight` (≥ 84 jawaban identik), `longstring_max`, `row_sd` | 15 + 8 baris; tidak dihapus |
| C10 | Simpan `cleaning_log` sebagai DataFrame/CSV dan tampilkan tabel sebelum-sesudah | Untuk laporan |

**Yang tidak dilakukan (dan alasannya dicatat):**
- Tidak ada imputasi IPS/IPK, karena missing bersifat struktural.
- Tidak ada penghapusan outlier IPK.
- Tidak ada recode Likert.
- Tidak ada pembulatan IPS di dataset prepared. Pembulatan hanya untuk tampilan.

---

## 7. Strategi data preparation

**Jalur A (aturan dosen tersedia):** implementasikan aturan secara eksplisit dalam satu fungsi per framework, cantumkan sumber aturannya di docstring dan data dictionary, lalu validasi rentang hasilnya. Contoh: hitungan VARK 0–16, rata-rata subskala 1–5.

**Jalur B (aturan belum tersedia):** buat hanya turunan yang netral dan dapat ditelusuri:
- One-hot VARK per opsi: `VARK_Q01_V` … `VARK_Q16_K` (64 kolom biner). Ini representasi ulang tanpa mengubah makna.
- `VARK_nsel_Qxx`: jumlah opsi dipilih per item.
- Ringkasan eksploratif (`*_mean_explor`), diberi tanda di dictionary sebagai "belum skor resmi".

**Derived variables lain (dua jalur):**
- `angkatan_inferred`: tahun IPS pertama; kategori "≤2023", "2024", "2025", "2026", "unknown"
- `n_semester_ips`: jumlah semester ber-IPS (tanpa 2026 S1)
- `ipk_valid`: IPK setelah C6
- `acq_index`: rata-rata 88 item Likert, sebagai indikator bias setuju (untuk EDA kualitas)
- Kolom flag dari C9

**Struktur output:**
- `dataset/KelpX_survey_prepared.csv`: ID + item mentah ter-rename + derived + flag. Raw dan prepared dipisah.
- `dictionary/KelpX_data_dictionary.xlsx` dengan kolom: `nama_kolom` · `teks_asli` · `framework` · `subskala` · `jenis` (metadata/raw item/score/label/prepared, sesuai tabel Bagian 7.3 spesifikasi) · `tipe_data` · `nilai_valid` · `raw_atau_derived` · `sumber_aturan` · `catatan`. Dictionary ini sebaiknya di-generate dari kode, dari tabel mapping C1.

---

## 8. Strategi EDA (setiap grafik menjawab satu pertanyaan)

| # | Pertanyaan | Metode/visual | Catatan interpretasi |
|---|---|---|---|
| E1 | Berapa data awal, berapa yang dipertahankan, dan kenapa? | Tabel alur 194 → 192 + jumlah flag | Kaitkan ke cleaning log |
| E2 | Siapa respondennya? | Bar `angkatan_inferred`; histogram + boxplot IPK | Angkatan = inferensi; gabung ≤ 2023; tanpa prodi, generalisasi terbatas |
| E3 | Di mana missing terjadi dan apakah acak? | Bar/heatmap missing per kelompok kolom; tabel pola IPS per angkatan | Missing IPS = struktural (angkatan), bukan non-respons |
| E4 | Bagaimana kualitas pola respons? | Histogram `row_sd`/`longstring_max`; distribusi `acq_index`; tandai straightliner | Dasar keputusan flag |
| E5 | Bagaimana distribusi jawaban Likert per subskala? | Diverging stacked bar per item, dikelompokkan per subskala | Kecenderungan setuju, item terendah/tertinggi |
| E6 | Apakah item dalam subskala konsisten? | Tabel Cronbach α + corrected item-total, dengan & tanpa straightliner | α ≥ 0,7 memadai; α turun tanpa SL |
| E7 | Opsi VARK mana yang paling dipilih? | Heatmap 16 item × 4 modalitas (% dipilih); bar jumlah opsi per item | Multi-jawab: total > 100% |
| E8 | Bagaimana distribusi skor per framework? | Box/violin per subskala (Jalur A: skor resmi; Jalur B: eksploratif) | Sebutkan aturan yang dipakai |
| E9 | Bagaimana distribusi label & ada class imbalance? | Bar label + proporsi (hanya Jalur A) | Jika tidak ada label resmi, tulis eksplisit "tidak dibuat" |
| E10 | Bagaimana hubungan antar-subskala dalam satu framework? | Heatmap Spearman | Kutub berlawanan berkorelasi positif, jadi bahas acquiescence dan format instrumen |
| E11 | Apakah framework berbeda mengukur hal serupa? | Heatmap Spearman lintas-framework (HM ↔ Kolb ↔ FS ↔ VARK count) | Jangan anggap konstruk identik; VARK lemah ke Likert |
| E12 | Adakah hubungan gaya belajar dengan IPK? | Spearman + koreksi FDR; scatter dengan garis LOWESS untuk 2–3 pasangan teratas | Lemah, tidak signifikan setelah koreksi, bukan kausal |
| E13 | Apakah kesimpulan berubah tanpa straightliner? | Tabel perbandingan α dan korelasi kunci, n = 192 vs 177 | Bukti robustness |

**Aturan visual:** setiap grafik punya judul berupa pertanyaan atau temuan, label sumbu, legend, n, dan 2–3 kalimat interpretasi tentang *apa yang bisa* dan *apa yang tidak bisa* disimpulkan. Simpan ke `figures/`.

### 8.1 Pemilihan metode sesuai tipe data (hindari korelasi numerik pada nominal)

| Pasangan tipe data | Metode yang tepat | Hindari |
|---|---|---|
| Likert item (ordinal) | Frekuensi, median, IQR, diverging bar | Rata-rata sebagai satu-satunya ringkasan |
| Skor subskala (rata-rata Likert, kuasi-interval) | Mean ± SD dan median; box/violin | – |
| Skor vs skor / vs IPK | **Spearman ρ** (IPK menceng, data ordinal) + koreksi FDR | Pearson tanpa cek asumsi |
| Hitungan VARK (count 0–16) | Spearman; histogram/box | – |
| Opsi VARK biner (dipilih/tidak) | Proporsi; φ / Cramér's V antar-opsi | Korelasi Pearson pada string |
| Label kategorikal vs label | Tabel kontingensi, χ²/Fisher, **Cramér's V** | Korelasi numerik |
| Label vs numerik (IPK/skor) | Box per grup, Kruskal-Wallis, effect size ε² | ANOVA pada grup sangat kecil |
| Reliabilitas | Cronbach α, corrected item-total r | – |

---

## 9. Rencana kerja sampai tenggat (Jum 6 – Kam 12 Nov)

| Hari | Target |
|---|---|
| **Jum 6 Nov** | Kirim pertanyaan ke dosen (Bagian 5). Siapkan struktur folder dan kerangka notebook. Kerjakan C0–C10. |
| Sab 7 Nov | Selesaikan preparation (Jalur B dulu), data dictionary generator, dan cleaning log |
| Min 8 Nov | EDA E1–E7 |
| Sen 9 Nov | EDA E8–E13. Pindah ke Jalur A jika aturan dosen sudah ada. |
| Sel 10 Nov | Interpretasi dan draf laporan (data understanding, cleaning log, preparation, EDA, keterbatasan) |
| Rab 11 Nov | *Restart & Run All* untuk cek reproducibility. Dokumentasi rekrutmen, `KelpX_anggota.txt`, review silang antar-anggota. |
| Kam 12 Nov | Cadangan waktu, susun ZIP sesuai format LMS, kumpulkan sebelum 23.59 WIB |

**Kerangka notebook `KelpX_UTS_DataPreparation_EDA.ipynb`:**
0. Setup & path relatif
1. Provenance & aturan privasi
2. Data understanding
3. Cleaning (C0–C10) + log
4. Preparation & derived variables
5. Data dictionary (generate & simpan)
6. EDA (E1–E13)
7. Ringkasan temuan & keterbatasan
8. Ekspor output

---

## 10. Keterbatasan yang perlu ditulis di laporan

- Tidak ada metadata prodi, semester, atau waktu pengisian, sehingga tidak bisa mendeteksi *speeder* maupun menganalisis per prodi. Angkatan hanya hasil inferensi.
- Semua item Likert berarah positif, sehingga bias setuju tidak bisa dipisahkan dari preferensi sebenarnya.
- Instrumen versi Likert berbeda dari format asli Felder ILS/Kolb LSI (pilihan paksa). Hasilnya tidak bisa dibandingkan langsung dengan norma instrumen asli.
- Sampel non-probabilitas (rekrutmen sukarela, jaringan kelompok) dan didominasi angkatan 2024 (n = 107).
- Pemetaan item → subskala masih dugaan sampai dikonfirmasi dosen.
- Analisis IPK hanya pada n = 173 (atau 160 tanpa straightliner). Hubungannya korelasional, bukan kausal.

---

## Lampiran A – Usulan pemetaan item

### A.1 Usulan pemetaan opsi VARK (perlu dikonfirmasi dosen)

| Item | Kolom usulan | V | A | R | K |
|---|---|---|---|---|---|
| Q01: Ketika mempelajari topik baru yang cukup kompleks, saya paling terbantu jika… | `VARK_Q01` | melihat diagram, bagan, peta konsep, atau ilustrasi hubungan antarbagian | mendengar penjelasan dan dapat berdiskusi atau bertanya secara lisan | membaca uraian tertulis, definisi, atau catatan yang terstruktur | mencoba contoh, simulasi, praktik, atau melihat penerapan nyata |
| Q02: Saat harus memahami prosedur penggunaan suatu perangkat atau aplikasi, saya cenderung… | `VARK_Q02` | mengikuti gambar langkah demi langkah atau tangkapan layar | meminta seseorang menjelaskan langkah-langkahnya secara lisan | membaca petunjuk tertulis dan daftar langkah | langsung mencoba sambil memperbaiki kesalahan yang muncul |
| Q03: Untuk mengingat materi menjelang ujian, saya biasanya lebih suka… | `VARK_Q03` | membuat peta konsep, skema, warna, atau diagram | menjelaskan materi dengan suara keras atau berdiskusi dengan teman | menulis ulang ringkasan, membuat daftar, atau membaca catatan | berlatih soal, mencoba contoh, atau menghubungkan materi dengan pengalaman |
| Q04: Ketika dosen menjelaskan konsep abstrak, saya lebih mudah memahami bila… | `VARK_Q04` | ada visualisasi yang menunjukkan hubungan antar konsep | penjelasan disampaikan dengan contoh verbal dan tanya jawab | tersedia teks atau rumusan tertulis yang dapat saya baca kembali | ada demonstrasi, studi kasus, atau aktivitas langsung |
| Q05: Saat menerima tugas baru, saya biasanya ingin terlebih dahulu… | `VARK_Q05` | melihat contoh hasil akhir atau struktur visual tugas | mendengarkan penjelasan dosen tentang apa yang harus dilakukan | membaca instruksi dan rubrik secara rinci | mencoba bagian awal tugas untuk memahami apa yang diminta |
| Q06: Jika saya tersesat di tempat yang belum dikenal, saya lebih memilih… | `VARK_Q06` | melihat peta atau petunjuk arah visual | bertanya kepada orang lain dan mendengarkan arahannya | membaca nama jalan, papan petunjuk, atau instruksi tertulis | berjalan mencoba rute sambil menyesuaikan arah |
| Q07: Saat mempelajari data atau hasil penelitian, saya paling terbantu oleh… | `VARK_Q07` | grafik, diagram, dan visualisasi pola | penjelasan lisan tentang makna hasil | tabel, uraian tertulis, dan penjelasan metodologis | contoh penerapan data pada kasus nyata |
| Q08: Jika harus menjelaskan ide kepada teman, saya cenderung… | `VARK_Q08` | menggambar skema atau menunjukkan visual | menjelaskannya melalui percakapan | menuliskan poin-poin atau memberi bacaan singkat | menunjukkan bagaimana ide tersebut bekerja melalui contoh atau demonstrasi |
| Q09: Dalam kelas daring, saya paling tertarik ketika materi menyediakan… | `VARK_Q09` | infografik, diagram, atau video dengan visual yang jelas | rekaman audio, diskusi sinkron, atau penjelasan verbal | modul teks, transkrip, dan catatan yang lengkap | simulasi, kuis interaktif, latihan, atau aktivitas berbasis kasus |
| Q10: Ketika menghadapi masalah yang belum pernah saya temui, saya biasanya… | `VARK_Q10` | membuat sketsa atau memetakan masalah secara visual | membicarakan masalah dengan orang lain untuk memperoleh ide | menuliskan fakta, alternatif, dan langkah pemecahan | mencoba beberapa solusi dan belajar dari hasil percobaan |
| Q11: Saat memilih sumber belajar tambahan, saya lebih tertarik pada… | `VARK_Q11` | video animasi, diagram interaktif, atau infografik | podcast, rekaman kuliah, atau diskusi kelompok | artikel, buku, modul, atau catatan tertulis | tutorial praktik, proyek, eksperimen, atau studi kasus |
| Q12: Ketika ingin memastikan saya benar-benar memahami suatu konsep, saya… | `VARK_Q12` | dapat menggambarkan hubungan konsep tersebut dalam bentuk visual | dapat menjelaskannya secara lisan dengan kata-kata saya sendiri | dapat menuliskan definisi dan penjelasannya secara runtut | dapat menerapkannya untuk menyelesaikan masalah atau tugas nyata |
| Q13: Jika harus belajar mandiri dari materi yang panjang, saya cenderung… | `VARK_Q13` | mencari bagian visual terlebih dahulu untuk memahami gambaran besar | mendengarkan penjelasan terkait sebelum atau sambil belajar | membaca dan membuat catatan secara sistematis | mencari latihan atau contoh penerapan setelah memahami pokok materi |
| Q14: Ketika menerima umpan balik atas tugas, saya lebih terbantu bila… | `VARK_Q14` | ditunjukkan bagian mana yang perlu diperbaiki secara visual atau dengan markup | dibicarakan secara langsung sehingga saya dapat bertanya | diberikan komentar tertulis yang rinci | diberi kesempatan mencoba revisi dan melihat dampaknya |
| Q15: Dalam kerja kelompok, saya biasanya berkontribusi paling baik ketika… | `VARK_Q15` | menyusun bagan, desain, atau struktur visual | berdiskusi, mempresentasikan, atau mengoordinasikan secara lisan | menulis dokumen, rangkuman, atau notulen | mengerjakan prototipe, praktik, atau implementasi |
| Q16: Jika harus memilih satu cara untuk mempelajari sesuatu yang benar-benar baru, saya akan memprioritaskan… | `VARK_Q16` | melihat representasi visual tentang bagaimana hal itu bekerja | mendengar seseorang menjelaskannya dan berdialog | membaca penjelasan tertulis yang lengkap | mengalami atau mencoba secara langsung |

### A.2 Usulan pengelompokan item Likert (berdasarkan urutan kolom dan isi pernyataan)

| Kolom Excel | Framework | Subskala (dugaan) | Prefix usulan | Jumlah item | Contoh item pertama |
|---|---|---|---|---|---|
| 17–24 | Honey-Mumford | Activist | `HM_ACT_01…08` | 8 | Saya menikmati aktivitas belajar baru meskipun belum sepenuhnya tahu bagaimana hasilnya. |
| 25–32 | Honey-Mumford | Reflector | `HM_REF_01…08` | 8 | Saya suka mengamati terlebih dahulu sebelum mengambil kesimpulan. |
| 33–40 | Honey-Mumford | Theorist | `HM_THE_01…08` | 8 | Saya merasa nyaman jika suatu materi memiliki konsep, model, atau kerangka yang logis. |
| 41–48 | Honey-Mumford | Pragmatist | `HM_PRA_01…08` | 8 | Saya tertarik pada ide yang dapat segera diterapkan untuk memecahkan masalah nyata. |
| 49–52 | Felder-Silverman | Active | `FS_ACT_01…04` | 4 | Saya memahami materi lebih cepat ketika dapat mendiskusikannya atau mengerjakan sesuatu secara aktif. |
| 53–56 | Felder-Silverman | Reflective | `FS_REF_01…04` | 4 | Saya lebih suka memikirkan materi secara mendalam sebelum membicarakan atau menerapkannya. |
| 57–60 | Felder-Silverman | Sensing | `FS_SEN_01…04` | 4 | Saya lebih menyukai fakta, data, contoh nyata, dan prosedur yang jelas. |
| 61–64 | Felder-Silverman | Intuitive | `FS_INT_01…04` | 4 | Saya tertarik pada kemungkinan, pola, konsep baru, atau pendekatan yang belum biasa. |
| 65–68 | Felder-Silverman | Visual | `FS_VIS_01…04` | 4 | Diagram, grafik, peta konsep, dan representasi visual membantu saya memahami materi. |
| 69–72 | Felder-Silverman | Verbal | `FS_VRB_01…04` | 4 | Penjelasan verbal atau teks tertulis membantu saya memahami materi dengan baik. |
| 73–76 | Felder-Silverman | Sequential | `FS_SEQ_01…04` | 4 | Saya lebih nyaman belajar ketika materi disusun langkah demi langkah secara berurutan. |
| 77–80 | Felder-Silverman | Global | `FS_GLO_01…04` | 4 | Kadang saya baru memahami suatu topik ketika tiba-tiba melihat gambaran besarnya. |
| 81–86 | Kolb | CE | `KOLB_CE_01…06` | 6 | Saya lebih mudah mulai belajar ketika memiliki pengalaman atau contoh konkret yang dapat saya amati. |
| 87–92 | Kolb | RO | `KOLB_RO_01…06` | 6 | Setelah mengikuti suatu kegiatan belajar, saya biasanya meluangkan waktu untuk memikirkan kembali apa yang terjadi. |
| 93–98 | Kolb | AC | `KOLB_AC_01…06` | 6 | Saya menyukai penjelasan yang memiliki kerangka konsep atau teori yang jelas. |
| 99–104 | Kolb | AE | `KOLB_AE_01…06` | 6 | Saya suka mencoba menerapkan ide baru untuk melihat apakah ide tersebut bekerja dalam praktik. |

*Kolom Excel dihitung dari 0 (kolom 0 = `ID`). Pengelompokan Likert didukung data: korelasi rata-rata antar-item di dalam kelompok selalu lebih tinggi daripada ke item di luar kelompok, untuk ke-16 kelompok.*
