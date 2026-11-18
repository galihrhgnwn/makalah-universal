# makalah-universal

Generator **makalah kuliah berbahasa Indonesia** dalam format `.docx` siap kumpul, untuk berbagai perguruan tinggi. Format dokumen menyesuaikan universitas, fakultas, dan program studi pengguna; bila tidak ada yang cocok, dipakai format umum yang lazim di perguruan tinggi Indonesia.

> **Forked** | Credit: Kurniawan Hafiz | Versi 5.0.0 | [English](README.en.md)

## Apa yang baru di v5

- **Universal**: tidak lagi khusus Sistem Informasi UNRI. Format dipilih otomatis dari kampus/prodi pengguna lewat `references/formats.json`, atau dipaksa dengan `meta.format`.
- **Format bawaan umum Indonesia**: A4, margin 4-3-3-3 cm, Times New Roman 12, spasi 1,5, rata kiri-kanan, daftar pustaka APA 7.
- **Makalah individu**: `meta.jumlah_anggota = 1` membuat cover satu baris "Nama Mahasiswa". Kelompok tetap "Nama Anggota 1..n".
- Margin, spasi, ukuran font, dan lebar teks mengikuti format terpilih, tidak lagi terkunci.
- Ditambah `package.json`, `package-lock.json`, dan `.gitignore`.

## Fitur

- Cover otomatis, daftar isi otomatis dengan nomor halaman, penomoran heading, rumus Word (equation), gambar, dan daftar pustaka APA 7 menggantung sekaligus rata kiri-kanan.
- Validator isi sebelum build (`scripts/check_content.py`) dan preview halaman penting ke PNG (`scripts/preview.sh`).
- Satu perintah untuk seluruh alur: `python3 scripts/build.py content.json out.docx`.

Daftar lengkap ada di [CHANGELOG.md](CHANGELOG.md).

## Format yang didukung

| id | Kampus | Catatan | Sumber pedoman |
|---|---|---|---|
| `default` | Umum Indonesia | A4, 4-3-3-3, TNR 12, spasi 1,5 | Kompilasi pedoman berbagai PT |
| `unri:fmipa:si` | Universitas Riau | Sistem Informasi FMIPA | Pedoman Skripsi Prodi SI FMIPA UNRI (2017) |
| `unri:fakultas-teknik` | Universitas Riau | Fakultas Teknik | Panduan Penulisan Tugas Akhir FT UNRI (2012) |
| `unri:fakultas-pertanian` | Universitas Riau | Fakultas Pertanian | Pedoman Penulisan Skripsi FP UNRI (2025) |
| `ugm:feb` | Universitas Gadjah Mada | Fakultas Ekonomika dan Bisnis | Pedoman Penulisan Karya Ilmiah FEB UGM |
| `ugm:kimia` | Universitas Gadjah Mada | Kimia FMIPA | Pedoman Penulisan Proposal dan Skripsi Kimia UGM (2026) |

Cara format dipilih:
1. `meta.format` di content.json (id dari tabel di atas).
2. Otomatis dari `meta.universitas`, `meta.fakultas`, `meta.prodi`. Utamakan format Universitas Riau bila kampusnya UNRI.
3. Tidak ada yang cocok = format `default`.

Untuk menambah format baru, tambahkan entri di `references/formats.json`. Penjelasan field ada di [`references/schema.md`](references/schema.md).

## Cara pakai dengan AI

Panggil skill `/makalah-universal` lalu jawab pertanyaan yang diajukan: jenis (individu/kelompok), universitas, fakultas, program studi, judul, mata kuliah, dosen pengampu, materi, dan referensi. AI akan menyesuaikan format dengan prodi Anda.

Contoh:

```
/makalah-universal
Buatkan makalah.
- Jenis: kelompok, 4 orang
- Universitas: Universitas Riau
- Fakultas: Matematika dan Ilmu Pengetahuan Alam
- Program Studi: Sistem Informasi
- Judul: Kombinasi, Permutasi, dan Koefisien Binomial
- Mata kuliah: Matematika Diskrit
- Dosen pengampu: Nama Dosen, gelar.
- Materi / poin bahasan:
  - Pengertian dan jenis kombinasi
  - Pengertian dan jenis permutasi
  - Koefisien binomial
- Referensi: 10 jurnal, terbit 10 tahun terakhir, minimal 5 berbahasa Indonesia
- Studi kasus: 1, bertema TI/SI, tanpa referensi
```

Setelah file jadi:

- Edit nama anggota, NIM (`( NIM )`), dan bulan-tahun di kata pengantar.
- Klik kanan daftar isi lalu pilih **Update Field** supaya nomor halaman pasti, terutama bila makalah memuat rumus.

## Cara pasang (skill AI)

1. Di halaman repo ini, klik **Code**, lalu **Download ZIP**.
2. Di Claude/assis, buka **Settings**, lalu menu **Skills**, dan upload file zip tadi.
3. Mulai chat baru supaya skill terbaca.

Catatan:

- Kalau sudah ada skill dengan nama sama, hapus yang lama dulu sebelum upload versi baru.
- Zip dari GitHub berisi folder bernama `makalah-universal-main`. Kalau upload ditolak karena nama folder, ekstrak zip-nya, ganti nama foldernya jadi `makalah-universal`, kompres lagi, lalu upload.

## Cara pakai manual (tanpa AI)

Kebutuhan: Node.js, paket npm `docx`, Python 3, LibreOffice (`soffice`) untuk menghitung nomor halaman, dan opsional `pandoc` dan `poppler-utils` (`pdftoppm`, `pdftotext`) untuk cek rumus dan preview.

