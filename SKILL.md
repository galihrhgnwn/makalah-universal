---
name: "makalah-universal"
description: "Membuat makalah kuliah berbahasa Indonesia dalam format .docx siap kumpul untuk berbagai perguruan tinggi. Format dokumen menyesuaikan universitas/fakultas/program studi pengguna lewat references/formats.json; bawaan format umum Indonesia. Mendukung cover individu dan kelompok, daftar isi otomatis, penomoran heading dan halaman, rumus Word (equation), gambar, serta daftar pustaka APA dengan link valid. Pakai skill ini untuk setiap permintaan makalah. Sebelum mulai, tanya jenis (individu/kelompok), universitas, fakultas, dan program studi."
---

# Makalah Universal (docx) v5

Skill ini menghasilkan makalah Bahasa Indonesia dalam format `.docx` yang siap dikumpulkan. Format dokumen tidak terkunci ke satu kampus: dipilih dari `references/formats.json` berdasarkan data kampus pengguna. Bila tidak ada format yang cocok, dipakai format umum perguruan tinggi Indonesia.

Semua perintah dijalankan dari folder skill ini (atau pakai path absolut). Skrip tidak bergantung pada path khusus; berkas sementara dibuat di folder sementara sistem.

## Alur kerja

1. **Kumpulkan input.** Tanya hanya yang belum diberikan, jangan lebih dari beberapa pertanyaan sekaligus:
   - **Jenis makalah: individu atau kelompok?** Bila kelompok, jumlah anggota (default 4). Individu berarti `meta.jumlah_anggota = 1`.
   - **Universitas, fakultas, dan program studi.** Dipakai memilih format. Utamakan format Universitas Riau bila kampusnya UNRI dan format prodinya tersedia.
   - Materi atau poin bahasan, judul makalah, mata kuliah, nama dosen pengampu.
   - Referensi dari pengguna (DOI, link, PDF); boleh ditambah referensi lain yang valid.
   - Syarat referensi (jumlah, rentang tahun terbit, minimal berbahasa Indonesia, ada/tidak studi kasus) bila belum disebut. Simpan di `meta.ref_config`.
   - Logo: pakai logo format terpilih bila ada (mis. UNRI). Pengguna boleh mengunggah logo sendiri lewat `meta.logo`.
2. **Pilih format.** Bila pengguna menyebut format tertentu, isi `meta.format` dengan id-nya (contoh `unri:fmipa:si`, `ugm:feb`). Tanpa itu, format dipilih otomatis dari `meta.universitas`/`meta.fakultas`/`meta.prodi`; bila tidak cocok, format umum (`default`). Detail di bagian Pemilihan format.
3. **Cari dan verifikasi referensi** (lihat bagian Referensi). Jangan menulis isi sebelum tahu sumber mana yang benar-benar bisa dipakai. Catat hasilnya di `verifikasi_referensi` dan `sumber_dibuang`.
4. **Tulis `content.json`** sesuai `references/schema.md`. Rumus ditulis dengan LaTeX, gambar lewat blok `image`.
5. **Bangun dengan satu perintah:**
   ```bash
   python3 scripts/build.py content.json hasil/Makalah_<Judul>.docx --tabel --preview
   ```
   Urutannya: `check_content.py` -> `build_makalah.js` -> cek dukungan rumus LibreOffice -> `finalize.py` -> validasi struktur -> `preview.sh`. `--tabel` mencetak tabel status referensi; `--strict` menjadikan peringatan sebagai galat.
6. **Perbaiki galat dan tinjau peringatan.** Galat (sitasi di Bab III atau studi kasus, sitasi tanpa entri pustaka atau sebaliknya, daftar pustaka tidak urut abjad, jumlah atau tahun referensi di luar syarat, rumus rusak) harus dibereskan. Peringatan ditinjau dan diperbaiki bila memang benar.
7. **Cek visual.** Lihat PNG di folder `*_preview/`: cover, daftar isi, halaman pertama tiap bab, halaman berisi rumus/gambar, dan daftar pustaka. Pastikan cover satu halaman, nomor halaman sesuai, dan entri daftar pustaka rata kiri-kanan dengan baris kedua menjorok. Bila LibreOffice di mesin tidak punya modul Math, rumus tampil kosong di preview; itu keterbatasan tampilan, rumusnya tersimpan sebagai equation Word dan lolos cek pandoc.
8. **Serahkan** file `.docx` hasil `build.py` (bukan berkas sementara). Di ringkasan akhir tulis:
   - yang perlu diedit pengguna (nama, NIM, bulan-tahun di kata pengantar),
   - tabel status referensi dari `--tabel` dan daftar sumber yang dibuang beserta alasannya,
   - bila nomor halaman daftar isi dihitung dari perkiraan, minta pengguna klik kanan daftar isi lalu **Update Field** di Word.

