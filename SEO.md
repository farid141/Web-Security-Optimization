# SEO

SEO (Search Engine Optimization) pada dasarnya adalah **cara membuat halaman webmu mudah ditemukan, dipahami, dan dinilai relevan oleh mesin pencari seperti Google**. Tujuan akhirnya: ketika orang mencari sesuatu yang berkaitan, situsmu muncul di hasil pencarian — idealnya di halaman pertama.

Mari kita bedah mekanismenya, terutama dalam konteks situs seperti `mypage.com` yang punya beberapa halaman (`/home`, `/about`, dll):

## 1. **Crawling (perayapan)**

Mesin pencari menggunakan bot (misalnya *Googlebot*) untuk “menjelajahi” web.
Bot ini mulai dari satu halaman — misalnya `mypage.com/home` — lalu mengikuti semua tautan (link) di dalamnya untuk menemukan halaman lain seperti `/about`, `/contact`, dan seterusnya.

Kalau kamu punya `sitemap.xml`, itu seperti peta yang mempermudah bot mengetahui struktur situsmu.
Jika tidak ada tautan internal yang `baik`, halaman tertentu bisa “tersembunyi” dan tidak terindeks.

### 1. **Bot memang “menjelajahi” web seperti penjelajah sungguhan**

Googlebot dan teman-temannya **tidak hanya mengunjungi satu halaman** lalu berhenti.
Ia memulai dari *seed URLs* — sekumpulan alamat web yang sudah diketahui (misalnya dari link di web lain, sitemap, atau domain baru yang ditemukan lewat DNS) — lalu **mengikuti semua tautan yang bisa ditemukan** di dalam halaman itu.

Kamu bisa bayangkan seperti laba-laba yang menjelajah jaringnya:

* Ia datang ke `mypage.com/` (homepage).
* Di dalamnya, ia melihat link `<a href="/about">About</a>`.
* Ia mencatat dan kemudian mengunjungi `mypage.com/about`.
* Dari situ, ia mungkin menemukan `/contact`, `/services`, dan seterusnya.

Itu sebabnya mesin pencari disebut *web crawler* — karena memang ia “merayapi” tautan satu per satu.

### 2. **Setiap halaman adalah entitas tersendiri**

Jika bot berhasil mengakses `mypage.com/about`, halaman itu **akan dianggap terpisah** dari root (`mypage.com/`).
Masing-masing bisa:

* Memiliki *title*, *description*, dan konten unik.
* Diindeks dan dirangking secara independen untuk kata kunci berbeda.

Jadi ya — halaman `/about`, `/home`, `/contact`, dsb semuanya *bisa* masuk indeks Google **asalkan bisa ditemukan dan tidak diblokir** (misalnya lewat `robots.txt` atau tag `<meta name="robots" content="noindex">`).

### 3. **Bagaimana bot tahu harus mengikuti tautan?**

Bot membaca seluruh HTML — bukan seperti browser, tapi lebih seperti *text parser*.
Ia mencari elemen-elemen `<a href="...">`, `<link rel="canonical" ...>`, `<img src="...">`, dan bahkan beberapa script jika terdeteksi ada *routing logic* (meskipun ini terbatas untuk situs SPA/React).

Kalau link-nya statis dan bisa dibaca (contoh: `<a href="/about">About</a>`), bot akan tahu ada halaman baru yang bisa dikunjungi.
Tapi kalau link dibuat lewat JavaScript dinamis yang baru muncul *setelah event klik*, bot mungkin **tidak akan menemukannya**, kecuali kamu sudah pakai prerendering atau SSR.

### 4. **Bagaimana bot tahu mana yang utama (root vs child)?**

Bot tidak membuat asumsi seperti “root pasti utama”.
Ia menggunakan banyak sinyal:

* Jumlah backlink dari situs lain ke halaman itu.
* Internal link (berapa banyak halaman lain di situsmu yang menautkan ke halaman tersebut).
* *Sitemap* yang kamu kirim ke Google Search Console.

