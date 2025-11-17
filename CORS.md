# CORS

Cross-Origin Resource Sharing. Melindungi resource/response dari site tidak dikenal. Protecting BE from:

- `non-registered` ORIGINS(domain, protocol, or port).
- `HTTP` Methods (GET, POST, DELETE, ...)
- `HEADERS`

CORS melindungi akses browser dari `response`, bukan request. Sehingga sebenarnya proses masih dilakukan di server **KECUALI** request berupa `preflight` (request method OPTIONS).

## Preflight

A **preflight request** is an `OPTIONS` HTTP request that the browser sends **before** the actual request.
Its purpose is to ask the server:

> “Is it safe for this origin to send this type of request?”

If the server allows it, it responds with correct CORS headers.
Only then will the browser send the actual request.

### Browser Mengirim Simple Request jika

#### 1. HTTP method is one of

- `GET`
- `POST`
- `HEAD`

#### 2. Headers are strictly limited to

- `Accept`
- `Accept-Language`
- `Content-Language`
- `Content-Type` (but ONLY some values, see next)

#### 3. If `Content-Type` exists, it must be

- `text/plain`
- `application/x-www-form-urlencoded`
- `multipart/form-data`

#### 4. No custom headers

Examples of custom headers that trigger preflight:

- `Authorization`
- `X-API-Key`
- `X-Requested-With`
- Anything not above

#### 5. No credentials unless allowed (`withCredentials: true`)

If any of these rules fail → **preflight required**

### Example 1 (Authorization header)

```bash
POST /orders
Content-Type: application/json
Authorization: Bearer <token>
```

This triggers preflight because:

- Content-Type is JSON (not simple)
- Authorization header is custom

Browser sends:

```bash
OPTIONS /orders
Origin: https://frontend.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Authorization, Content-Type
```

If server responds properly:

```bash
Access-Control-Allow-Origin: https://frontend.com
Access-Control-Allow-Methods: POST
Access-Control-Allow-Headers: Authorization, Content-Type
```

Only then browser sends POST.

### Melindungi dari non-registered simple request

Because **simple requests do not require preflight**, even malicious sites can issue:

- `POST` form submissions (`application/x-www-form-urlencoded`)
- Auto-submitted HTML forms
- Basic GET requests

That's why CSRF can happen even with CORS.

**Ya, benar.**
Jika penyerang membuat situs lain yang mengirim **simple request (non-preflight)** ke server Anda, maka:

1. Browser tetap mengirim request ke server Anda

2. Server tetap mengeksekusi logic-nya

3. Browser hanya mencegah JavaScript di situs penyerang agar tidak bisa membaca response

Dengan kata lain:

> **CORS hanya melindungi response dari dibaca, bukan melindungi request dari dikirim.**

Inilah mengapa CORS **tidak** melindungi dari CSRF.