## Pemilihan format

`references/formats.json` adalah daftar format. Prioritas pemilihan:
1. `meta.format` = id eksplisit (mis. `unri:fmipa:si`).
2. Pencocokan otomatis: nama universitas (paling kuat), lalu fakultas dan program studi.
3. Tidak ada yang cocok: format umum (`default`): A4, margin 4-3-3-3 cm, Times New Roman 12, spasi 1,5, rata kiri-kanan, cover tanpa logo, daftar pustaka APA 7.

Format awal yang tersedia:
- `default` - format umum perguruan tinggi Indonesia
- `unri:fmipa:si` - Sistem Informasi FMIPA Universitas Riau
- `unri:fakultas-teknik` - Fakultas Teknik Universitas Riau
- `unri:fakultas-pertanian` - Fakultas Pertanian Universitas Riau
- `ugm:feb` - Fakultas Ekonomika dan Bisnis Universitas Gadjah Mada
- `ugm:kimia` - Program Studi Kimia Universitas Gadjah Mada

Cara menambah format: salin salah satu entri di `references/formats.json`, lalu isi field `id`, `nama`, `kampus`, `fakultas`, `prodi`, `logo`, `margin`, `spasi`, `font`, `ukuran_font`, `cover_ukuran_font`, `cover_susunan`, dan `sumber` (tautan pedoman resmi). `cover_susunan` menentukan urutan baris identitas di cover (mis. `["prodi", "fakultas"]`). Jangan mengubah konstanta margin/font di skrip untuk kebutuhan satu format; tambahkan entri format sebagai gantinya.

## Format dokumen (umum)

- A4; margin, spasi, font, dan ukuran font mengikuti format terpilih (bawaan: 4-3-3-3 cm, Times New Roman 12, spasi 1,5). Cover memakai `cover_ukuran_font` format itu.
- Urutan: cover, daftar isi, kata pengantar, Bab I-III, daftar pustaka.
- Cover: judul, logo (bila format punya), "Disusun untuk memenuhi tugas mata kuliah ...", "Dosen Pengampu: ...", "Disusun oleh:" + tabel identitas, baris institusi sesuai `cover_susunan`, lalu tahun. Individu: satu baris "Nama Mahasiswa" tanpa nomor. Kelompok: nomor, "Nama Anggota n", NIM. Placeholder NIM: `( NIM )`.
- Cover dihitung halaman i tetapi nomornya disembunyikan. Daftar isi dan kata pengantar bernomor romawi di tengah bawah. Bab I dst bernomor angka mulai 1 di kanan bawah (posisi mengikuti format). Daftar pustaka tanpa nomor yang terlihat.
- Penomoran heading 1.1, 1.2, 2.1, dan sub-subbab 2.4.1. Paragraf di bawah subbab sejajar dengan teks subbab (bukan angkanya), baris pertama tetap menjorok.
- Daftar isi otomatis (field TOC, level 1-3). Entri pertama adalah "HALAMAN JUDUL": Heading 1 teks kecil berwarna putih di paling atas cover, tidak terlihat di cover tetapi muncul di daftar isi dengan halaman i.
- Semua Heading 1 memakai spasi sebelum 0 dan setelah 12 pt (permanen di skrip).
- Daftar pustaka gaya APA 7, urut abjad, menggantung (hanging indent) sekaligus rata kiri-kanan (justified).
- Rumus: blok `equation` menjadi equation Word (OMML) yang bisa diedit, dengan nomor opsional. Rumus sebaris ditulis `$...$`. Gambar: blok `image` dengan keterangan "Gambar n." otomatis.
- Hindari tabel (maksimal satu per makalah, ubah lewat `meta.maks_tabel`).

