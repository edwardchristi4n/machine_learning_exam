# Aturan Scoring Berbasis Teori (Versi 2)
**Proyek UTS Pembelajaran Mesin – Preferensi Gaya Belajar (Gasal 2026/2027)**

> **Dasar versi ini.** Arahan dosen (6 November 2026): *"Justru itulah yang harus mahasiswa selesaikan di projek UTS, bagaimana akan mengolah datanya agar siap untuk modelling. Harus sesuai teorinya di keempat model yang dipakai."*
> Jadi scoring **dan** label dibuat, dengan setiap aturan dan batas nilai diambil dari **teori/manual asli** masing-masing framework. Tidak ada angka batas yang dikarang sendiri. Versi ini **menggantikan** versi sebelumnya yang hanya membuat skor tanpa label.

---

## 0. Ringkasan satu halaman

| Framework | Teori asli | Yang kita lakukan | Label yang dihasilkan |
|---|---|---|---|
| **VARK** | Hitung pilihan V/A/R/K, lalu tentukan preferensi dengan **stepping distance** | Dipakai **apa adanya**, karena format survei sama dengan VARK asli (16 soal, boleh pilih >1) | Profil (`K`, `VA`, `VARK`, …) dan tipe (Single/Bimodal/Trimodal/Quadmodal) |
| **Felder-Silverman** | Skor dimensi = jumlah jawaban kutub a − kutub b; kekuatan 1–3 / 5–7 / 9–11 | Rumus yang sama: jumlah item kutub a − jumlah item kutub b. Ambang kekuatan diskalakan proporsional. | Kutub per dimensi (ACT/REF, SEN/INT, VIS/VRB, SEQ/GLO) + kekuatan |
| **Kolb** | AC−CE dan AE−RO, dipotong di **median kelompok norma** → 4 tipe | Rumus yang sama; median dihitung dari sampel kita (hasilnya 0 untuk kedua sumbu) | Diverging / Assimilating / Converging / Accommodating |
| **Honey-Mumford** | Skor per gaya, lalu dibandingkan dengan **norma persentil** (10/20/40/20/10%) | Metode persentil yang sama dengan norma dari sampel kita | Pita preferensi per gaya + gaya dominan |

**Hasil pada data kita:**

| Label | Distribusi |
|---|---|
| VARK tipe (n = 192) | Quadmodal 74 · Single 57 · Bimodal 31 · Trimodal 30 |
| VARK single (n = 57) | K 21 · V 16 · A 16 · R 4 |
| Kolb (n = 177) | Diverging 79 · Accommodating 42 · Assimilating 38 · Converging 18 |
| Honey-Mumford dominan (n = 177) | Reflector 51 · Activist 48 · Theorist 37 · Pragmatist 37 · seri 4 |
| Felder kutub (n = 177) | ACT 67 / REF 64 / netral 46 · SEN 108 / INT 33 / netral 36 · VIS 81 / VRB 45 / netral 51 · SEQ 86 / GLO 41 / netral 50 |

n = 177 berarti 15 straightliner tidak diberi label skala Likert (alasannya di Bagian 7).

---

## 1. Konsep dasar

### 1.1 Skor dan label
- **Skor** = angka ringkasan. Contoh: "selisih Active − Reflective = +8".
- **Label** = kategori yang ditentukan dari skor memakai **aturan batas dari teori**. Contoh: "ACT, moderate".

Label inilah yang nanti bisa menjadi **target (kelas)** di tahap modelling.

### 1.2 Kenapa perlu "adaptasi"?
Tiga dari empat instrumen asli punya format yang berbeda dengan survei kita:

| Framework | Format instrumen asli | Format survei kita | Perlu adaptasi? |
|---|---|---|---|
| VARK | 16 soal, 4 opsi, boleh pilih >1 | Sama persis | **Tidak** |
| Felder (ILS) | 44 soal pilih a/b (11 per dimensi) | 32 pernyataan skala 1–5 (4 per kutub) | Ya |
| Kolb (LSI) | 12 soal, 4 akhiran kalimat **diranking** 1–4 | 24 pernyataan skala 1–5 (6 per tahap) | Ya |
| Honey-Mumford (LSQ) | 80 pernyataan, centang jika setuju (20 per gaya) | 32 pernyataan skala 1–5 (8 per gaya) | Ya |

**Prinsip adaptasi:** pakai **rumus dan logika teori asli**. Kalau batas angkanya terikat pada format atau populasi instrumen asli, pakai **cara penurunan batas yang sama** seperti di manual aslinya (misalnya median atau persentil), dihitung dari sampel kita. Dengan begitu tidak ada batas yang dikarang sendiri.

---

## 2. Aturan umum

| Kode | Aturan |
|---|---|
| U1 | Semua scoring lewat kode di notebook. Data asli tidak diubah; hasil ditambahkan sebagai kolom baru. |
| U2 | 2 respons kosong total (ID 11, 133) dibuang sebelum scoring → **192 responden**. |
| U3 | 15 straightliner (88 jawaban Likert identik) tetap mendapat **skor**, tetapi **label skala Likert** (Felder, Kolb, Honey-Mumford) diisi kosong (NaN). Label VARK tetap dibuat. |
| U4 | Norma lokal (median Kolb, persentil Honey-Mumford) dihitung dari **177 responden non-straightliner**. |
| U5 | Setiap kolom skor dan label dicatat di data dictionary beserta rumus dan sumber teorinya. |

---

## 3. VARK (Fleming)

### 3.1 Teorinya
VARK mengelompokkan cara orang menerima informasi menjadi 4 modalitas: **V**isual (gambar), **A**uditori (mendengar/diskusi), **R**ead/Write (membaca/menulis), dan **K**inestetik (praktik). Menurut Fleming, banyak orang punya **lebih dari satu** preferensi (multimodal), jadi hasilnya bukan selalu satu huruf.

### 3.2 Aturan

**Langkah 1 – Skor.** Hitung berapa kali seseorang memilih opsi V, A, R, dan K di 16 soal (0–16 per modalitas). Lalu jumlahkan total semua pilihan (16–64).

**Langkah 2 – Stepping distance** (jarak langkah), ditentukan dari total skor:

| Total skor | Stepping distance |
|---|---|
| 16–21 | 1 |
| 22–27 | 2 |
| 28–32 | 3 |
| > 32 | 4 |

**Langkah 3 – Tentukan preferensi.**
1. Urutkan skor dari terbesar: n1 ≥ n2 ≥ n3 ≥ n4.
2. Modalitas tertinggi (n1) selalu menjadi preferensi.
3. Jika n1 − n2 **lebih besar** dari stepping distance, berhenti: hasilnya **single**. Jika tidak, n2 ikut menjadi preferensi.
4. Ulangi untuk n2 − n3, lalu n3 − n4.

**Intinya:** modalitas yang skornya "berdekatan" dengan skor tertinggi ikut dihitung sebagai preferensi. Semakin banyak opsi yang dipilih seseorang (total besar), semakin longgar jaraknya.

### 3.3 Contoh

**ID 4:** V = 4, A = 3, R = 4, K = 15 → total 26 → stepping distance **2**.
- Urutan: K 15, V 4, R 4, A 3.
- K − V = 15 − 4 = 11 > 2, jadi berhenti.
- Hasil: **`K`**, Single.

**ID 1:** V = 12, A = 12, R = 7, K = 0 → total 31 → stepping distance **3**.
- Urutan: V 12, A 12, R 7, K 0.
- V − A = 0 ≤ 3, jadi A ikut.
- A − R = 12 − 7 = 5 > 3, jadi berhenti.
- Hasil: **`VA`**, Bimodal.

### 3.4 Hasil pada data (n = 192)
- **Quadmodal (VARK): 74 (38,5%)** · Single: 57 (29,7%) · Bimodal: 31 (16,1%) · Trimodal: 30 (15,6%)
- Di antara yang single: K 21 · V 16 · A 16 · R 4
- Profil multimodal terbanyak: VAR 9, VAK 8, AR 8

