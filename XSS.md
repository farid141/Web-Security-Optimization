# XSS

merupakan sebuah serangan dimana ada script tidak diinginkan yang dieksekusi di browser client

Jenis-Jenis XSS

1. Stored XSS: Script disimpan permanen di server (misalnya lewat komentar).
2. Reflected XSS: Script dikirim lewat URL atau form dan langsung dipantulkan oleh server. seperti query string
3. DOM-based XSS: Script muncul karena manipulasi DOM di sisi klien, seperti update DOM ketika form input berbahaya.

contoh dom-based:

Dari code memang ada penggunaan script, tetapi input yang di harapkan ternyata berbahaya.
`https://example.com/#<img src=x onerror=alert('xss')>`

```js
document.getElementById("output").innerHTML = location.hash.substring(1);
```

contoh reflected:

Dari code sebenarnya tidak ada menjalankan script sama sekali, tapi tercipta dari rendering server menerima param berupa `script`.

```php
<!-- search.php -->
<html>
  <head><title>Search</title></head>
  <body>
    <h2>Search Results for: <?php echo $_GET["query"]; ?></h2>
  </body>
</html>
```

url yang diakses attacker:

`https://example.com/search.php?query=<script>alert('XSS')</script>`

server akan mengambil parameter url tersebut dan merender kembali ke browser client:

```php
<h2>Search Results for: <script>alert('XSS')</script></h2>
```