## Menulis isi

Struktur umum (sesuaikan dengan materi pengguna):

- **Kata pengantar**: puji syukur, terima kasih kepada dosen pengampu dan rekan kelompok, permohonan kritik dan saran. Tiga paragraf pendek.
- **Bab I Pendahuluan**: 1.1 Latar Belakang (dari gambaran umum ke masalah, dengan sitasi), 1.2 Rumusan Masalah (daftar bernomor, satu pertanyaan per materi), 1.3 Tujuan Penulisan (jumlah sama dengan rumusan), 1.4 Manfaat Penulisan. Studi kasus tidak dimasukkan ke latar belakang, rumusan masalah, maupun tujuan, kecuali pengguna memintanya.
- **Bab II Pembahasan**: satu subbab per butir materi pengguna, urut sesuai materi. Gunakan kutipan dari referensi di tiap subbab, dan sertakan contoh bila materinya memungkinkan. Bila materi punya pengertian dan jenis-jenis, buat subbab "Pengertian" dan "Jenis-jenis" dengan sub-subbab per jenis. Contoh diberi label "Contoh [judul sub materi]". Bila ada studi kasus, ikuti aturan studi kasus di bawah.
- **Bab III Penutup**: 3.1 Kesimpulan (menjawab tiap rumusan masalah), 3.2 Saran. Bab III tidak memakai sitasi dan tidak merujuk referensi. Isinya rangkuman temuan Bab II dan pemikiran penulis sendiri.

### Studi kasus

- Ditulis tanpa sitasi dan tanpa referensi. Tidak ada kutipan (Penulis, tahun) di dalamnya, dan tidak ada entri daftar pustaka khusus untuknya, kecuali pengguna memintanya.
- Tidak masuk ke Latar Belakang, Rumusan Masalah, dan Tujuan Penulisan. Tempatnya di Bab II (atau bagian yang ditunjuk pengguna).
- Studi kasus rekaan diberi label "contoh rekaan" di teksnya. Beri judul subbab yang memuat kata "Studi Kasus" supaya validator mengenalinya.

### Gaya penulisan

- Bahasa Indonesia baku, ilmiah, mengalir. Hindari tanda pisah panjang (em dash). Hindari pembuka klise ("Dalam era ..."), kata gemar AI (krusial, esensial, komprehensif, holistik, signifikan), daftar tiga yang dipaksakan, penutup satu kalimat yang menggurui, serta label tebal di dalam paragraf.
- Variasikan panjang kalimat dan awal kalimat. Gunakan "kami" hanya di kata pengantar, Bab III, dan bagian yang memang bersuara penulis.
- Semua kutipan berupa parafrase dengan kata sendiri. Jangan menyalin kalimat sumber. Kutipan langsung hanya bila perlu dan sangat pendek.
- Klaim yang disandarkan pada sumber hanya boleh sebatas yang benar-benar dibaca. Kalau hanya abstrak yang terbaca, tulis hal yang ada di abstrak saja. Kalau menafsirkan sendiri, tulis sebagai pendapat penulis ("menurut kami").
- Sitasi dalam teks: Penulis (2023), (Penulis, 2023), dua penulis dengan "dan" atau "&", tiga penulis atau lebih memakai "dkk.". Sitasi hanya di Bab I dan Bab II. Daftar pustaka hanya memuat sumber yang disitasi di Bab I atau Bab II.

## Referensi

### Mode pencarian (v4)

Sebelum mencari sumber, tentukan dulu mode pencariannya. Tidak perlu bertanya ke pengguna untuk ini.

