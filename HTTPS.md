# HTTPS

add security layer (SSL/TLS) on your web apps

benefits:

- data enc menggunakan SSL/TLS
- SEO
- data integrity
- authentication, sertifikat server akan disesuaikan dengan domain ketika request terjadi.

sertifikat SSL/TLS. Sertifikat ini diterbitkan oleh Certificate Authority (CA), semacam lembaga terpercaya di internet (contohnya: DigiCert, Let's Encrypt).

1. Browser meminta sertifikat dari server.
2. Sertifikat itu berisi nama domain dan tanda tangan digital dari CA.
3. Browser memeriksa apakah tanda tangan itu valid dan cocok dengan domain yang diminta.

Inti singkat

Sertifikat X.509 berisi identitas yang diizinkan (nama domain/ IP) — Subject Alternative Name (SAN) adalah tempat utama. Saat klien menerima sertifikat, klien akan memastikan setidaknya satu entri SAN cocok dengan hostname yang diminta; jika tidak cocok, koneksi dianggap tidak aman.

Elemen sertifikat yang relevan

- Subject Alternative Name (SAN): daftar DNS names dan/atau IP addresses yang sertifikat berlaku untuknya. Ini adalah sumber kebenaran modern untuk pengecekan hostname.

- Common Name (CN): bagian dari Subject; dulu dipakai untuk domain, sekarang tidak boleh diandalkan karena standar modern mengharuskan SAN; beberapa implementasi fallback ke CN jika SAN tidak ada, tapi itu lama dan tidak dianjurkan.

- Wildcard: bisa muncul dalam SAN sebagai *.example.com. Aturan umum: wildcard hanya menggantikan satu label kiri (mis. `*.example.com` cocok untuk api.example.com, tapi tidak untuk deep.api.example.com). Wildcard tidak bisa menutupi TLD atau public suffix.

- IP dalam SAN: jika server diakses via IP literal (mis. <https://1.2.3.4>), sertifikat harus memiliki SAN bertipe IP Address yang persis sama.

Key Usage / Extended Key Usage: mengindikasikan sertifikat boleh dipakai untuk autentikasi server (TLS Web Server Authentication).