```bash
npm install

# 1. salin dan edit contoh isi (skemanya ada di references/schema.md)
cp assets/example_content.json content.json

# 2. satu perintah: cek isi, bangun, daftar isi, validasi, preview
python3 scripts/build.py content.json hasil/Makalah_Judul.docx --tabel --preview
```

Opsi `build.py`:

| Opsi | Fungsi |
|---|---|
| `--strict` | Peringatan validator ikut menghentikan build |
| `--tabel` | Cetak tabel status referensi |
| `--preview` | Render halaman penting ke PNG di `hasil/Makalah_Judul_preview/` |
| `--tanpa-cek` | Lewati validator (tidak disarankan) |

Variabel lingkungan `MAKALAH_SOFFICE` bisa dipakai untuk menunjuk perintah LibreOffice tertentu.

## Struktur folder

```
makalah-universal/
├── README.md                   # file ini
├── README.en.md                # versi Inggris
├── SKILL.md                    # instruksi skill untuk AI
├── CHANGELOG.md                # catatan perubahan
├── LICENSE                     # MIT
├── package.json                # metadata npm + dependency docx
├── package-lock.json
├── variant.json                # kompatibilitas mundur (tidak dipakai sejak v5)
├── assets/
│   ├── logo_unri.png           # logo UNRI untuk cover
│   ├── example_content.json    # contoh isi makalah (rumus, gambar, sitasi)
│   ├── example_diagram.png     # gambar untuk contoh
│   └── prompt_template.md      # template prompt bila skill tidak terpasang
├── references/
│   ├── schema.md               # skema content.json
│   └── formats.json            # daftar format dokumen per kampus/prodi
└── scripts/
    ├── build.py                # satu perintah: cek, bangun, daftar isi, validasi, preview
    ├── check_content.py        # validator isi sebelum build
    ├── build_makalah.js        # membangun .docx dari content.json
    ├── omml.js                 # konverter LaTeX ke equation Word
    ├── finalize.py             # mengisi daftar isi dan nomor halaman
    └── preview.sh              # render halaman penting ke PNG
```

## Format `content.json`

Ringkasnya: `meta` (judul, matkul, dosen, universitas, tahun, jumlah_anggota, kota, bulan_tahun, opsional fakultas, prodi, format, logo, maks_tabel, ref_config), `kata_pengantar` (array paragraf), `bab` (array bab berisi `sections` dan `blocks`), `daftar_pustaka` (array string APA 7), `verifikasi_referensi`, dan `sumber_dibuang`. Tipe blok: `p`, `quote`, `list`, `table`, `equation`, `image`. Penjelasan lengkap ada di [`references/schema.md`](references/schema.md).

Field baru yang penting:

- `meta.format`: paksa format dengan id (mis. `unri:fmipa:si`). Kosong = otomatis.
- `meta.jumlah_anggota`: `1` untuk individu, lebih dari 1 untuk kelompok.

## Batasan yang diketahui

- LibreOffice tanpa modul Math tidak menggambar rumus. Di mesin seperti itu rumus kosong di preview dan nomor halaman daftar isi dihitung dari perkiraan. Di Word rumus tampil normal; klik kanan daftar isi lalu **Update Field** untuk angka pasti.
- Konverter rumus mendukung subset LaTeX (lihat schema). Perintah di luar daftar menimbulkan galat saat build, bukan hasil diam-diam salah.
- Validator membaca sitasi dengan pola `Penulis (tahun)` dan `(Penulis, tahun)`. Pola yang tidak biasa bisa terlewat atau salah baca; pakai `meta.lewati_sitasi` untuk kasus yang sengaja.
- Bahasa referensi dihitung dari `verifikasi_referensi[].bahasa`; tanpa itu hanya perkiraan.
- Format di `references/formats.json` adalah kumpulan pedoman yang tersedia saat ini. Bila kampus/prodi Anda belum ada, pakai format `default` atau kirimkan pedoman resminya untuk ditambahkan.
- Skill tidak bisa memastikan sebuah link terbuka dari mesinmu. Periksa kembali setiap referensi sebelum dikumpulkan.

## Pemecahan masalah

| Gejala | Penyebab dan solusi |
|---|---|
| `Cannot find module 'docx'` | Jalankan `npm install` di folder skill, atau pasang global. |
| `LibreOffice (soffice) tidak ditemukan` | Pasang LibreOffice atau set `MAKALAH_SOFFICE`. |
| Cover tanpa logo | Format terpilih tidak punya logo; unggah logo lewat `meta.logo`. |
| Font/margin tidak sesuai harapan | Cek `meta.format` dan entri `references/formats.json`; tambahkan format bila perlu. |
| Build berhenti dengan GALAT sitasi | Sitasi di teks harus ada di daftar pustaka dan sebaliknya; Bab III dan studi kasus tidak boleh bersitasi. |
| Rumus kosong di preview | Lihat batasan di atas. Cek di Word. |
| Nomor halaman daftar isi meleset | Klik kanan daftar isi, **Update Field**. |
| Zip ditolak saat upload skill | Ganti nama folder hasil ekstrak jadi `makalah-universal`, kompres ulang. |

## Catatan

- Logo UNRI adalah milik Universitas Riau. File logo disertakan hanya sebagai kelengkapan cover tugas kuliah.
- Nama anggota, NIM, dan tanggal selalu berupa placeholder dan harus diedit sendiri.
- Periksa kembali setiap referensi sebelum dikumpulkan.

## Lisensi dan kredit

Dirilis dengan lisensi MIT (lihat [LICENSE](LICENSE)). Credit: **Kurniawan Hafiz**. Format kampus dihimpun dari pedoman resmi masing-masing fakultas/prodi (tautan ada di `references/formats.json`).