**Catatan untuk EDA dan modelling:**
- Banyaknya quadmodal sesuai dengan teori Fleming bahwa multimodal itu umum. Penyebab lainnya: responden yang memilih banyak opsi mendapat stepping distance besar.
- Untuk target klasifikasi, `VARK_type` (4 kelas) cukup seimbang. `VARK_profile` punya 15 kombinasi dan banyak kelas kecil, jadi kurang cocok sebagai target langsung.
- R single hanya 4 orang, jadi ada *class imbalance*.

---

## 4. Felder-Silverman (Index of Learning Styles)

### 4.1 Teorinya
Felder-Silverman melihat gaya belajar sebagai **4 dimensi**. Setiap dimensi adalah **garis kontinum** dengan dua ujung (kutub):

| Dimensi | Kutub a | Kutub b |
|---|---|---|
| Pemrosesan | **ACT**ive – belajar dengan melakukan/diskusi | **REF**lective – memikirkan sendiri dulu |
| Persepsi | **SEN**sing – fakta, detail, konkret | **INT**uitive – konsep, pola, abstrak |
| Input | **VIS**ual – gambar, diagram | **V**e**RB**al – kata-kata, teks |
| Pemahaman | **SEQ**uential – langkah demi langkah | **GLO**bal – gambaran besar dulu |

Di ILS asli, skor dimensi = jumlah jawaban "a" − jumlah jawaban "b" (rentang −11 s.d. +11). Tandanya menunjukkan kutub, besarnya menunjukkan kekuatan:

| Skor ILS asli (|a − b|) | Arti |
|---|---|
| 1–3 | Balanced: seimbang, hanya kecenderungan ringan |
| 5–7 | Moderate: preferensi sedang |
| 9–11 | Strong: preferensi kuat |

### 4.2 Aturan (adaptasi)

**Skor dimensi** = jumlah jawaban 4 item kutub a − jumlah jawaban 4 item kutub b → rentang **−16 s.d. +16**.

Contoh: `FS_ACT_REF_score` = (jumlah 4 item Active) − (jumlah 4 item Reflective).

**Kutub (label utama):**
- Skor > 0 → kutub a (misalnya ACT)
- Skor < 0 → kutub b (misalnya REF)
- Skor = 0 → NEUTRAL

Ini sama dengan cara ILS melaporkan hasil, misalnya "3 ACT".

**Kekuatan (label pelengkap):** ambang ILS diskalakan proporsional dari maksimum 11 ke maksimum 16.

| ILS asli | Proporsi dari maksimum | Survei kita (|skor|) |
|---|---|---|
| Balanced (≤ 3 dari 11) | < 4/11 ≈ 36% | **0–5** |
| Moderate (5–7 dari 11) | 36%–73% | **6–11** |
| Strong (9–11 dari 11) | ≥ 8/11 ≈ 73% | **12–16** |

Keuntungan tambahan: dengan mengurangkan kutub a dan b, **bias "suka setuju" ikut hilang**, karena orang yang menjawab tinggi di semua pernyataan akan mendapat selisih mendekati 0.

### 4.3 Contoh: ID 4

| Dimensi | Kutub a (jumlah) | Kutub b (jumlah) | Skor | Kutub | Kekuatan |
|---|---|---|---|---|---|
| ACT_REF | Active 16 | Reflective 8 | +8 | ACT | Moderate |
| SEN_INT | Sensing 20 | Intuitive 8 | +12 | SEN | Strong |
| VIS_VRB | Visual 12 | Verbal 11 | +1 | VIS | Balanced |
| SEQ_GLO | Sequential 20 | Global 12 | +8 | SEQ | Moderate |

### 4.4 Hasil pada data (n = 177)

| Dimensi | Kutub a | NEUTRAL | Kutub b | Balanced / Moderate / Strong |
|---|---|---|---|---|
| ACT_REF | ACT 67 | 46 | REF 64 | 168 / 8 / 1 |
| SEN_INT | SEN 108 | 36 | INT 33 | 156 / 20 / 1 |
| VIS_VRB | VIS 81 | 51 | VRB 45 | 157 / 19 / 1 |
| SEQ_GLO | SEQ 86 | 50 | GLO 41 | 159 / 18 / 0 |

**Catatan penting:**
- **88–95% responden "Balanced"**, sehingga label kekuatan sangat tidak seimbang. Penyebabnya, skala 1–5 per pernyataan membuat selisih kutub cenderung kecil.
- Karena itu, untuk EDA dan modelling gunakan **kutub** (3 kelas, cukup seimbang) atau **skor numerik** (−16 s.d. +16). Kekuatan dilaporkan sebagai temuan, bukan sebagai target.
- Temuan menarik untuk EDA: mahasiswa cenderung **Sensing** (108 vs 33), yaitu lebih suka fakta dan contoh konkret daripada konsep abstrak.

---

## 5. Kolb (Learning Style Inventory)

### 5.1 Teorinya
Kolb memandang belajar sebagai **siklus 4 tahap**:

| Tahap | Singkatan | Artinya |
|---|---|---|
| Concrete Experience | CE | Mengalami langsung (*feeling*) |
| Reflective Observation | RO | Merenungkan/mengamati (*watching*) |
| Abstract Conceptualization | AC | Membangun teori (*thinking*) |
| Active Experimentation | AE | Mencoba menerapkan (*doing*) |

Tahap-tahap itu membentuk **2 sumbu**:
- **AC − CE** (cara menangkap informasi): positif = abstrak/berpikir, negatif = konkret/merasakan
- **AE − RO** (cara mengolah informasi): positif = aktif/melakukan, negatif = reflektif/mengamati

Gabungan kedua sumbu menghasilkan **4 tipe**:

| | **Reflektif** (AE − RO ≤ batas) | **Aktif** (AE − RO > batas) |
|---|---|---|
| **Konkret** (AC − CE ≤ batas) | **Diverging** – merasakan & mengamati | **Accommodating** – merasakan & melakukan |
| **Abstrak** (AC − CE > batas) | **Assimilating** – berpikir & mengamati | **Converging** – berpikir & melakukan |

**Batasnya dari mana?** Manual teknis LSI 3.1 (Kolb & Kolb, 2005) menetapkan batas di **persentil ke-50 (median) kelompok norma**, yaitu +7 untuk AC−CE dan +6 untuk AE−RO. Nilai yang **sama dengan** batas masuk ke sisi konkret/reflektif.

### 5.2 Aturan (adaptasi)
1. Skor tahap = jumlah 6 item per tahap (6–30): `KOLB_CE_sum`, `KOLB_RO_sum`, `KOLB_AC_sum`, `KOLB_AE_sum`.
2. `KOLB_ACCE` = AC − CE; `KOLB_AERO` = AE − RO (rentang −24 s.d. +24).
3. Batas = **median sampel** (n = 177). Ini mengikuti cara Kolb menurunkan batas. Angka +7/+6 tidak bisa dipakai karena berasal dari format ranking.
4. Aturan nilai sama dengan batas mengikuti manual: ≤ batas → konkret/reflektif.

**Hasilnya, median = 0 untuk kedua sumbu.** Batas empiris (cara Kolb) ternyata **sama persis** dengan titik seimbang teoritis (AC = CE, AE = RO). Ini argumen kuat bahwa batasnya tidak arbitrer.

### 5.3 Contoh: ID 4
- CE = 30, RO = 18, AC = 26, AE = 30
- AC − CE = 26 − 30 = **−4** → ≤ 0 → **konkret**
- AE − RO = 30 − 18 = **+12** → > 0 → **aktif**
- Hasil: **Accommodating** (merasakan & melakukan). Ini konsisten dengan VARK-nya yang `K`.

### 5.4 Hasil pada data (n = 177)
Diverging **79 (44,6%)** · Accommodating 42 (23,7%) · Assimilating 38 (21,5%) · Converging 18 (10,2%)

