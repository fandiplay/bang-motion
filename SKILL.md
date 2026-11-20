---
name: bang-motion
description: Membangun motion graphic sinematik dan video explainer dengan Remotion, React, TypeScript, CSS, dan Three.js/React Three Fiber bila diperlukan. Cocok untuk promo, opener, intro, bumper, kinetic typography, dan explainer 16:9 atau 9:16.
license: MIT
metadata:
  author: Bang Tutorial
  author_url: https://youtube.com/bangtutorial
  version: "1.19.0"
  updated: "2026-11-20"
  homepage: https://github.com/bangtutorial/bang-motion
---

# Bang Motion — motion graphic web yang bukan PPT

**Edisi Remotion.** Gunakan dependensi proyek yang sudah tersedia; jangan menambahkan GSAP atau mengganti stack kecuali user memintanya.

Skill ini untuk AI coding agent apa pun. Isinya hukum desain dan cara kerja untuk motion graphic yang terasa seperti video, bukan presentasi. Aturan visual di bawah tetap berlaku; implementasinya menggunakan Remotion/React, bukan starter HTML + GSAP.

## Stack proyek dan batas dependensi

Proyek target memakai Remotion 4, React 19, TypeScript, `@remotion/cli`, `@remotion/google-fonts`, `@remotion/three`, `@react-three/fiber`, dan `three`. Ikuti versi yang tercantum di `package.json` dan jangan melakukan upgrade massal.

- Bangun komposisi sebagai React components yang didaftarkan pada Remotion `Root`/`Composition` (ikuti pola entry point yang sudah ada di proyek). Gunakan `useCurrentFrame()` dan `useVideoConfig()` sebagai sumber waktu dan dimensi.
- Animasi DOM/CSS: turunkan semua nilai dari frame; gunakan `interpolate()` atau `spring()` Remotion, transform CSS, dan komponen/primitive Remotion. Jangan menambahkan GSAP untuk timeline.
- Audio dan urutan: gunakan komponen Remotion seperti `Sequence`, `Audio`, `OffthreadVideo`/`Video` sesuai kebutuhan dan API versi proyek. Sinkronkan semuanya dalam frame.
- 3D bersifat opsional. Pakai `@remotion/three` bersama R3F/Three hanya bila konsep benar-benar membutuhkan 3D. Untuk adegan 2D, React + CSS lebih sederhana dan lebih mudah dirender.
- `@remotion/google-fonts` dapat menyediakan font. Pastikan font selesai dimuat sebelum render (gunakan mekanisme `delayRender()`/`continueRender()` yang sesuai), dan sediakan fallback.
- `@google/genai` bukan keharusan untuk animasi. Jika dipakai untuk membuat aset, jangan pernah menaruh API key di komposisi, browser, atau file yang akan dibagikan; gunakan proses aset lokal/aman dan simpan hasilnya sebagai aset proyek.
- Jangan memasang paket baru tanpa kebutuhan dan persetujuan. `allowScripts.esbuild: false` berarti jangan mengandalkan lifecycle script esbuild untuk menyiapkan proyek.

### Pratinjau dan render

Remotion memerlukan Node sesuai `engines` di `package.json`; deliverable adalah proyek Remotion yang bisa dipratinjau di Remotion Studio dan dirender, bukan satu `index.html` yang dibuka dengan klik dua kali. Gunakan script proyek yang tersedia:

- `npm run dev` — Remotion Studio.
- `npm run lint` — ESLint dan TypeScript.
- `npm run build` — bundle Remotion sesuai script proyek; ini bukan otomatis ekspor MP4.
- Ekspor video: gunakan CLI Remotion dengan composition ID dan entry point proyek yang benar, misalnya `npx remotion render <entry-point> <composition-id> out/video.mp4`. Jangan menebak ID atau entry point; periksa konfigurasi dan source terlebih dahulu.

Jangan render semua frame hanya untuk pengecekan rutin. Jalankan Studio, periksa frame kunci/segmen, lalu render MP4 hanya bila diminta atau dibutuhkan untuk memvalidasi output. Pastikan codec, browser/headless dependencies, dan aset lokal tersedia di mesin render. Tidak ada ffmpeg-specific workflow yang diwajibkan oleh skill ini; ikuti pipeline Remotion proyek.

