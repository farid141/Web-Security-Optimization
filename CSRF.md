# CSRF

Cross-Site Request Forgery. Memanfaatkan celah keamanan session user login yang disimpan di cookies. Hal ini dikarenakan sifat cookies yang `otomatis dikirim ke server`, sehingga laman jahat melakukan request ke endpoint server dengan `membawa cookies session` tersebut.

Berbeda dengan CSRF Token yang membutuhkan token dikirim sebagai `header`.

## 1. Apa yang terjadi jika request masuk kategori “simple request”?

Contoh simple request:

```bash
POST /api/transfer
Content-Type: application/x-www-form-urlencoded
```

Browser melakukan:

1. Mengirim request langsung ke server (TANPA preflight).
2. Menyertakan semua cookie yang berkaitan (jika user login).
3. Server menerima request seperti biasa.
4. Server menjalankan controller/service/DB.
5. Server mengembalikan response.
6. **Browser memblok JavaScript penyerang dari membaca response**.

Tetapi logic server sudah jalan.

## 2. Contoh Attack (CSRF)

Misalkan user login di `bank.com`.
Lalu user membuka `evil.com`.

`evil.com` mengirim hidden form:

```html
<form action="https://bank.com/transfer" method="POST">
  <input type="hidden" name="to" value="attacker">
  <input type="hidden" name="amount" value="5000000">
</form>

<script>
  document.forms[0].submit();
</script>
```

Karena aturan simple request:

- Ini adalah **POST x-www-form-urlencoded** (simple request).
- Browser **tidak perlu preflight**.
- Browser otomatis mengirim **cookie bank.com** (session user).
- Server bank.com memproses transfer.
- Response tidak bisa dibaca oleh evil.com, tetapi **damage sudah terjadi**.

Jadi CORS **tidak** menghentikan request dan tidak melindungi user.

## 3. Mengapa ini terjadi?

Karena alasan historis:

HTML form submission sudah ada jauh sebelum CORS diciptakan, dan harus tetap kompatibel.

Browser tidak boleh memblokir form submission (GET/POST basic) karena itu akan merusak banyak aplikasi web lama.

## 4. Jadi apa perlunya CORS?

CORS adalah untuk mencegah website lain:

- membaca data API Anda
- mengakses API Anda menggunakan **custom headers** atau JSON body
- mengakses API Anda dengan token di header
- melakukan operasi dengan script yang kompleks

Tetapi CORS **bukan** untuk mencegah request sederhana dikirim.

## 5. Bagaimana mencegah server menjalankan logic ketika diserang?

**Gunakan CSRF protection** jika Anda menggunakan cookies:

1. CSRF token
2. SameSite cookies
3. Double-submit cookie
4. Referer/Origin checking

Jika Anda menggunakan token-based auth dalam header (JWT di Authorization header):

- Browser **tidak bisa** mengirim header Authorization dari situs penyerang
- Maka **CSRF tidak mungkin terjadi**

## 6. Kesimpulan

**Benar:**
Penyerang bisa mengirim simple request dan server Anda tetap memproses logic.

**CORS tidak menghentikan request. CORS hanya menghentikan response untuk dibaca.**

Karena itu, **CORS + token di header = aman dari CSRF.**
Tetapi **CORS + cookies = tidak aman dari CSRF** (butuh CSRF token).