**Catatan:**
- Banyak responden yang nilainya **tepat 0** (AC−CE = 0 ada 42 orang; AE−RO = 0 ada 35 orang). Sesuai manual, mereka masuk sisi konkret/reflektif, sehingga **Diverging membesar**. Sebutkan ini di laporan sebagai keterbatasan.
- Converging hanya 10%, jadi ada class imbalance. Catat untuk tahap modelling.
- Versi 9 tipe Kolb (dipotong per tertil) **tidak dipakai** karena 177 responden ÷ 9 kelas terlalu kecil per kelas.

---

## 6. Honey-Mumford (Learning Styles Questionnaire)

### 6.1 Teorinya
Honey & Mumford mengadaptasi siklus Kolb menjadi **4 gaya** dengan bahasa sehari-hari:

| Gaya | Ciri | Padanan tahap Kolb |
|---|---|---|
| **Activist** | Langsung terjun, suka pengalaman baru | CE / mengalami |
| **Reflector** | Mengamati, menimbang dari berbagai sisi | RO / merenungkan |
| **Theorist** | Logis, suka model dan teori | AC / menyimpulkan |
| **Pragmatist** | Praktis, mencoba apakah ide bisa diterapkan | AE / merencanakan langkah |

Di LSQ asli, responden mencentang pernyataan yang disetujui dan skor tiap gaya 0–20. Skor itu **tidak dibaca mentah**, melainkan dibandingkan dengan **norma** yang dibagi berdasarkan persentil:

| Pita | Posisi dalam kelompok norma |
|---|---|
| Very strong | 10% teratas |
| Strong | 20% berikutnya |
| Moderate | 40% tengah |
| Low | 20% berikutnya |
| Very low | 10% terbawah |

Alasannya, tiap gaya punya "rata-rata normal" yang berbeda. Skor Reflector 15 bisa termasuk "strong", sementara Activist 15 sudah "very strong". Jadi gaya yang menonjol dilihat **relatif terhadap norma**, bukan dari skor mentah tertinggi.

### 6.2 Kenapa norma asli tidak dipakai
Norma LSQ diturunkan dari sekitar 3.500 responden (kebanyakan manajer di Inggris), dengan format centang 20 item. Kalau survei kita dikonversi paksa ke norma itu (setuju = 4–5, dihitung, lalu dikali 2,5 ke skala 0–20), hasilnya **57,6% responden "Very strong Activist"**, padahal menurut definisi seharusnya hanya 10%. Norma asli jelas tidak cocok untuk format dan populasi kita.

### 6.3 Aturan (adaptasi)
1. Skor gaya = jumlah 8 item (8–40): `HM_ACT_sum`, `HM_REF_sum`, `HM_THE_sum`, `HM_PRA_sum`.
2. Norma lokal: hitung **peringkat persentil** skor tiap gaya di antara 177 responden (`HM_xxx_pct`), lalu potong dengan definisi pita LSQ (10/20/40/20/10%). Hasilnya `HM_xxx_band`.
3. **Gaya dominan** (`HM_dominant`) = gaya dengan peringkat persentil **tertinggi**, yaitu yang paling menonjol relatif terhadap norma, persis prinsip LSQ. Jika seri, kedua gaya ditulis (misalnya `ACT+REF`).

Rentang skor jumlah per pita pada data kita, sebagai gambaran:

| Gaya | Very low | Low | Moderate | Strong | Very strong |
|---|---|---|---|---|---|
| Activist | 18–24 | 25–28 | 29–33 | 34–37 | 38–40 |
| Reflector | 14–25 | 26–29 | 30–33 | 34–36 | 37–40 |
| Theorist | 14–25 | 26–29 | 30–34 | 35–37 | 38–40 |
| Pragmatist | 19–25 | 26–29 | 30–34 | 35–38 | 39–40 |

### 6.4 Contoh: ID 4

| Gaya | Skor (8–40) | Persentil | Pita |
|---|---|---|---|
| Activist | 40 | 97,5% | Very strong |
| Reflector | 27 | 16,1% | Low |
| Theorist | 36 | 80,5% | Strong |
| Pragmatist | 40 | 97,2% | Very strong |

Activist dan Pragmatist sama-sama skor 40, tetapi persentil Activist sedikit lebih tinggi karena lebih sedikit orang yang mencapai skor 40 di Activist. Dominan: **Activist**. Ini konsisten dengan Kolb Accommodating dan VARK K.

### 6.5 Hasil pada data (n = 177)
- **Dominan:** Reflector 51 · Activist 48 · Theorist 37 · Pragmatist 37 · seri 4. **Cukup seimbang** untuk target modelling.
- Pita per gaya mendekati proporsi 10/20/40/20/10 sesuai definisi (karena ada nilai seri, proporsinya tidak persis).
- Perbandingan: kalau dominan diambil dari rata-rata mentah tertinggi, ada **49 responden seri** dan hasilnya condong ke Pragmatist. Ini bukti mengapa teori LSQ membaca skor relatif terhadap norma.

---

## 7. Straightliner dan kasus khusus

| Kasus | Dampak pada scoring | Penanganan |
|---|---|---|
| 15 straightliner (semua 88 jawaban Likert sama) | Felder: semua skor 0 → NEUTRAL. Kolb: AC−CE = AE−RO = 0 → semuanya **Diverging**. Honey-Mumford: keempat gaya seri. Labelnya artefak, bukan preferensi. | Skor tetap dihitung, **label Likert = NaN**, `flag_straightline = True`. Label VARK tetap dibuat karena jawaban VARK mereka bervariasi. |
| Memilih semua opsi VARK (ID 55) | V = A = R = K = 16 → `VARK` (quadmodal) | Sesuai algoritma; tidak perlu aturan khusus |
| 2 respons kosong total | Tidak dapat diskor | Dibuang sebelum scoring (cleaning log) |

Alasan untuk laporan: *"Responden yang memberikan jawaban identik pada seluruh 88 pernyataan, termasuk pernyataan dengan kutub berlawanan, tidak memberikan informasi untuk membedakan preferensi. Label hasil teori (misalnya tipe Kolb) akan menjadi artefak aturan batas, bukan cerminan preferensi. Karena itu skor tetap dihitung, tetapi label skala Likert tidak ditetapkan."*

---

## 8. Kolom output untuk data dictionary & kesiapan modelling

| Kolom | Isi | Tipe | Rentang/kategori | Sumber aturan |
|---|---|---|---|---|
| `VARK_V/A/R/K` | Jumlah pilihan per modalitas | int | 0–16 | Fleming (hitungan VARK) |
| `VARK_total` | Total pilihan | int | 16–64 | Fleming |
| `VARK_profile` | Modalitas yang menjadi preferensi | kategori | `V` … `VARK` (15 kombinasi) | Fleming (stepping distance) |
| `VARK_n_modal`, `VARK_type` | Jumlah preferensi | int / kategori | 1–4 / Single–Quadmodal | Fleming |
| `FS_<DIM>_score` | Jumlah kutub a − kutub b | int | −16 … +16 | Felder & Soloman (ILS: a − b) |
| `FS_<DIM>_pole` | Arah preferensi | kategori | kutub a / kutub b / NEUTRAL | ILS |
| `FS_<DIM>_strength` | Kekuatan | kategori ordinal | Balanced / Moderate / Strong | ILS 1–3/5–7/9–11, diskalakan proporsional |
| `KOLB_<MODE>_sum` | Jumlah 6 item per tahap | int | 6–30 | Kolb (skor tahap) |
| `KOLB_ACCE`, `KOLB_AERO` | Skor sumbu | int | −24 … +24 | Kolb LSI |
| `KOLB_style` | Tipe Kolb | kategori | 4 tipe | Kolb LSI 3.1: batas = median norma (di sini 0) |
| `HM_<GAYA>_sum` | Jumlah 8 item per gaya | int | 8–40 | Honey & Mumford (skor gaya) |
| `HM_<GAYA>_pct` | Peringkat persentil dalam sampel | float | 0–1 | Norma LSQ (metode persentil) |
| `HM_<GAYA>_band` | Pita preferensi | kategori ordinal | Very low … Very strong | LSQ 10/20/40/20/10% |
| `HM_dominant` | Gaya paling menonjol relatif norma | kategori | ACT/REF/THE/PRA (+seri) | LSQ |
| `flag_straightline` | Jawaban 88 item identik | bool | True/False | Pemeriksaan kualitas |