## Player, durasi, dan format

Video Remotion tidak autoplay/loop sebagai halaman web. Atur `fps`, `durationInFrames`, lebar, tinggi, dan composition ID secara eksplisit di Composition. Default rasio 16:9; gunakan 9:16 hanya bila diminta. Durasi mengikuti brief dan audio, bukan jumlah adegan tetap. Kontrol preview ditangani Remotion Studio; jangan membuat player tambahan di dalam frame kecuali memang bagian dari desain.

## Hukum #1 — Ini video, bukan slide

Penyakit paling umum: adegan seperti slide—judul, subjudul, paragraf, lalu badge. Hindari.

1. Adegan harus hidup: aksi objek, perubahan dunia, gerak kamera, atau transformasi visual. Jangan hanya memudar-munculkan panel berisi teks.
2. Pertahankan benang merah visual: satu subjek/aksi berkelanjutan atau satu dunia yang ditelusuri kamera. Satu gambar dan satu kalimat yang berganti tiap adegan tetap slideshow, meski ada zoom.
3. Maksimal dua tingkat teks per adegan. Untuk opener, satu kalimat pendek dan maksimal satu objek utama; tidak ada kicker, paragraf, atau tumpukan badge kecuali merupakan UI produk yang didemokan.
4. Informasi baru harus memicu respons visual yang relevan; tidak setiap kalimat perlu menjadi adegan baru.
5. Variasikan framing, komposisi, ritme, dan transisi. Ken Burns pada foto bukan satu-satunya gerak. Kamera bergerak karena mengikuti atau mengungkap aksi.
6. Explainer harus dimulai dari starter Remotion yang sesuai bila tersedia; jangan menyalin starter HTML lama menjadi deliverable Remotion. Starter HTML di `assets/` hanya referensi desain/gerak, bukan template implementasi untuk proyek ini.
7. Hindari miniatur/peta yang membuat subjek terlalu kecil. Dunia dan aset harus terbaca pada ukuran output.
8. Opener/promo perlu konsep dan struktur yang lahir dari produknya, bukan urutan baku hook → fitur → janji → logo → CTA.

Jika membaca `references/anti-ppt.md`, `references/opener-konsep.md`, atau dokumen desain lain, gunakan sebagai prinsip visual/konseptual. Terjemahkan resep teknis GSAP/HTML ke API Remotion; jangan menyalin kode yang bertentangan dengan stack proyek.

## Hukum #2 — Deterministik atau mati

Render Remotion dapat meminta frame secara acak dan urutannya tidak dijamin. Setiap tampilan harus merupakan fungsi murni dari frame, props, dan aset tetap.

- Gunakan `useCurrentFrame()`; jangan membaca `Date.now()`, `performance.now()`, `Math.random()`, atau waktu wall-clock dalam jalur render.
- Jangan mengakumulasi posisi/kecepatan melalui `useEffect`, state yang berubah per frame, atau delta waktu. Hitung nilai langsung dari frame.
- Untuk variasi pseudo-acak, gunakan seed deterministik berbasis indeks/seed tetap, bukan `Math.random()` saat render.
- Bila memakai R3F/Three, posisi, rotasi, kamera, shader uniform, dan material harus dapat dihitung ulang untuk frame yang diminta. Hindari simulasi yang hasilnya bergantung pada urutan frame; kirim frame dari komposisi ke scene dan hitung state dari frame. `useFrame`/delta R3F tidak boleh menjadi akumulator non-deterministik.
- Jangan memakai network fetch, API generatif, atau sumber waktu nyata saat merender komposisi. Siapkan aset terlebih dahulu dan jadikan render offline/reproducible.

## Hukum #3 — Gaya lahir dari tema, bukan starter atau referensi

Sebelum menulis kode, buat style brief dan rundown, lalu minta persetujuan user. Untuk pekerjaan yang belum punya detail penting, tanyakan dalam satu pesan: jenis video, rasio, durasi, audio/VO, dan pilihan gaya explainer bila relevan.

