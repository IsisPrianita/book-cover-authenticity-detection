# book-cover-authenticity-detection

Sistem deteksi keaslian sampul buku menggunakan feature matching berbasis ORB dan SIFT dengan ensemble decision, dilengkapi pipeline augmentasi dan pembuatan sampel palsu untuk keperluan pengujian.

---

## Deskripsi

Proyek ini mengembangkan sistem verifikasi sampul buku yang mengklasifikasikan gambar ke dalam tiga kelas: **AUTHENTIC**, **SUSPICIOUS**, dan **COUNTERFEIT**. Sistem membandingkan gambar uji terhadap referensi asli menggunakan dua detektor fitur (ORB dan SIFT), kemudian menggabungkan hasilnya melalui ensemble decision dengan threshold yang dituning per buku secara otomatis.

---

## Struktur Repository

```
.
├── compvis.ipynb        # Notebook utama: feature matching, threshold tuning, evaluasi
├── augment1.py          # Augmentasi gambar untuk data uji (rotasi, blur, crop, noise, dll.)
├── generatefake.py      # Pembuatan sampel palsu dengan kombinasi degradasi visual
├── dataset1/
│   └── [NamaBuku]/
│       ├── reference/
│       │   └── asli.jpg         # Gambar referensi asli
│       ├── test/                # Hasil augmentasi (auto-generated)
│       └── fake/                # Sampel palsu (auto-generated)
└── output_v6/
    ├── hasil_v6.csv             # Hasil klasifikasi lengkap
    ├── confusion_matrix_v6.png  # Confusion matrix ensemble
    └── matches/                 # Visualisasi feature matching per pasangan gambar
```

---

## Pipeline

### 1. Augmentasi Data Uji — `augment1.py`

Menghasilkan variasi gambar asli di folder `test/` untuk setiap buku:

| Augmentasi | Detail |
|---|---|
| Rotasi | -30, -15, -10, +10, +15, +30 derajat |
| Brightness | Gelap (0.5x), Normal, Terang (1.4x) |
| Blur | Ringan (5x5), Sedang (11x11), Berat (21x21) |
| Noise | Gaussian noise std=20 |
| Crop + Resize | 90%, 75%, 60%, 50% dari ukuran asli |
| Perspective | Transformasi perspektif ringan |

### 2. Pembuatan Sampel Palsu — `generatefake.py`

Menghasilkan 13 varian gambar palsu di folder `fake/` dengan kombinasi:

- Kontras ekstrem + blur / noise / crop
- Watermark teks (COPY, BAJAKAN) + degradasi
- Crop agresif + JPEG artifacts (quality=8)
- Multi-efek struktural berlapis

### 3. Feature Matching & Klasifikasi — `compvis.ipynb`

- Ekstraksi fitur menggunakan **ORB** (4000 keypoints) dan **SIFT** (4000 keypoints)
- Matching dengan BFMatcher + Lowe's ratio test (threshold=0.75)
- Skor dihitung sebagai rasio terhadap baseline (brightness_normal.jpg)
- Threshold `t_auth` dan `t_susp` dituning otomatis per buku menggunakan grid search
- Ensemble decision menggabungkan prediksi ORB dan SIFT dengan aturan confidence-based
- Visualisasi matching disimpan untuk setiap pasangan gambar uji

---

## Kelas Prediksi

| Kelas | Keterangan |
|---|---|
| AUTHENTIC | Gambar sangat mirip dengan referensi asli |
| SUSPICIOUS | Gambar mungkin merupakan salinan yang telah dimodifikasi |
| COUNTERFEIT | Gambar berbeda signifikan dari referensi asli |

---

## Instalasi

```bash
pip install opencv-python opencv-contrib-python numpy pandas matplotlib scikit-learn
```

> `opencv-contrib-python` diperlukan untuk akses SIFT.

---

## Cara Menjalankan

1. Siapkan dataset dengan struktur `dataset1/[NamaBuku]/reference/asli.jpg`
2. Jalankan augmentasi untuk membuat data uji:
   ```bash
   python augment1.py
   ```
3. Jalankan pembuatan sampel palsu:
   ```bash
   python generatefake.py
   ```
4. Jalankan seluruh cell di `compvis.ipynb` untuk klasifikasi dan evaluasi

---

## Output

- `output_v6/hasil_v6.csv` — hasil prediksi lengkap per gambar
- `output_v6/confusion_matrix_v6.png` — confusion matrix ensemble
- `output_v6/matches/` — visualisasi feature matching ORB dan SIFT side-by-side

---

## Teknologi

- [OpenCV](https://opencv.org/) — ORB, SIFT, BFMatcher, augmentasi gambar
- [scikit-learn](https://scikit-learn.org/) — evaluasi, confusion matrix
- [NumPy](https://numpy.org/) / [Pandas](https://pandas.pydata.org/) — pengolahan data
- [Matplotlib](https://matplotlib.org/) — visualisasi

---

## Lisensi

MIT License
