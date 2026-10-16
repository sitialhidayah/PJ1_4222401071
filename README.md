# PJ1 — Deteksi Objek dengan YOLO (RET503)

**Nama:** Siti Robbiah Al-Hidayah
**NIM:** 4222401071
**Tanggal eksekusi:** 16 Oktober 2026
**Platform:** Google Colab (GPU T4)

## 1. Tujuan

Melatih model deteksi objek YOLO pada dataset kecil milik sendiri, lalu membandingkan tiga eksperimen untuk melihat pengaruh (a) transfer learning dan (b) ukuran gambar masukan. Hasil akhir dinilai di data test yang hanya dipakai sekali.

## 2. Dataset

Dataset diambil sebagai subset COCO dengan skrip `00_ambil_subset_coco.py`, lalu diperiksa anotasinya (`01_cek_anotasi.py`: 0 masalah), diuji konversi YOLO → COCO → YOLO, dan dibagi 70/20/10.

**Kelas dipilih:** `person`, `cup`, `tv`.
`person` adalah kelas dominan, sedangkan `cup` dan `tv` jauh lebih sedikit. Ketidakseimbangan ini disengaja agar efeknya bisa diamati.

| Split | Gambar | person | cup | tv |
|---|---|---|---|---|
| train | 230 | 707 | 56 | 26 |
| val | 66 | 261 | 34 | 3 |
| test | 34 | 97 | 29 | 4 |

Sidik jari label dataset: `B12C14E55348`.

## 3. Eksperimen

Model: `yolo26n`, 30 epoch, batch 16, seed 0. Tiap eksperimen hanya mengubah **satu** hal dari E1.

| Eksperimen | Perubahan | imgsz | Pretrained |
|---|---|---|---|
| E1_pretrained | (acuan) | 416 | Ya |
| E2_scratch | tanpa bobot pretrained | 416 | Tidak |
| E3_tuning | ukuran gambar | 640 | Ya |

## 4. Hasil (data validasi)

| Eksperimen | mAP50 | mAP50-95 | Precision | Recall | Inferensi (ms) | Waktu latih (menit) |
|---|---|---|---|---|---|---|
| E1_pretrained | 0,532 | 0,391 | 0,869 | 0,462 | 6,9 | 2,0 |
| E2_scratch | 0,014 | 0,003 | 0,714 | 0,027 | 4,0 | 1,9 |
| E3_tuning | 0,795 | 0,525 | 0,762 | 0,706 | 8,0 | 2,5 |

Grafik latihan: `E1_pretrained_results.png`, `E2_scratch_results.png`, `E3_tuning_results.png`.
Confusion matrix: `E*_confusion_matrix.png`.

**Model terpilih:** E3_tuning (mAP50-95 val tertinggi).

## 5. Hasil di data test (dipakai sekali)

| Model | mAP50 | mAP50-95 |
|---|---|---|
| E3_tuning | 0,808 | 0,578 |

## 6. Analisis

**E1 vs E2 (transfer learning).** E1 yang memakai bobot pretrained mencapai mAP50-95 0,391, sedangkan E2 yang dilatih dari nol hanya 0,003. Dengan hanya 230 gambar latih dan 30 epoch, model dari nol belum sempat mempelajari ciri dasar seperti tepi, bentuk, dan tekstur, sedangkan model pretrained sudah memilikinya dan tinggal menyesuaikan diri. Precision E2 (0,714) tidak boleh dianggap bagus karena recall-nya hanya 0,027: model hampir tidak menghasilkan deteksi, sehingga sedikit tebakan yang keluar kebetulan benar.

**E1 vs E3 (ukuran gambar).** Dengan mengubah imgsz dari 416 ke 640 saja, mAP50-95 naik dari 0,391 ke 0,525 dan recall naik dari 0,462 ke 0,706. Dugaan saya, objek kecil (misalnya `cup` dan `tv`, atau orang yang jauh) hanya menempati sedikit piksel pada 416, sehingga lebih mudah terlewat; pada 640 detailnya lebih banyak. Dugaan ini tidak diuji langsung, karena saya tidak mengukur performa per ukuran objek. Harganya: waktu inferensi naik dari 6,9 ke 8,0 ms per gambar dan waktu latih dari 2,0 ke 2,5 menit. Precision juga turun dari 0,869 ke 0,762, artinya model lebih berani menebak sehingga lebih banyak tebakan yang salah, tetapi jauh lebih sedikit objek yang terlewat.

**Confusion matrix dan ketidakseimbangan kelas.** Pada data val, E3 menemukan 246 dari 261 kotak `person`, 34 dari 34 `cup`, dan 3 dari 3 `tv`, dengan nyaris tanpa tertukar antar kelas. Angka besar di kolom `background` (misalnya 4679 pada baris `person`) berasal dari tebakan berkeyakinan sangat rendah yang ikut dihitung pada pembuatan matriks, sehingga tidak sama dengan Precision di tabel. Kelas `tv` hanya punya 26 kotak di train dan 3 di val, sehingga hasilnya sangat tidak pasti: satu kesalahan saja mengubah angka sebesar sepertiga. Saya tidak menghitung mAP per kelas, jadi saya tidak bisa menyatakan kelas mana yang terlemah. Yang bisa dikatakan hanya bahwa `person` paling banyak datanya (sekitar 90% kotak train) dan `tv` paling sedikit (sekitar 3%).

**Hasil test.** E3 memperoleh mAP50-95 0,578 di data test, lebih tinggi daripada di val (0,525). Saya tidak menyimpulkan model menjadi lebih baik. Data test hanya 34 gambar dengan 4 kotak `tv`, sehingga selisih ini mudah muncul karena kebetulan. Angka 0,578 dilaporkan apa adanya sebagai hasil akhir.

## 7. Kesimpulan dan keterbatasan

- Transfer learning sangat menentukan pada data sekecil ini.
- Menaikkan imgsz ke 640 memberi peningkatan terbesar dengan biaya waktu yang kecil.
- Model terpilih E3_tuning cukup untuk tujuan latihan ini, tetapi belum layak untuk dipakai sungguhan: kelas `cup` dan `tv` masih sedikit datanya, data val dan test kecil, dan tidak ada mAP per kelas.
- Perbaikan yang bisa dicoba: menambah gambar untuk kelas jarang, memeriksa mAP per kelas, dan mencoba model atau epoch yang lebih besar.

## 8. Isi repositori

| File | Keterangan |
|---|---|
| `Praktik_YOLO_Siti.ipynb` | Notebook yang dijalankan di Colab |
| `hasil_praktik_4222401071.txt` / `.csv` | Ringkasan hasil ketiga eksperimen |
| `E1_pretrained_*`, `E2_scratch_*`, `E3_tuning_*` | `results.png`, `results.csv`, `confusion_matrix.png`, `args.yaml` |
| `E1_pretrained_best.pt`, `E3_tuning_best.pt` | Bobot terbaik |
| `README.md` | Laporan ini |

Skrip `00`–`06` (`.py`) berasal dari dosen dan tidak diubah.
