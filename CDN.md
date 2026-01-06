# CDN

content delivery network

`Origin server` atau server asal adalah komputer yang menghosting `versi asli file website` Anda. Di sisi lain, `edge atau server CDN` adalah komputer yang bisa `menyimpan file cache konten web` yang diambil dari origin server.

Misalnya, website UMKM yang hanya beroperasi di satu area geografis kecil tidak terlalu membutuhkan jaringan distribusi konten. `Pilih saja origin server yang ada di daerah tersebut`.

Zaman dulu, CDN hanya digunakan untuk file statis seperti js, html, css. Tapi saat ini bisa digunakan untuk `API dinamis`.

## Manfaat CDN

1. Mengurangi Penggunaan Bandwidth
2. 






Baik. Saya akan lanjut dengan **panduan praktis yang bisa langsung Anda terapkan** untuk menerapkan caching REST API dengan CDN, termasuk bagaimana invalidation-nya dilakukan.

Silakan pilih jika Anda ingin implementasi spesifik untuk stack tertentu.

---

# 1. Pola Caching REST API + CDN yang Direkomendasikan

## **A. Gunakan TTL (max-age) untuk data yang tidak sensitif**

Ini adalah baseline.

Header:

```
Cache-Control: public, max-age=60, s-maxage=300
```

Penjelasan:

* `public`: boleh di-cache CDN
* `max-age`: berapa lama client boleh cache
* `s-maxage`: berapa lama CDN boleh cache (server-side cache)

Biasanya:

* Client TTL kecil (10–60 detik)
* CDN TTL lebih besar (1–5 menit)

---

## **B. Gunakan ETag untuk validasi**

Tambahkan ETag di respons:

```
ETag: "abcdef123456"
```

Client/CDN akan mengirim:

```
If-None-Match: "abcdef123456"
```

Jika data belum berubah:

* Server membalas `304 Not Modified` (tanpa payload, sangat cepat)

ETag cocok kalau:

* Data berubah sering
* Tapi tidak ingin selalu purge CDN

---

## **C. Invalidation manual (jika perubahan harus real-time)**

Ada tiga tipe:

### 1) **Purge by URL**

Jika data per endpoint berubah:

* Purge URL tertentu saja
* Tidak mengganggu cache lainnya

### 2) **Purge by Tag (paling rapih)**

Beberapa CDN mendukung:

* Fastly: Surrogate Keys
* Cloudflare: Cache Tags
* Akamai: Cache Tags

Anda bisa memberi tag saat mengirim respons:

```
Cache-Tag: product, product-123
```

Saat produk berubah:

* Purge berdasarkan tag “product-123”
  → Seluruh cache untuk produk tersebut dibersihkan sekaligus.

### 3) **Purge All (jarang digunakan)**

Hanya dipakai jika deployment besar, schema berubah, atau ada bug cache.

---

# 2. Kapan Anda Harus Melakukan Invalidation?

Gunakan aturan ini:

| Kondisi                                    | Strategi           |
| ------------------------------------------ | ------------------ |
| Data jarang berubah                        | TTL (max-age) saja |
| Data sedang sering berubah                 | TTL kecil + ETag   |
| Data berubah penting (harga, stok, status) | Purge by URL/Tag   |
| Data harus realtime                        | Jangan di-cache    |

---

# 3. Contoh Alur Kerja Lengkap (Umum Dipakai di Production)

Misal endpoint:

```
GET /products/123
```

Alurnya:

1. CDN cache 5 menit (`s-maxage=300`)
2. Client cache 30 detik (`max-age=30`)
3. Respons menyertakan ETag
4. Jika harga produk berubah:

   * Backend memanggil API CDN: purge cache untuk `/products/123`
5. Request berikutnya:

   * CDN fetch ulang data
   * CDN cache versi baru
   * Client akan pakai versi baru

Hasilnya:

* Data update real-time saat diperlukan
* Beban server minimal
* CDN menangani 80–90% traffic GET

---

# 4. Kalau Anda ingin implementasi langsung

Saya bisa berikan:

* Contoh konfigurasi Cloudflare (cache rules + purge API)
* Contoh header caching yang aman untuk API
* Contoh implementasi ETag di Node.js/Express, Go, atau NestJS
* Desain arsitektur caching end-to-end untuk REST API Anda

Cukup sebutkan:

* Framework backend apa yang Anda pakai (misal Express, NestJS, Go Fiber, Laravel, dsb)
* CDN yang Anda gunakan (Cloudflare, CloudFront, Fastly, dsb)













Baik. Berikut adalah panduan **praktis, lengkap, dan production-ready** untuk menerapkan **caching + invalidation Cloudflare** pada REST API yang ditulis dengan **Golang**.

Struktur penjelasan:

1. Header caching yang direkomendasikan
2. Implementasi di Go (TTL + ETag)
3. Konfigurasi Cloudflare agar bisa cache REST API
4. Cara melakukan invalidation (purge) Cloudflare dari Go
5. Best practice arsitektur

---

# 1. Header Caching yang Direkomendasikan (untuk Cloudflare)

Untuk REST API:

```
Cache-Control: public, max-age=30, s-maxage=300
ETag: "<hash>"
```

Penjelasan:

* `max-age=30`: browser cache 30 detik
* `s-maxage=300`: Cloudflare cache 5 menit
* `ETag`: validasi ke origin jika data berubah

Ini pola umum dan aman untuk produksi.

---

# 2. Implementasi Go (TTL + ETag)

Contoh simple menggunakan `net/http`:

```go
package main

import (
	"crypto/sha1"
	"encoding/hex"
	"encoding/json"
	"net/http"
)

type Product struct {
	ID    string  `json:"id"`
	Price float64 `json:"price"`
}

func main() {
	http.HandleFunc("/products/123", ProductHandler)
	http.ListenAndServe(":8080", nil)
}

func ProductHandler(w http.ResponseWriter, r *http.Request) {
	product := Product{
		ID:    "123",
		Price: 10000,
	}

	// JSON encode
	jsonBytes, _ := json.Marshal(product)

	// Generate ETag
	hash := sha1.Sum(jsonBytes)
	etag := hex.EncodeToString(hash[:])

	// Check if client or Cloudflare already has same ETag
	if match := r.Header.Get("If-None-Match"); match == etag {
		w.WriteHeader(http.StatusNotModified)
		return
	}

	// Set caching headers
	w.Header().Set("Cache-Control", "public, max-age=30, s-maxage=300")
	w.Header().Set("ETag", etag)
	w.Header().Set("Content-Type", "application/json")

	w.Write(jsonBytes)
}
```

Kelebihan:

* Cloudflare akan cache respons 5 menit.
* Jika ada request ke origin, ETag akan mencegah payload besar dikirim ulang.
* Sangat ringan dan simple.

---

# 3. Konfigurasi Cloudflare

### **A. Aktifkan Cache untuk API Routes**

Di Cloudflare Dashboard:

1. Rules → Cache Rules
2. Create Rule

   * If URI Path matches `/products/*`
   * Then → Cache: Eligible for Cache
   * Edge TTL = 300 detik (atau override dengan header)

### **B. Izinkan Cloudflare Mengikuti Header Origin**

Konfigurasi:

* "Origin Cache Control": Respect Existing Headers
* "Cache By Device Type": Off
* "Query String Sort": On (opsional)

### **C. Cloudflare tidak akan cache POST, PUT, DELETE**

Itu normal dan sesuai best practice.

---

# 4. Implementasi Purge (Invalidation) Cloudflare dari Go

Gunakan Cloudflare API:

* Endpoint: `POST https://api.cloudflare.com/client/v4/zones/{zone_id}/purge_cache`

Contoh Go code:

```go
package cf

import (
	"bytes"
	"encoding/json"
	"fmt"
	"net/http"
)

type PurgeRequest struct {
	Files []string `json:"files"`
}

func PurgeCache(urls []string, zoneID, apiToken string) error {
	body, _ := json.Marshal(PurgeRequest{Files: urls})

	req, _ := http.NewRequest("POST",
		fmt.Sprintf("https://api.cloudflare.com/client/v4/zones/%s/purge_cache", zoneID),
		bytes.NewBuffer(body),
	)

	req.Header.Set("Authorization", "Bearer "+apiToken)
	req.Header.Set("Content-Type", "application/json")

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	if resp.StatusCode >= 300 {
		return fmt.Errorf("Cloudflare purge failed: %s", resp.Status)
	}

	return nil
}
```

Pemakaian:

```go
err := PurgeCache(
    []string{"https://api.example.com/products/123"},
    "YOUR_ZONE_ID",
    "YOUR_API_TOKEN",
)

if err != nil {
    fmt.Println("Failed purge:", err)
}
```

Gunakan purge ketika:

* Harga berubah
* Stok berubah
* Data reputasi tinggi harus update real-time

---

# 5. Best Practice Arsitektur (Golang + Cloudflare)

### **1. GET = boleh cache**

Gunakan TTL + ETag seperti contoh.

### **2. POST/PUT/PATCH/DELETE = tidak boleh cache**

Set:

```
Cache-Control: no-store
```

### **3. Set Cache Tags (opsional, best practice premium)**

Jika menggunakan Cloudflare Enterprise, Anda bisa set tag:

```
CF-Cache-Tag: product-123, product
```

Kemudian purge by tag:

* Sangat efisien
* Tidak harus purge per URL

### **4. Gunakan short TTL + purge untuk data cepat berubah**

Contoh:

* `max-age=10, s-maxage=30`
* Purge by URL ketika data berubah

### **5. Hindari cache untuk data per-user**

Contoh endpoint:

```
GET /me
GET /orders
```

Tambahkan:

```
Cache-Control: private, no-store
```

---

# Jika Anda ingin, saya bisa lanjut ke:

* Contoh lengkap project Go (folder structure + middleware caching)
* Contoh Cloudflare cache rules terbaik untuk API
* Desain caching untuk API yang memiliki banyak query parameter
* Cara menghindari stale data dalam distributed system

Cukup beri tahu kebutuhan Anda.