Style brief mencakup:
- tema/produk dan tiga kata sifat yang ingin ditinggalkan;
- konsep, jumlah adegan, objek/aksi utama tiap adegan, pembuka, dan penutup;
- palet dari brand/tema dengan hex dan sumbernya; warna turunan diberi alasan;
- font display berkarakter dan font pendamping (jangan otomatis memilih Poppins, Inter, Roboto, Montserrat, atau Arial);
- pilihan penekanan teks, bahasa gerak latar, dan permukaan latar berlapis;
- keluarga render objek (flat, clay, glossy, kaca, isometrik, realistis, dll.);
- transisi yang memang diperlukan dan satu momen istimewa;
- bila memakai foto/video: sumber aset dan daftar shot, disepakati dengan user.

Starter hanya arsitektur, bukan gaya. Jangan mewarisi palet, font, latar, atau urutan contoh begitu saja. Dari referensi video, ambil ritme, energi, durasi shot, hierarki atensi, dan prinsip gerak—bukan salin kulit, komposisi, kalimat, atau gimmick khasnya.

Jika proyek/proyek sebelumnya tersedia dalam konteks, proyek baru harus berbeda setidaknya tiga dari empat aspek: palet, font, bahasa latar, dan tanda tangan gerak. Jangan membuka folder proyek lama hanya untuk menyalin gayanya.

Latar tidak otomatis gelap dan tidak boleh terasa statis. Pilih bahasa yang cocok: editorial terang, color-block, kertas/cetak, retro, pastel, mewah, grainy gradient, sinematik gelap, atau paduan yang disengaja. Gerak dan pergantian warna harus punya alasan/pemicu visual.

## Hukum #4 — Teks hidup, latar tunduk

- Utamakan sentence case dan bobot 500–600; kapital tebal hanya untuk satu momen penekanan.
- Frasa pendek biasanya tanpa titik. Tanda baca harus punya tujuan dan konsisten.
- Sorotan kata opsional; jika dipakai, pilih satu bahasa bentuk untuk satu video dan turunkan ornamen dari brand.
- Dua adegan berurutan jangan memakai ukuran, arah masuk, dan posisi teks yang sama.
- Pastikan area di belakang teks punya kontras tinggi: cukup gelap atau cukup terang, tidak ramai sedang. Elemen dekor di area teks redup dan tidak mengalahkan subjek.
- Gunakan motion blur secara terkendali bila meningkatkan gerak; jangan membuat teks sulit dibaca.

## Alur kerja Remotion

1. **Kenali jenis pekerjaan.** Opener/promo, bumper, intro/outro, kinetic typography, title/lower third, atau explainer. Untuk explainer, baca `references/explainer.md` dan pilih gaya/rasio/durasi/suara sebelum produksi.
2. **Tulis style brief dan rundown** (`detik | adegan | kalimat | visual`) sebelum kode. Untuk promo, tentukan tiga kandidat konsep dan pilih satu yang paling dituntut produk. Minta persetujuan user.
3. **Periksa proyek yang ada.** Baca `package.json`, struktur `src`, entry point Remotion, root/composition, aset, ukuran/fps, dan skrip lint/build. Jangan mengganti konfigurasi atau mengasumsikan path.
4. **Rancang komposisi.** Buat Composition dengan ukuran, fps, dan durasi eksplisit. Gunakan `Sequence`/`Series` atau interpolasi frame untuk mengatur waktu; komponen reusable untuk objek/adegan; properti CSS/React untuk 2D. Jangan memecah tiap adegan menjadi halaman yang hanya fade.
5. **Bangun adegan satu per satu.** Setiap tween/interpolasi harus dibatasi pada frame yang jelas, dengan `extrapolateLeft`/`extrapolateRight` yang sesuai agar nilai tidak meluber. Gunakan `spring()` untuk gerak fisik yang perlu, bukan sebagai efek otomatis pada semua elemen. Gunakan Three/R3F hanya untuk kebutuhan visual yang nyata.
6. **Audio dan font.** Tambahkan audio sebagai aset lokal, letakkan pada timeline frame, dan cek titik masuk/keluar serta sinkronisasi. Tunggu font/aset yang dimuat secara asynchronous menggunakan mekanisme Remotion sebelum render. Jangan mengandalkan jaringan yang berubah-ubah saat render.
7. **Verifikasi visual.** Jalankan Studio dan periksa 6–20 frame kunci: awal komposisi/adegan, teks muncul, aksi penting, transisi, dan akhir. Periksa crop, keterbacaan pada ukuran target, tumpang tindih, timing, kontinuitas dunia, dan kemungkinan slideshow. Jalankan lint/TypeScript. Perbaiki lalu tinjau ulang frame yang sama.
8. **Serahkan.** Sebutkan file/komposisi yang dibuat, perintah preview dan render yang benar untuk proyek, output render bila diminta, aset yang perlu disertakan, serta asumsi/batasan. Jangan mengklaim MP4 berhasil dibuat bila belum dirender dan diperiksa.
9. **Khusus explainer.** Sertakan naskah VO lengkap dan ajak user merekam/menghasilkan audio lalu mengirimnya untuk sinkronisasi. Simpan `vo-script.md` di folder proyek bila sesuai struktur proyek.