Kalau `/about` jarang ditautkan, bisa saja terindeks tapi punya “nilai” lebih kecil dibanding homepage.

### 5. **Apakah semua halaman diindeks otomatis?**

Tidak sepenuhnya otomatis. Ada 3 kemungkinan:

1. **Terindeks penuh** — bot menemukan, membaca, dan menambahkan ke indeks.
2. **Ditemukan tapi belum diindeks** — bot tahu halaman ada, tapi menunggu sinyal relevansi.
3. **Tidak ditemukan sama sekali** — karena link-nya tersembunyi, atau terhalang `robots.txt` atau JavaScript.

Kalau kamu ingin memastikan semua halaman ikut diindeks, kamu bisa bantu bot dengan:

* Membuat *sitemap.xml*.
* Menyediakan navigasi internal yang jelas.
* Tidak menghalangi crawling lewat `robots.txt`.
* Menambahkan situsmu ke Google Search Console.

Jadi, singkatnya:

> Bot tidak berhenti di `mypage.com/`, tapi berjalan mengikuti semua tautan internal yang bisa dibaca dan mengindeks setiap halaman yang ditemukannya — selama tidak diblokir.

## 2. **Indexing (pengindeksan)**

Setelah halaman ditemukan, bot akan **menganalisis isi halaman** — teks, tag HTML, gambar, meta description, dan struktur data lainnya — lalu menyimpannya di indeks Google, semacam perpustakaan raksasa.

Contoh:

* `/about` mungkin dianggap relevan dengan kata kunci “tentang kami” atau “profil perusahaan”.
* `/home` mungkin terkait dengan “jasa pengembangan IoT” kalau itu isi utamanya.

Agar indexing berjalan baik:

* Gunakan **tag HTML semantik** (`<h1>`, `<h2>`, `<p>`).
* Isi **meta tags** seperti `<title>` dan `<meta name="description">`. Dalam react ada fitur `meta tag generation`
* Pastikan halaman bisa diakses tanpa login atau JavaScript berat yang menghalangi konten muncul.

## 3. **Ranking (peringkat)**

Setelah halaman diindeks, mesin pencari menentukan **peringkat** berdasarkan ratusan faktor, termasuk:

* **Relevansi kata kunci**: Apakah isi halaman cocok dengan kata yang dicari pengguna?
* **Kualitas konten**: Apakah kontennya orisinal, berguna, dan terpercaya?
* **Backlink**: Apakah situs lain yang kredibel menaut ke halamanmu?
* **Kecepatan dan mobile-friendliness**: Apakah situsmu cepat dan responsif di semua perangkat?
* **User engagement**: Apakah orang betah membaca halamanmu atau cepat pergi?

## 4. **Halaman-halaman berbeda punya peran berbeda**

Misalnya:

* `/home` adalah “landing” utama — penting untuk kata kunci paling umum.
* `/about` bisa dioptimalkan untuk kata kunci seperti “tentang [nama perusahaan]” atau “profil [nama brand]”.
* `/services` bisa ditargetkan untuk layanan spesifik seperti “pengembangan sistem IoT”.
* `/blog/...` bisa untuk membidik kata kunci long-tail seperti “cara membuat dashboard IoT”.

Setiap halaman bisa dioptimalkan secara individual tapi tetap saling terhubung lewat *internal links* yang logis.

## 5. **Teknis tambahan (on-page dan off-page SEO)**

* **On-page**: struktur HTML, heading, URL rapi (`mypage.com/about` lebih baik daripada `mypage.com/page?id=123`), gambar dengan alt text, dsb.
* **Off-page**: backlink, reputasi domain, dan aktivitas sosial.
* **Technical SEO**: sitemap, robots.txt, HTTPS, kecepatan, dan *structured data* (misalnya JSON-LD schema untuk menandai profil organisasi).