**Kandidat target modelling** (untuk proyek berikutnya):

| Target | Jumlah kelas | Keseimbangan | Catatan |
|---|---|---|---|
| `HM_dominant` | 4 | Baik (37–51 per kelas) | Pilihan paling seimbang |
| `KOLB_style` | 4 | Sedang (Converging 10%) | Perlu perhatian class imbalance |
| `VARK_type` | 4 | Sedang (30–74) | |
| `FS_<DIM>_pole` | 3 | Sedang; SEN_INT condong ke SEN | |
| `FS_<DIM>_strength` | 3 | **Buruk** (≈ 90% Balanced) | Tidak disarankan sebagai target |

## 8b. Label versi panduan UAS (sel S5 di notebook)

Panduan UAS memakai **aturan label operasional** yang lebih sederhana dari aturan teori di atas. Supaya dataset UTS langsung bisa dipakai di UAS, label versi UAS dihitung juga, **di kolom terpisah**. Label teori (bagian 3–6) tidak diubah.

| Kolom | Aturan UAS | Bedanya dengan label teori |
|---|---|---|
| `VARK_Qxx_V/A/R/K` (64 kolom) | 1 = opsi dipilih, 0 = tidak; soal kosong tetap kosong | Format fitur X untuk VARK. Jumlahnya per modalitas = `VARK_V/A/R/K` |
| `VARK_label_uas` | Modalitas dengan skor tertinggi yang unik; kalau seri → **Seri** | Teori memakai stepping distance (bisa multimodal) |
| `FS_AR_label_uas` | Tanda `FS_ACT_REF_score`: + Active, − Reflective, 0 **Setara** | Hanya sumbu Active–Reflective, tanpa kekuatan |
| `HM_label_uas` | Gaya dengan jumlah tertinggi yang unik; kalau seri → **Seri** | Teori memakai persentil (norma lokal) |
| `KOLB_label_uas` | Tanda d1 = AC−CE dan d2 = AE−RO; salah satu 0 → **Batas** | Teori memasukkan nilai 0 ke sisi konkret/reflektif |

Straightliner **tetap diberi label** karena aturan UAS bisa menghitungnya (semuanya menjadi Setara/Seri/Batas). Keputusan memakai atau mengecualikan mereka diambil saat modelling di UAS.

**Hasil pada data (n = 192):**

| Label UAS | Distribusi | Kategori Seri/Setara/Batas |
|---|---|---|
| `VARK_label_uas` | Kinesthetic 51, Aural 42, Visual 38, Read/Write 31, Seri 30 | 16% |
| `FS_AR_label_uas` | Active 67, Reflective 64, Setara 61 | 32% |
| `HM_label_uas` | Seri 64, Pragmatist 43, Reflector 36, Theorist 31, Activist 18 | 33% (kelas terbesar) |
| `KOLB_label_uas` | Batas 70, CE_RO 42, CE_AE 32, AC_RO 30, AC_AE 18 | 36% (kelas terbesar) |

Artinya, untuk klasifikasi UAS, VARK dan Felder lebih siap; HM dan Kolb didominasi kategori operasional (grafik E5c).

---

## 9. Paragraf siap pakai untuk laporan

> *Scoring dilakukan mengikuti teori dan manual asli setiap framework. VARK memakai algoritma Fleming (hitungan pilihan per modalitas dan stepping distance berdasarkan total skor), karena format survei sama dengan kuesioner VARK asli. Untuk Felder-Silverman, Kolb, dan Honey-Mumford, survei memakai skala Likert 1–5, berbeda dari format asli (pilihan a/b, ranking, dan centang). Karena itu rumus teori dipertahankan, yaitu selisih kutub ILS, sumbu AC−CE dan AE−RO Kolb, serta skor per gaya LSQ. Batas kategori diturunkan dengan metode yang sama seperti manual aslinya: ambang ILS diskalakan proporsional, batas Kolb pada median sampel sesuai cara LSI 3.1 menurunkan cut-point dari persentil ke-50 kelompok norma, dan pita preferensi Honey-Mumford memakai pembagian persentil 10/20/40/20/10% dengan norma lokal. Norma Honey-Mumford asli tidak digunakan karena menghasilkan 57,6% responden "very strong Activist", tidak sesuai definisi 10% teratas.*

**Keterbatasan yang perlu ditulis:**
- Instrumen berupa adaptasi Likert. Hasil tidak dapat dibandingkan langsung dengan norma internasional ILS, LSI, atau LSQ.
- Norma lokal bergantung pada sampel (n = 177, didominasi angkatan 2024). Label bersifat **relatif terhadap sampel ini**.
- Banyak nilai seri di batas Kolb (0) sehingga Diverging membesar.
- Kekuatan Felder hampir seluruhnya Balanced, sehingga daya bedanya rendah.
- Pengelompokan item ke subskala disusun dari urutan kolom dan isi pernyataan, dan didukung Cronbach's alpha sekitar 0,70 ke atas.

---