1. **Cek Consensus AI.** Cari tool `mcp__Consensus__search` (bila hanya tampil sebagai tool tertunda, muat lewat ToolSearch), lalu coba panggil. Tool tidak ada, atau panggilan gagal karena konektor belum terhubung: pakai **pencarian web** (cara pada bagian "Verifikasi dan penulisan entri" di bawah).
2. **Consensus terpasang, terhubung, dan kuota belum habis: Consensus menjadi pencari referensi default**, bukan cadangan. Ini berlaku juga bila pengguna sudah memberi beberapa sumber sendiri; sumber pengguna tetap diverifikasi dan ikut dihitung.
3. **Kuota Consensus habis** (galat batas penggunaan atau pesan upgrade tanpa hasil): pakai pencarian web untuk sisa pekerjaan. Hasil yang sudah didapat sebelum kuota habis tetap boleh dipakai setelah diverifikasi. Galat batas laju sementara (terlalu banyak permintaan) bukan kuota habis: tunggu sekitar 30 detik dan ulangi sekali; bila tetap gagal, pakai pencarian web.
4. **Filter default Consensus** pada setiap panggilan `search`: `open_access=true`, `exclude_preprints=true`, dan `year_min` = tahun berjalan dikurangi 5 (tahun 2026 menjadi `year_min=2021`). Ketiganya memang permintaan pengguna sebagai standar, jadi aturan konektor "jangan pakai filter" tidak berlaku di sini.
5. **Permintaan pengguna di prompt menang atas default.** Bila pengguna menyebut filter lain (rentang tahun berbeda, preprint boleh, tidak harus open access, dan sebagainya), pakai itu menggantikan default yang bersangkutan. Rentang tahun dari `meta.ref_config` (mis. `umur_maks_tahun`) juga menggantikan 5 tahun. Filter Consensus di luar tiga default (`study_types`, `sjr_max`, `domain`, `citation_min`, dan seterusnya), fitur Consensus lain (mis. `create_thread`), serta skill lain (mis. humanizer atau stop-slop) hanya dipakai bila pengguna memintanya di prompt. Jangan melonggarkan atau menambah filter sendiri.
6. **Cara memanggil.** Query berupa istilah akademik yang spesifik (Inggris untuk topik umum), satu query per butir materi, maksimal 3 panggilan sekaligus.
7. **Sumber berbahasa Indonesia** (bila ada syarat `min_indonesia` atau pengguna memintanya): panggil Consensus dulu dengan `country="id"`, atau dengan query berbahasa Indonesia. Bila tetap tidak cukup, baru lengkapi dengan pencarian web (Garuda, SINTA, DOAJ, OJS jurnal).
8. **Kekurangan sumber.** Bila setelah verifikasi sumber dari Consensus belum memenuhi syarat jumlah atau bahasa, lengkapi dengan pencarian web. Filter default tidak dilonggarkan sendiri.
9. **Hasil Consensus tetap diverifikasi.** Consensus hanya menemukan kandidat. Penulis, tahun, judul, jurnal, volume, dan halaman dicocokkan dengan laman aslinya (DOI, laman jurnal, DOAJ, Garuda) seperti aturan di bawah. Paket gratis tidak selalu memberi DOI atau link: cari DOI atau laman artikelnya lewat pencarian web berdasarkan judul dan penulis, lalu buka. Tanpa link yang bisa dibuka dan metadatanya cocok, sumber dibuang dan dicatat di `sumber_dibuang`. Klaim di makalah hanya sebatas abstrak atau teks yang benar-benar terbaca.
10. **Pencatatan.** Isi `ditemukan_lewat` pada tiap entri `verifikasi_referensi`: `Consensus` atau `Pencarian web` (atau `Dari pengguna`). Kolom `dicek_lewat` tetap untuk cara verifikasinya (DOI, laman jurnal, dst). Bila tool Consensus mengembalikan pesan penggunaan atau ajakan upgrade, sampaikan apa adanya di akhir ringkasan akhir. Sitasi bernomor [1] ala konektor hanya untuk balasan chat; isi makalah tetap APA 7.

### Verifikasi dan penulisan entri (berlaku untuk semua mode)