## Checklist sebelum menyerahkan

- [ ] Style brief dan rundown disetujui sebelum implementasi.
- [ ] Kode memakai Remotion/React/TypeScript proyek; tidak ada GSAP atau dependency baru tanpa alasan/persetujuan.
- [ ] Composition memiliki fps, dimensi, durasi, dan ID yang eksplisit dan sesuai brief.
- [ ] Semua gerak deterministik dari frame/props; jalur render bebas waktu sistem, `Math.random()`, network request, dan akumulasi delta.
- [ ] 3D (bila ada) dapat menghasilkan state yang sama ketika frame diminta dalam urutan berbeda.
- [ ] Tidak ada pola slide: adegan hidup, ada benang merah, teks maksimal dua tingkat, komposisi dan gerak bervariasi.
- [ ] Gaya diturunkan dari tema/brand; bukan placeholder starter, salinan referensi, atau kulit proyek sebelumnya.
- [ ] Area teks terbaca; ukuran dan kontras diuji pada resolusi output dan perangkat kecil.
- [ ] Audio, font, dan aset dimuat/ditempatkan dengan cara yang aman untuk rendering.
- [ ] Frame kunci diperiksa secara visual di Studio; lint dan TypeScript dijalankan jika lingkungan memungkinkan.
- [ ] Perintah render memakai entry point dan composition ID aktual, bukan tebakan; hasil render tidak diklaim tanpa verifikasi.
- [ ] Explainer menyertakan naskah VO dan permintaan audio untuk sinkronisasi.

## Isi paket referensi

Dokumen berikut tetap berguna untuk keputusan visual dan editorial. Kode di dalamnya mungkin memakai HTML/GSAP dan harus diterjemahkan, bukan dijalankan langsung sebagai implementasi Remotion.

| Berkas | Kapan dibaca |
|---|---|
| `references/anti-ppt.md` | Sebelum menyusun adegan |
| `references/opener-konsep.md` | Sebelum memilih konsep opener/promo |
| `references/explainer.md` | Untuk explainer: gaya, VO, fakta, sudut pandang |
| `references/kartun-panggung.md` | Untuk explainer kartun panggung |
| `references/techniques.md` | Ide gerak, framing, transisi, dan permukaan visual; adaptasikan implementasi |
| `references/architecture.md` | Arsitektur starter lama sebagai konteks, bukan arsitektur Remotion |
| `references/ae-bridge-higgsfield.md` | Hanya bila user meminta workflow After Effects melalui bridge |
| `references/roadmap.md` | Bila mengembangkan skill ini |
| `assets/starter-*.html` | Referensi visual/ide saja; bukan starter kode proyek Remotion |
| `scripts/vo-pauses.html` | Alat opsional berbasis browser untuk membantu menemukan jeda VO |
| `scripts/ae/bridge/*` | Hanya workflow After Effects/bridge, tidak diperlukan untuk Remotion |