## 10. Sumber
- Fleming, N. D. – VARK Questionnaire v7.1 (adaptasi dari vark-learn.com): [University of Utah Learning Center](https://learningcenter.utah.edu/_resources/documents/vark-learning-styles-inventory.pdf)
- Algoritma stepping distance VARK: Khongpit, V., Sintanakul, K., & Nomphonkrang, T. (2018). *The VARK Learning Style of the University Student in Computer Course.* International Journal of Learning and Teaching, 4(2), 102–106. [PDF](https://www.ijlt.org/uploadfile/2018/0528/20180528100104475.pdf)
- Felder, R. M., & Soloman, B. A. – *Learning Styles and Strategies* (interpretasi skor ILS 1–3/5–7/9–11): [PDF](https://www.andrews.edu/services/ctcenter/career-center/learning-styles-strategies/learning-styles-and-strategies.pdf)
- Felder, R. M., & Spurlin, J. (2005). *Applications, Reliability and Validity of the Index of Learning Styles.* International Journal of Engineering Education, 21(1), 103–112. [PDF](https://www.ijee.ie/articles/Vol21-1/IJEE1553.pdf)
- Kolb, A. Y., & Kolb, D. A. (2005). *The Kolb Learning Style Inventory – Version 3.1: 2005 Technical Specifications.* [PDF](https://shrm.org/content/dam/en/shrm/topics-tools/news/organizational-employee-development/lsitechmanual.pdf)
- Honey, P., & Mumford, A. – Learning Styles Questionnaire (80 item, skoring & norma): [University of Manchester](https://sites.manchester.ac.uk/gmba/wp-content/uploads/sites/116/2019/08/Learning-Styles-Questionaire.pdf), [DCM Learning](https://dcmlearning.ie/as/userfiles/People%20Management%20Resources/DCM%20Learning%20People%20Management%20-%20Learning%20Styles%20Questionnaire.pdf)

---

## Lampiran 1 – Pemetaan opsi VARK

Setiap pertanyaan punya tepat 4 opsi, satu untuk tiap modalitas.

| Pertanyaan | V (Visual) | A (Auditori) | R (Read/Write) | K (Kinestetik) |
|---|---|---|---|---|
| **Q01** Ketika mempelajari topik baru yang cukup kompleks, saya paling terbantu jika… | melihat diagram, bagan, peta konsep, atau ilustrasi hubungan antarbagian | mendengar penjelasan dan dapat berdiskusi atau bertanya secara lisan | membaca uraian tertulis, definisi, atau catatan yang terstruktur | mencoba contoh, simulasi, praktik, atau melihat penerapan nyata |
| **Q02** Saat harus memahami prosedur penggunaan suatu perangkat atau aplikasi, saya cenderung… | mengikuti gambar langkah demi langkah atau tangkapan layar | meminta seseorang menjelaskan langkah-langkahnya secara lisan | membaca petunjuk tertulis dan daftar langkah | langsung mencoba sambil memperbaiki kesalahan yang muncul |
| **Q03** Untuk mengingat materi menjelang ujian, saya biasanya lebih suka… | membuat peta konsep, skema, warna, atau diagram | menjelaskan materi dengan suara keras atau berdiskusi dengan teman | menulis ulang ringkasan, membuat daftar, atau membaca catatan | berlatih soal, mencoba contoh, atau menghubungkan materi dengan pengalaman |
| **Q04** Ketika dosen menjelaskan konsep abstrak, saya lebih mudah memahami bila… | ada visualisasi yang menunjukkan hubungan antar konsep | penjelasan disampaikan dengan contoh verbal dan tanya jawab | tersedia teks atau rumusan tertulis yang dapat saya baca kembali | ada demonstrasi, studi kasus, atau aktivitas langsung |
| **Q05** Saat menerima tugas baru, saya biasanya ingin terlebih dahulu… | melihat contoh hasil akhir atau struktur visual tugas | mendengarkan penjelasan dosen tentang apa yang harus dilakukan | membaca instruksi dan rubrik secara rinci | mencoba bagian awal tugas untuk memahami apa yang diminta |
| **Q06** Jika saya tersesat di tempat yang belum dikenal, saya lebih memilih… | melihat peta atau petunjuk arah visual | bertanya kepada orang lain dan mendengarkan arahannya | membaca nama jalan, papan petunjuk, atau instruksi tertulis | berjalan mencoba rute sambil menyesuaikan arah |
| **Q07** Saat mempelajari data atau hasil penelitian, saya paling terbantu oleh… | grafik, diagram, dan visualisasi pola | penjelasan lisan tentang makna hasil | tabel, uraian tertulis, dan penjelasan metodologis | contoh penerapan data pada kasus nyata |
| **Q08** Jika harus menjelaskan ide kepada teman, saya cenderung… | menggambar skema atau menunjukkan visual | menjelaskannya melalui percakapan | menuliskan poin-poin atau memberi bacaan singkat | menunjukkan bagaimana ide tersebut bekerja melalui contoh atau demonstrasi |
| **Q09** Dalam kelas daring, saya paling tertarik ketika materi menyediakan… | infografik, diagram, atau video dengan visual yang jelas | rekaman audio, diskusi sinkron, atau penjelasan verbal | modul teks, transkrip, dan catatan yang lengkap | simulasi, kuis interaktif, latihan, atau aktivitas berbasis kasus |
| **Q10** Ketika menghadapi masalah yang belum pernah saya temui, saya biasanya… | membuat sketsa atau memetakan masalah secara visual | membicarakan masalah dengan orang lain untuk memperoleh ide | menuliskan fakta, alternatif, dan langkah pemecahan | mencoba beberapa solusi dan belajar dari hasil percobaan |
| **Q11** Saat memilih sumber belajar tambahan, saya lebih tertarik pada… | video animasi, diagram interaktif, atau infografik | podcast, rekaman kuliah, atau diskusi kelompok | artikel, buku, modul, atau catatan tertulis | tutorial praktik, proyek, eksperimen, atau studi kasus |
| **Q12** Ketika ingin memastikan saya benar-benar memahami suatu konsep, saya… | dapat menggambarkan hubungan konsep tersebut dalam bentuk visual | dapat menjelaskannya secara lisan dengan kata-kata saya sendiri | dapat menuliskan definisi dan penjelasannya secara runtut | dapat menerapkannya untuk menyelesaikan masalah atau tugas nyata |
| **Q13** Jika harus belajar mandiri dari materi yang panjang, saya cenderung… | mencari bagian visual terlebih dahulu untuk memahami gambaran besar | mendengarkan penjelasan terkait sebelum atau sambil belajar | membaca dan membuat catatan secara sistematis | mencari latihan atau contoh penerapan setelah memahami pokok materi |
| **Q14** Ketika menerima umpan balik atas tugas, saya lebih terbantu bila… | ditunjukkan bagian mana yang perlu diperbaiki secara visual atau dengan markup | dibicarakan secara langsung sehingga saya dapat bertanya | diberikan komentar tertulis yang rinci | diberi kesempatan mencoba revisi dan melihat dampaknya |
| **Q15** Dalam kerja kelompok, saya biasanya berkontribusi paling baik ketika… | menyusun bagan, desain, atau struktur visual | berdiskusi, mempresentasikan, atau mengoordinasikan secara lisan | menulis dokumen, rangkuman, atau notulen | mengerjakan prototipe, praktik, atau implementasi |
| **Q16** Jika harus memilih satu cara untuk mempelajari sesuatu yang benar-benar baru, saya akan memprioritaskan… | melihat representasi visual tentang bagaimana hal itu bekerja | mendengar seseorang menjelaskannya dan berdialog | membaca penjelasan tertulis yang lengkap | mengalami atau mencoba secara langsung |

## Lampiran 2 – Daftar pernyataan per tipe (skala 1–5)

Nomor kolom dihitung dari kolom pertama Excel = 0 (`ID`).

### Honey-Mumford – Activist (langsung terjun, suka hal baru)  →  skor `HM_ACT`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `HM_ACT_01` | 17 | Saya menikmati aktivitas belajar baru meskipun belum sepenuhnya tahu bagaimana hasilnya. |
| `HM_ACT_02` | 18 | Saya lebih bersemangat ketika dapat langsung terlibat dalam tugas atau tantangan baru. |
| `HM_ACT_03` | 19 | Saya suka mencoba berbagai cara tanpa terlalu lama menunggu sebelum memulai. |
| `HM_ACT_04` | 20 | Diskusi, simulasi, atau aktivitas kelompok membuat saya lebih aktif belajar. |
| `HM_ACT_05` | 21 | Saya tertarik pada pengalaman yang menuntut spontanitas dan keterlibatan langsung. |
| `HM_ACT_06` | 22 | Saya cenderung menerima tantangan baru sebagai kesempatan untuk belajar. |
| `HM_ACT_07` | 23 | Saya merasa belajar paling hidup ketika banyak hal terjadi dan saya dapat ikut berpartisipasi. |
| `HM_ACT_08` | 24 | Saya mudah tertarik mencoba metode atau alat baru dalam pembelajaran. |

### Honey-Mumford – Reflector (mengamati & menimbang dulu)  →  skor `HM_REF`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `HM_REF_01` | 25 | Saya suka mengamati terlebih dahulu sebelum mengambil kesimpulan. |
| `HM_REF_02` | 26 | Saya mempertimbangkan pengalaman dari beberapa sudut pandang sebelum bertindak. |
| `HM_REF_03` | 27 | Saya sering meninjau kembali apa yang terjadi untuk memahami pelajaran yang dapat diambil. |
| `HM_REF_04` | 28 | Saya lebih nyaman jika memiliki cukup informasi sebelum membuat keputusan. |
| `HM_REF_05` | 29 | Saya suka mendengarkan pandangan orang lain sebelum menyampaikan kesimpulan saya. |
| `HM_REF_06` | 30 | Saya sering membuat catatan atau refleksi setelah menyelesaikan aktivitas belajar. |
| `HM_REF_07` | 31 | Saya cenderung berhati-hati dan tidak terburu-buru ketika menilai suatu situasi. |
| `HM_REF_08` | 32 | Saya menikmati waktu untuk berpikir dan menghubungkan berbagai pengalaman. |

### Honey-Mumford – Theorist (logika & teori)  →  skor `HM_THE`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `HM_THE_01` | 33 | Saya merasa nyaman jika suatu materi memiliki konsep, model, atau kerangka yang logis. |
| `HM_THE_02` | 34 | Saya mencoba memahami prinsip yang menjelaskan mengapa sesuatu terjadi. |
| `HM_THE_03` | 35 | Saya menyukai argumen yang konsisten dan didukung alasan yang jelas. |
| `HM_THE_04` | 36 | Saya tertarik menyusun berbagai informasi menjadi kerangka pemahaman yang sistematis. |
| `HM_THE_05` | 37 | Saya cenderung mempertanyakan asumsi yang tidak memiliki dasar logis. |
| `HM_THE_06` | 38 | Saya suka membandingkan suatu pengalaman dengan teori atau konsep yang relevan. |
| `HM_THE_07` | 39 | Saya merasa lebih yakin ketika sebuah keputusan dapat dijelaskan secara rasional. |
| `HM_THE_08` | 40 | Saya senang menganalisis hubungan sebab-akibat dalam materi yang dipelajari. |

### Honey-Mumford – Pragmatist (praktis, bisa langsung dipakai)  →  skor `HM_PRA`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `HM_PRA_01` | 41 | Saya tertarik pada ide yang dapat segera diterapkan untuk memecahkan masalah nyata. |
| `HM_PRA_02` | 42 | Saya suka menguji apakah konsep yang dipelajari benar-benar berguna dalam praktik. |
| `HM_PRA_03` | 43 | Saya cenderung mencari cara yang lebih efektif atau efisien untuk mengerjakan sesuatu. |
| `HM_PRA_04` | 44 | Saya merasa puas ketika hasil belajar dapat diterjemahkan menjadi tindakan konkret. |
| `HM_PRA_05` | 45 | Saya sering bertanya bagaimana sebuah teori dapat digunakan pada situasi nyata. |
| `HM_PRA_06` | 46 | Saya suka bereksperimen dengan teknik yang menjanjikan manfaat praktis. |
| `HM_PRA_07` | 47 | Saya lebih tertarik pada solusi yang dapat diterapkan dibanding diskusi yang terlalu abstrak. |
| `HM_PRA_08` | 48 | Saya cenderung menilai suatu ide dari kegunaannya dalam membantu menyelesaikan tugas. |

### Felder – Active (belajar dengan melakukan/diskusi)  →  skor `FS_ACT`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `FS_ACT_01` | 49 | Saya memahami materi lebih cepat ketika dapat mendiskusikannya atau mengerjakan sesuatu secara aktif. |
| `FS_ACT_02` | 50 | Belajar bersama atau memecahkan masalah secara kolaboratif membantu saya memahami konsep. |
| `FS_ACT_03` | 51 | Saya suka mencoba segera setelah mendapatkan penjelasan baru. |
| `FS_ACT_04` | 52 | Saya sering memperoleh pemahaman ketika menjelaskan ide kepada orang lain. |

### Felder – Reflective (memikirkan sendiri dulu)  →  skor `FS_REF`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `FS_REF_01` | 53 | Saya lebih suka memikirkan materi secara mendalam sebelum membicarakan atau menerapkannya. |
| `FS_REF_02` | 54 | Saya membutuhkan waktu sendiri untuk memahami konsep baru. |
| `FS_REF_03` | 55 | Saya cenderung mengamati dan menganalisis sebelum ikut mencoba. |
| `FS_REF_04` | 56 | Saya lebih nyaman menyusun pemahaman secara internal sebelum berdiskusi. |

### Felder – Sensing (fakta, detail, konkret)  →  skor `FS_SEN`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `FS_SEN_01` | 57 | Saya lebih menyukai fakta, data, contoh nyata, dan prosedur yang jelas. |
| `FS_SEN_02` | 58 | Saya merasa terbantu ketika tugas memiliki langkah dan kriteria yang konkret. |
| `FS_SEN_03` | 59 | Saya cenderung memperhatikan detail dan kondisi praktis dari suatu masalah. |
| `FS_SEN_04` | 60 | Saya menyukai materi yang menunjukkan hubungan langsung dengan aplikasi nyata. |

### Felder – Intuitive (konsep, pola, abstrak)  →  skor `FS_INT`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `FS_INT_01` | 61 | Saya tertarik pada kemungkinan, pola, konsep baru, atau pendekatan yang belum biasa. |
| `FS_INT_02` | 62 | Saya menikmati pertanyaan yang menuntut interpretasi dan pemikiran abstrak. |
| `FS_INT_03` | 63 | Saya senang menemukan hubungan konseptual yang tidak langsung terlihat. |
| `FS_INT_04` | 64 | Saya cukup nyaman dengan gagasan teoritis meskipun belum ada contoh praktisnya. |

### Felder – Visual (gambar, diagram)  →  skor `FS_VIS`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `FS_VIS_01` | 65 | Diagram, grafik, peta konsep, dan representasi visual membantu saya memahami materi. |
| `FS_VIS_02` | 66 | Saya lebih cepat mengingat informasi ketika ditampilkan secara visual. |
| `FS_VIS_03` | 67 | Saya suka melihat hubungan antarkonsep melalui gambar atau skema. |
| `FS_VIS_04` | 68 | Saya merasa visualisasi membantu mengurangi kerumitan suatu topik. |

### Felder – Verbal (kata-kata, teks, penjelasan)  →  skor `FS_VRB`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `FS_VRB_01` | 69 | Penjelasan verbal atau teks tertulis membantu saya memahami materi dengan baik. |
| `FS_VRB_02` | 70 | Saya suka membaca uraian atau mendengarkan penjelasan rinci tentang suatu konsep. |
| `FS_VRB_03` | 71 | Saya dapat membangun pemahaman dari penjelasan kata-kata tanpa selalu membutuhkan gambar. |
| `FS_VRB_04` | 72 | Saya sering mengandalkan catatan tertulis atau penjelasan lisan ketika belajar. |

### Felder – Sequential (langkah demi langkah)  →  skor `FS_SEQ`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `FS_SEQ_01` | 73 | Saya lebih nyaman belajar ketika materi disusun langkah demi langkah secara berurutan. |
| `FS_SEQ_02` | 74 | Saya biasanya memahami bagian-bagian kecil terlebih dahulu sebelum melihat keseluruhan. |
| `FS_SEQ_03` | 75 | Saya merasa progres belajar lebih mudah diikuti bila urutannya jelas. |
| `FS_SEQ_04` | 76 | Saya suka menyelesaikan satu tahap sebelum beralih ke tahap berikutnya. |

### Felder – Global (gambaran besar dulu)  →  skor `FS_GLO`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `FS_GLO_01` | 77 | Kadang saya baru memahami suatu topik ketika tiba-tiba melihat gambaran besarnya. |
| `FS_GLO_02` | 78 | Saya dapat melompat antarbagian materi dan kemudian menghubungkannya menjadi satu pemahaman. |
| `FS_GLO_03` | 79 | Saya sering memerlukan gambaran keseluruhan sebelum detail-detailnya terasa bermakna. |
| `FS_GLO_04` | 80 | Saya nyaman memahami materi melalui hubungan luas antar topik, meskipun urutannya tidak selalu linear. |

### Kolb – CE, Concrete Experience (mengalami langsung)  →  skor `KOLB_CE`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `KOLB_CE_01` | 81 | Saya lebih mudah mulai belajar ketika memiliki pengalaman atau contoh konkret yang dapat saya amati. |
| `KOLB_CE_02` | 82 | Saya menikmati kegiatan belajar yang membuat saya terlibat langsung dalam situasi nyata atau simulasi. |
| `KOLB_CE_03` | 83 | Saya memahami materi lebih baik ketika konsep dikaitkan dengan pengalaman yang pernah saya alami. |
| `KOLB_CE_04` | 84 | Studi kasus nyata membantu saya menangkap makna suatu konsep. |
| `KOLB_CE_05` | 85 | Saya cenderung memperhatikan detail dari pengalaman sebelum menarik kesimpulan. |
| `KOLB_CE_06` | 86 | Saya merasa pembelajaran menjadi bermakna ketika ada sesuatu yang dapat dialami secara langsung. |

### Kolb – RO, Reflective Observation (merenungkan)  →  skor `KOLB_RO`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `KOLB_RO_01` | 87 | Setelah mengikuti suatu kegiatan belajar, saya biasanya meluangkan waktu untuk memikirkan kembali apa yang terjadi. |
| `KOLB_RO_02` | 88 | Saya suka meninjau suatu pengalaman dari beberapa sudut pandang sebelum membuat keputusan. |
| `KOLB_RO_03` | 89 | Saya sering mempertimbangkan mengapa suatu hasil terjadi, bukan hanya apa hasilnya. |
| `KOLB_RO_04` | 90 | Saya merasa terbantu ketika diberi waktu untuk mengamati dan merefleksikan sebelum bertindak. |
| `KOLB_RO_05` | 91 | Saya suka membandingkan pengalaman baru dengan pengalaman sebelumnya untuk menemukan pola. |
| `KOLB_RO_06` | 92 | Saya cenderung mencatat hal-hal yang saya pelajari setelah menyelesaikan suatu aktivitas. |

### Kolb – AC, Abstract Conceptualization (menarik teori)  →  skor `KOLB_AC`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `KOLB_AC_01` | 93 | Saya menyukai penjelasan yang memiliki kerangka konsep atau teori yang jelas. |
| `KOLB_AC_02` | 94 | Saya berusaha memahami prinsip umum di balik contoh-contoh yang diberikan. |
| `KOLB_AC_03` | 95 | Saya nyaman menggunakan model, konsep, atau teori untuk menjelaskan suatu masalah. |
| `KOLB_AC_04` | 96 | Saya sering mencoba merumuskan hubungan logis antar gagasan yang sedang saya pelajari. |
| `KOLB_AC_05` | 97 | Saya lebih yakin memahami materi ketika dapat menjelaskannya dengan konsep yang terstruktur. |
| `KOLB_AC_06` | 98 | Saya senang mengembangkan generalisasi atau kesimpulan dari beberapa contoh yang berbeda. |

### Kolb – AE, Active Experimentation (mencoba menerapkan)  →  skor `KOLB_AE`

| Kode item | Kolom | Pernyataan |
|---|---|---|
| `KOLB_AE_01` | 99 | Saya suka mencoba menerapkan ide baru untuk melihat apakah ide tersebut bekerja dalam praktik. |
| `KOLB_AE_02` | 100 | Saya cepat beralih dari memahami konsep ke mencoba menggunakannya dalam tugas atau proyek. |
| `KOLB_AE_03` | 101 | Saya tertarik menguji beberapa alternatif solusi terhadap suatu masalah. |
| `KOLB_AE_04` | 102 | Saya belajar banyak dari eksperimen, percobaan, dan perbaikan berulang. |
| `KOLB_AE_05` | 103 | Saya terdorong membuat rencana tindakan setelah mempelajari suatu konsep baru. |
| `KOLB_AE_06` | 104 | Saya cenderung mengevaluasi keberhasilan ide berdasarkan hasil ketika ide tersebut diterapkan. |

## Lampiran 3 – Kode Python scoring lengkap

Kode ini sudah dijalankan pada dataset dan hasilnya sama dengan angka di dokumen ini. Salin ke notebook; path file menyesuaikan struktur folder.

```python
import pandas as pd
import numpy as np

raw = pd.read_excel("dataset/raw/Survei_Gaya_Belajar_IPS_IPK_NO_PII.xlsx")
cols = list(raw.columns)

VARK_MAP = {
    1: {
        'melihat diagram, bagan, peta konsep, atau ilustrasi hubungan antarbagian.': 'V',
        'mendengar penjelasan dan dapat berdiskusi atau bertanya secara lisan.': 'A',
        'membaca uraian tertulis, definisi, atau catatan yang terstruktur.': 'R',
        'mencoba contoh, simulasi, praktik, atau melihat penerapan nyata.': 'K',
    },
    2: {
        'mengikuti gambar langkah demi langkah atau tangkapan layar.': 'V',
        'meminta seseorang menjelaskan langkah-langkahnya secara lisan.': 'A',
        'membaca petunjuk tertulis dan daftar langkah.': 'R',
        'langsung mencoba sambil memperbaiki kesalahan yang muncul.': 'K',
    },
    3: {
        'membuat peta konsep, skema, warna, atau diagram.': 'V',
        'menjelaskan materi dengan suara keras atau berdiskusi dengan teman.': 'A',
        'menulis ulang ringkasan, membuat daftar, atau membaca catatan.': 'R',
        'berlatih soal, mencoba contoh, atau menghubungkan materi dengan pengalaman.': 'K',
    },
    4: {
        'ada visualisasi yang menunjukkan hubungan antar konsep.': 'V',
        'penjelasan disampaikan dengan contoh verbal dan tanya jawab.': 'A',
        'tersedia teks atau rumusan tertulis yang dapat saya baca kembali.': 'R',
        'ada demonstrasi, studi kasus, atau aktivitas langsung.': 'K',
    },
    5: {
        'melihat contoh hasil akhir atau struktur visual tugas.': 'V',
        'mendengarkan penjelasan dosen tentang apa yang harus dilakukan.': 'A',
        'membaca instruksi dan rubrik secara rinci.': 'R',
        'mencoba bagian awal tugas untuk memahami apa yang diminta.': 'K',
    },
    6: {
        'melihat peta atau petunjuk arah visual.': 'V',
        'bertanya kepada orang lain dan mendengarkan arahannya.': 'A',
        'membaca nama jalan, papan petunjuk, atau instruksi tertulis.': 'R',
        'berjalan mencoba rute sambil menyesuaikan arah.': 'K',
    },
    7: {
        'grafik, diagram, dan visualisasi pola.': 'V',
        'penjelasan lisan tentang makna hasil.': 'A',
        'tabel, uraian tertulis, dan penjelasan metodologis.': 'R',
        'contoh penerapan data pada kasus nyata.': 'K',
    },
    8: {
        'menggambar skema atau menunjukkan visual.': 'V',
        'menjelaskannya melalui percakapan.': 'A',
        'menuliskan poin-poin atau memberi bacaan singkat.': 'R',
        'menunjukkan bagaimana ide tersebut bekerja melalui contoh atau demonstrasi.': 'K',
    },
    9: {
        'infografik, diagram, atau video dengan visual yang jelas.': 'V',
        'rekaman audio, diskusi sinkron, atau penjelasan verbal.': 'A',
        'modul teks, transkrip, dan catatan yang lengkap.': 'R',
        'simulasi, kuis interaktif, latihan, atau aktivitas berbasis kasus.': 'K',
    },
    10: {
        'membuat sketsa atau memetakan masalah secara visual.': 'V',
        'membicarakan masalah dengan orang lain untuk memperoleh ide.': 'A',
        'menuliskan fakta, alternatif, dan langkah pemecahan.': 'R',
        'mencoba beberapa solusi dan belajar dari hasil percobaan.': 'K',
    },
    11: {
        'video animasi, diagram interaktif, atau infografik.': 'V',
        'podcast, rekaman kuliah, atau diskusi kelompok.': 'A',
        'artikel, buku, modul, atau catatan tertulis.': 'R',
        'tutorial praktik, proyek, eksperimen, atau studi kasus.': 'K',
    },
    12: {
        'dapat menggambarkan hubungan konsep tersebut dalam bentuk visual.': 'V',
        'dapat menjelaskannya secara lisan dengan kata-kata saya sendiri.': 'A',
        'dapat menuliskan definisi dan penjelasannya secara runtut.': 'R',
        'dapat menerapkannya untuk menyelesaikan masalah atau tugas nyata.': 'K',
    },
    13: {
        'mencari bagian visual terlebih dahulu untuk memahami gambaran besar.': 'V',
        'mendengarkan penjelasan terkait sebelum atau sambil belajar.': 'A',
        'membaca dan membuat catatan secara sistematis.': 'R',
        'mencari latihan atau contoh penerapan setelah memahami pokok materi.': 'K',
    },
    14: {
        'ditunjukkan bagian mana yang perlu diperbaiki secara visual atau dengan markup.': 'V',
        'dibicarakan secara langsung sehingga saya dapat bertanya.': 'A',
        'diberikan komentar tertulis yang rinci.': 'R',
        'diberi kesempatan mencoba revisi dan melihat dampaknya.': 'K',
    },
    15: {
        'menyusun bagan, desain, atau struktur visual.': 'V',
        'berdiskusi, mempresentasikan, atau mengoordinasikan secara lisan.': 'A',
        'menulis dokumen, rangkuman, atau notulen.': 'R',
        'mengerjakan prototipe, praktik, atau implementasi.': 'K',
    },
    16: {
        'melihat representasi visual tentang bagaimana hal itu bekerja.': 'V',
        'mendengar seseorang menjelaskannya dan berdialog.': 'A',
        'membaca penjelasan tertulis yang lengkap.': 'R',
        'mengalami atau mencoba secara langsung.': 'K',
    },
}

LIKERT_GROUPS = {
    'HM_ACT': list(range(17, 25)),   # 8 item
    'HM_REF': list(range(25, 33)),   # 8 item
    'HM_THE': list(range(33, 41)),   # 8 item
    'HM_PRA': list(range(41, 49)),   # 8 item
    'FS_ACT': list(range(49, 53)),   # 4 item
    'FS_REF': list(range(53, 57)),   # 4 item
    'FS_SEN': list(range(57, 61)),   # 4 item
    'FS_INT': list(range(61, 65)),   # 4 item
    'FS_VIS': list(range(65, 69)),   # 4 item
    'FS_VRB': list(range(69, 73)),   # 4 item
    'FS_SEQ': list(range(73, 77)),   # 4 item
    'FS_GLO': list(range(77, 81)),   # 4 item
    'KOLB_CE': list(range(81, 87)),   # 6 item
    'KOLB_RO': list(range(87, 93)),   # 6 item
    'KOLB_AC': list(range(93, 99)),   # 6 item
    'KOLB_AE': list(range(99, 105)),   # 6 item
}


# =====================================================================
# 0. Data: buang 2 respons kosong total (ID 11 & 133), tandai straightliner
# =====================================================================
survey_cols = cols[1:105]
df = raw.dropna(subset=survey_cols, how="all").copy().reset_index(drop=True)
likert_cols = cols[17:105]
df["flag_straightline"] = df[likert_cols].nunique(axis=1).eq(1)
ok = ~df["flag_straightline"]            # dipakai untuk norma & label skala Likert

def items(code):
    return df[[cols[p] for p in LIKERT_GROUPS[code]]]

# =====================================================================
# 1. VARK  (Fleming) – format survei = format asli, aturan dipakai apa adanya
# =====================================================================
def pilihan(sel):
    return [p.strip() for p in str(sel).split(";") if p.strip()]

for m in "VARK":
    df[f"VARK_{m}"] = 0
for q in range(1, 17):
    for idx, sel in df[cols[q]].items():
        for opsi in pilihan(sel):
            df.at[idx, f"VARK_{VARK_MAP[q][opsi]}"] += 1
df["VARK_total"] = df[[f"VARK_{m}" for m in "VARK"]].sum(axis=1)

def stepping_distance(total):
    # Fleming: total 16-21 -> 1, 22-27 -> 2, 28-32 -> 3, >32 -> 4
    return 1 if total <= 21 else 2 if total <= 27 else 3 if total <= 32 else 4

def vark_profile(row):
    skor = {m: row[f"VARK_{m}"] for m in "VARK"}
    urut = sorted(skor.items(), key=lambda x: -x[1])          # n1 >= n2 >= n3 >= n4
    s = stepping_distance(row["VARK_total"])
    pref = [urut[0][0]]
    for i in range(3):
        if urut[i][1] - urut[i + 1][1] > s:                    # selisih > stepping -> berhenti
            break
        pref.append(urut[i + 1][0])
    return "".join(m for m in "VARK" if m in pref)

df["VARK_profile"] = df.apply(vark_profile, axis=1)           # contoh: "K", "VK", "VARK"
df["VARK_n_modal"] = df["VARK_profile"].str.len()             # 1 = single ... 4 = quadmodal
df["VARK_type"] = df["VARK_n_modal"].map({1: "Single", 2: "Bimodal", 3: "Trimodal", 4: "Quadmodal"})

# =====================================================================
# 2. Felder-Silverman (ILS) – skor = jumlah kutub a - jumlah kutub b
# =====================================================================
FS_DIMS = {  # nama dimensi: (kutub a, kutub b)
    "FS_ACT_REF": ("FS_ACT", "FS_REF"),
    "FS_SEN_INT": ("FS_SEN", "FS_INT"),
    "FS_VIS_VRB": ("FS_VIS", "FS_VRB"),
    "FS_SEQ_GLO": ("FS_SEQ", "FS_GLO"),
}
for dim, (a, b) in FS_DIMS.items():
    s = items(a).sum(axis=1) - items(b).sum(axis=1)            # rentang -16 .. +16
    df[f"{dim}_score"] = s
    df[f"{dim}_pole"] = np.where(s > 0, a[3:], np.where(s < 0, b[3:], "NEUTRAL"))
    # ambang ILS (1-3 balanced, 5-7 moderate, 9-11 strong dari maks 11) diskalakan ke maks 16
    df[f"{dim}_strength"] = pd.cut(s.abs(), [-1, 5, 11, 16], labels=["Balanced", "Moderate", "Strong"]).astype(str)
    df.loc[~ok, [f"{dim}_pole", f"{dim}_strength"]] = np.nan  # straightliner: label tidak dapat ditentukan

# =====================================================================
# 3. Kolb (LSI) – AC-CE dan AE-RO, titik potong = median sampel (cara manual LSI)
# =====================================================================
for mode in ["CE", "RO", "AC", "AE"]:
    df[f"KOLB_{mode}_sum"] = items(f"KOLB_{mode}").sum(axis=1)  # rentang 6..30
df["KOLB_ACCE"] = df["KOLB_AC_sum"] - df["KOLB_CE_sum"]       # + = abstrak, - = konkret
df["KOLB_AERO"] = df["KOLB_AE_sum"] - df["KOLB_RO_sum"]       # + = aktif,   - = reflektif
cut_acce = df.loc[ok, "KOLB_ACCE"].median()
cut_aero = df.loc[ok, "KOLB_AERO"].median()

def kolb_style(r):
    abstrak = r["KOLB_ACCE"] > cut_acce                        # manual: <= cut -> sisi konkret
    aktif = r["KOLB_AERO"] > cut_aero                          # manual: <= cut -> sisi reflektif
    return {(False, True): "Accommodating", (False, False): "Diverging",
            (True, True): "Converging", (True, False): "Assimilating"}[(abstrak, aktif)]

df["KOLB_style"] = df.apply(kolb_style, axis=1)
df.loc[~ok, "KOLB_style"] = np.nan

# =====================================================================
# 4. Honey-Mumford (LSQ) – skor per gaya + pita preferensi berbasis persentil (norma lokal)
# =====================================================================
HM = ["HM_ACT", "HM_REF", "HM_THE", "HM_PRA"]
for g in HM:
    df[f"{g}_sum"] = items(g).sum(axis=1)                       # rentang 8..40
    pr = df.loc[ok, f"{g}_sum"].rank(pct=True, method="average")
    df.loc[ok, f"{g}_pct"] = pr
    df.loc[ok, f"{g}_band"] = pd.cut(pr, [0, .10, .30, .70, .90, 1.0001],
        labels=["Very low", "Low", "Moderate", "Strong", "Very strong"]).astype(str)

def hm_dominant(r):
    pct = {g: r[f"{g}_pct"] for g in HM}
    if any(pd.isna(v) for v in pct.values()):
        return np.nan
    top = max(pct.values())
    return "+".join(g[3:] for g in HM if pct[g] == top)       # seri -> "ACT+REF"

df["HM_dominant"] = df.apply(hm_dominant, axis=1)

# =====================================================================
# 5. Validasi
# =====================================================================
assert df[[f"VARK_{m}" for m in "VARK"]].stack().between(0, 16).all()
assert df["VARK_total"].between(16, 64).all()
for dim in FS_DIMS:
    assert df[f"{dim}_score"].between(-16, 16).all()
assert df[["KOLB_ACCE", "KOLB_AERO"]].stack().between(-24, 24).all()
r = df.loc[df["ID"] == 4].iloc[0]                              # cek manual (lihat dokumen)
assert (r.VARK_V, r.VARK_A, r.VARK_R, r.VARK_K, r.VARK_profile) == (4, 3, 4, 15, "K")
print("Scoring selesai:", len(df), "responden;", int(ok.sum()), "punya label skala Likert")
```