- Coba buka setiap DOI atau link dari pengguna dengan tool pengambil halaman web. Bila terblokir, cari lewat tool pencarian web untuk menemukan judul, penulis, tahun, jurnal, volume, halaman, dan abstrak.
- Sumber yang tidak bisa diverifikasi atau tidak relevan tidak dipakai dan tidak masuk daftar pustaka. Katakan terus terang ke pengguna sumber mana dan kenapa (isi `sumber_dibuang`). Minta pengguna mengunggah PDF-nya bila ia ingin sumber itu dipakai.
- PDF yang diunggah pengguna boleh dibaca penuh dan dikutip lebih dalam.
- Boleh menambah sumber sendiri (jurnal, buku, artikel resmi) asal bisa diverifikasi dan linknya valid. Jangan pernah mengarang sumber, DOI, halaman, atau penulis. Bila daftar penulis tidak pasti, sampaikan ke pengguna.
- Jangan hanya mengandalkan DOI. Cari lewat beberapa basis data: DOAJ, Portal Garuda dan SINTA untuk jurnal Indonesia, Google Scholar, laman resmi jurnal (OJS), repositori kampus, serta arXiv atau CORE bila relevan. ResearchGate boleh dipakai sebagai pilihan terakhir.
- Syarat sumber non-DOI: teks lengkap atau minimal abstraknya bisa dibuka tanpa login dan tanpa bayar, dan metadatanya (penulis, tahun, judul, jurnal, volume, halaman) cocok dengan laman aslinya. ResearchGate sering menolak akses otomatis (403); cocokkan metadatanya lewat laman jurnal, DOAJ, atau Garuda. Kalau tetap tidak cocok, sumber tidak dipakai.
- Urutan prioritas link: DOI, laman artikel di situs resmi jurnal, halaman artikel di DOAJ, repositori institusi atau Garuda, lalu ResearchGate. Satu link per entri, dan hanya link yang benar-benar sudah dibuka atau dicek.
- Format di `content.json`: tiap entri satu string APA 7, `*tanda bintang*` untuk huruf miring (nama jurnal, volume, judul buku), urut abjad. DOI ditulis `https://doi.org/...`. Tanpa DOI, tulis URL laman artikelnya di akhir entri, tanpa kata "Diakses dari".
- Untuk setiap sumber isi `verifikasi_referensi`: `ref` (potongan awal entri, mis. "Alfa, A. (2023)"), `ditemukan_lewat` (`Consensus`, `Pencarian web`, atau `Dari pengguna`), `dicek_lewat` (DOI, laman jurnal, DOAJ, Garuda, dst), dan `bahasa` (`id` atau `en`). Dengan ini tabel status referensi dan syarat minimal bahasa Indonesia terhitung pasti, bukan perkiraan.
- Jumlah wajar untuk makalah mahasiswa: 5 sampai 8 sumber, atau sesuai syarat pengguna. Hitungan hanya untuk sumber yang disitasi di Bab I dan II.

## Hal yang sering salah

- Jangan menganggap hasil Consensus sudah valid. Metadata dan link tetap dicek ke laman asli sebelum masuk daftar pustaka.
- Jangan memakai Consensus tanpa tiga filter default (open access, tanpa preprint, 5 tahun terakhir) kecuali pengguna menggantinya di prompt, dan jangan menambah filter lain tanpa diminta.
- Jangan bertanya ke pengguna apakah Consensus terpasang atau kuotanya habis; cek sendiri lewat tool dan galatnya.
- Jangan lupa bertanya jenis individu/kelompok serta universitas/fakultas/prodi sebelum membangun. Menebak jumlah anggota atau kampus membuat cover salah.
- Jangan mengutak-atik konstanta margin/font di skrip untuk satu kampus; tambahkan entri di `references/formats.json`.
- Serahkan hasil `build.py`, bukan berkas mentah, karena daftar isi di berkas mentah kosong.
- Jangan membuang sumber hanya karena tidak punya DOI. Coba DOAJ, Garuda, laman jurnal, repositori, atau ResearchGate dulu.
- Jangan menaruh sitasi atau kutipan di Bab III maupun studi kasus. Validator akan menghentikan build.
- Jangan membuat daftar pustaka rata kiri saja. Entri harus rata kiri-kanan dan tetap menggantung.
- Rumus jangan ditulis sebagai teks biasa seperti "n!/(r!(n-r)!)". Pakai blok `equation` atau `$...$`. Perintah LaTeX di luar daftar yang didukung sengaja menimbulkan galat saat build; ganti dengan padanan yang didukung.
- Jangan menaruh tabel kecil untuk hal yang bisa ditulis sebagai kalimat.
- Nama, NIM, dan bulan-tahun kata pengantar dibiarkan placeholder dan disebut di ringkasan akhir.
- Bila pengguna ingin makalah sebagai PPT atau ringkasan, itu pekerjaan terpisah: gunakan isi makalah sebagai bahan.
