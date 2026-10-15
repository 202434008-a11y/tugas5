# Laporan Praktikum & Tugas Minggu 5: CRUD Resource Utama (Laravel API)

---

## 1. Identitas & Metadata Tugas

- **Mata Kuliah:** Praktikum Pemrograman Framework / Backend Web
- **Tugas:** Tugas 5 — CRUD Resource Utama pada Project Laravel
- **Resource Utama:** `Book` (Katalog Buku)
- **Framework:** Laravel 11.x (PHP 8.2+)
- **Dokumentasi File:** `docs/minggu-5.md`
- **Postman Collection:** `postman/tugas5_collection.json`
- **Repository GitHub:** [Tautan Repository GitHub Anda di sini]
- **Commit Terkait:** [Tautan Commit Hash di sini]

---

## 2. Tujuan

1. Mengimplementasikan operasi CRUD (Create, Read, Update, Delete) secara penuh pada resource utama `Book`.
2. Menerapkan validasi input data request menggunakan Laravel **Form Request** (`StoreBookRequest` dan `UpdateBookRequest`).
3. Menstandarkan format keluaran JSON menggunakan Laravel **API Resource** (`BookResource`).
4. Menyediakan penanganan response yang konsisten, baik untuk **Success Response** (HTTP status `200`, `201`, `204`) maupun **Error Response** (HTTP status `404`, `422`).
5. Menguji seluruh endpoint API menggunakan Postman dan mendokumentasikan bukti pengujian (evidence).
6. Menjaga keamanan aplikasi tanpa menyertakan secret key, token, kredensial, maupun file `.env`.

---

## 3. Ringkasan Perubahan dari Minggu Sebelumnya

Pada Minggu 4, API masih bersifat *Read-Only* (`GET /api/books` dan `GET /api/books/{id}`) dengan mengembalikan model Eloquent langsung tanpa standarisasi resource layer, serta pesan 404 yang belum terstruktur rapi.

Perubahan utama yang diterapkan pada Minggu 5:
1. **Routing:** Menambahkan rute lengkap RESTful melalui `Route::apiResource('books', BookController::class)` pada `routes/api.php` sehingga mendukung method `POST`, `PUT/PATCH`, dan `DELETE`.
2. **Form Request Validation:** 
   - Membuat `StoreBookRequest` untuk memvalidasi pembuatan buku baru (wajib mengisi `title`, `isbn` unik 13 karakter, dan opsi `available` bertipe boolean).
   - Membuat `UpdateBookRequest` untuk pembaruan buku (mendukung validasi opsional `sometimes` dan pengecualian ID buku pada aturan unik ISBN via `Rule::unique()->ignore($bookId)`).
   - Menambahkan pesan validasi kustom berbahasa Indonesia.
3. **API Resource:** 
   - Menggunakan `BookResource` (`App\Http\Resources\BookResource`) untuk memformat output JSON: memetakan tipe data `available` ke boolean murni (`true`/`false`), memformat `created_at` ke standar ISO 8601 string, dan menyembunyikan kolom internal yang tidak diperlukan oleh consumer API.
4. **Error Handling & Controller Consistency:** 
   - Controller menangani entitas yang tidak ditemukan (`notFound()`) dengan struktur JSON standar `{ "message": "Book not found", "errors": null }` dengan status code `404 Not Found`.
   - Mengonfigurasi `bootstrap/app.php` dengan `shouldRenderJsonWhen` untuk menjamin seluruh error validasi (`ValidationException`) selalu dikembalikan dalam bentuk JSON berseri status `422 Unprocessable Content`.

---

## 4. API Contract & Spesifikasi Endpoint

### Ringkasan Endpoint

| No | Method | Endpoint | Request Body | Status Code Sukses | Deskripsi |
|---|---|---|---|---|---|
| 1 | `GET` | `/api/books` | *None* | `200 OK` | Mengambil seluruh daftar buku |
| 2 | `POST` | `/api/books` | JSON Body | `201 Created` | Membuat buku baru |
| 3 | `GET` | `/api/books/{id}` | *None* | `200 OK` | Mengambil detail buku berdasarkan ID |
| 4 | `PUT` | `/api/books/{id}` | JSON Body | `200 OK` | Memperbarui data buku |
| 5 | `DELETE` | `/api/books/{id}` | *None* | `204 No Content` | Menghapus buku berdasarkan ID |

### Header Standar
Setiap request wajib menyertakan header berikut:
```http
Accept: application/json
Content-Type: application/json
```

---

### Detail Skema Request & Response

#### 1. GET `/api/books` (List Books)
- **Status:** `200 OK`
- **Response Body:**
```json
{
  "data": [
    {
      "id": 1,
      "title": "Clean Code",
      "isbn": "9780132350884",
      "available": true,
      "created_at": "2026-10-08T07:05:47.000000Z"
    },
    {
      "id": 2,
      "title": "The Pragmatic Programmer",
      "isbn": "9780135957059",
      "available": true,
      "created_at": "2026-10-08T07:05:47.000000Z"
    }
  ]
}
```

#### 2. POST `/api/books` (Create Book)
- **Request Body:**
```json
{
  "title": "Refactoring: Improving the Design of Existing Code",
  "isbn": "9780201485677",
  "available": true
}
```
- **Status:** `201 Created`
- **Response Body:**
```json
{
  "data": {
    "id": 3,
    "title": "Refactoring: Improving the Design of Existing Code",
    "isbn": "9780201485677",
    "available": true,
    "created_at": "2026-10-15T06:10:00.000000Z"
  }
}
```

#### 3. GET `/api/books/{id}` (Show Book)
- **Status:** `200 OK`
- **Response Body:**
```json
{
  "data": {
    "id": 1,
    "title": "Clean Code",
    "isbn": "9780132350884",
    "available": true,
    "created_at": "2026-10-08T07:05:47.000000Z"
  }
}
```

#### 4. PUT `/api/books/{id}` (Update Book)
- **Request Body:**
```json
{
  "title": "Clean Code: A Handbook of Agile Software Craftsmanship",
  "isbn": "9780132350884",
  "available": false
}
```
- **Status:** `200 OK`
- **Response Body:**
```json
{
  "data": {
    "id": 1,
    "title": "Clean Code: A Handbook of Agile Software Craftsmanship",
    "isbn": "9780132350884",
    "available": false,
    "created_at": "2026-10-08T07:05:47.000000Z"
  }
}
```

#### 5. DELETE `/api/books/{id}` (Delete Book)
- **Status:** `204 No Content`
- **Response Body:** *(Empty body)*

---

## 5. Bukti Pengujian (Evidence)

Pengujian dilakukan menggunakan Postman dengan skenario pengujian menyeluruh pada server lokal (`http://127.0.0.1:8000`).

### 1. Bukti Success Response

#### A. Create Book (`POST /api/books`) — 201 Created
- **URL:** `POST http://127.0.0.1:8000/api/books`
- **Headers:** `Accept: application/json`, `Content-Type: application/json`
- **Payload:**
```json
{
    "title": "Refactoring: Improving the Design of Existing Code",
    "isbn": "9780201485677",
    "available": true
}
```
- **Response Status:** `201 Created`
- **Response Body:**
```json
{
    "data": {
        "id": 3,
        "title": "Refactoring: Improving the Design of Existing Code",
        "isbn": "9780201485677",
        "available": true,
        "created_at": "2026-10-15T06:10:00.000000Z"
    }
}
```

#### B. Show Book (`GET /api/books/1`) — 200 OK
- **URL:** `GET http://127.0.0.1:8000/api/books/1`
- **Response Status:** `200 OK`
- **Response Body:**
```json
{
    "data": {
        "id": 1,
        "title": "Clean Code",
        "isbn": "9780132350884",
        "available": true,
        "created_at": "2026-10-08T07:05:47.000000Z"
    }
}
```

#### C. Update Book (`PUT /api/books/1`) — 200 OK
- **URL:** `PUT http://127.0.0.1:8000/api/books/1`
- **Payload:**
```json
{
    "title": "Clean Code: A Handbook of Agile Software Craftsmanship",
    "isbn": "9780132350884",
    "available": false
}
```
- **Response Status:** `200 OK`
- **Response Body:**
```json
{
    "data": {
        "id": 1,
        "title": "Clean Code: A Handbook of Agile Software Craftsmanship",
        "isbn": "9780132350884",
        "available": false,
        "created_at": "2026-10-08T07:05:47.000000Z"
    }
}
```

#### D. Delete Book (`DELETE /api/books/2`) — 204 No Content
- **URL:** `DELETE http://127.0.0.1:8000/api/books/2`
- **Response Status:** `204 No Content`
- **Response Body:** *(Kosong / null)*

---

## 6. Skenario Pengujian Error (Error Cases)

### Kasus Error 1: Resource Tidak Ditemukan (`404 Not Found`)
- **Request:** `GET http://127.0.0.1:8000/api/books/999999`
- **Headers:** `Accept: application/json`
- **Status:** `404 Not Found`
- **Response Body:**
```json
{
    "message": "Book not found",
    "errors": null
}
```
*Analisis:* Ketika model dengan ID yang diminta tidak ada di database, method `notFound()` pada controller memberikan response JSON dengan status HTTP 404 dan pesan yang informatif.

### Kasus Error 2: Validasi Gagal Saat Pembuatan (`422 Unprocessable Content`)
- **Request:** `POST http://127.0.0.1:8000/api/books`
- **Headers:** `Accept: application/json`, `Content-Type: application/json`
- **Payload:**
```json
{
    "title": "",
    "isbn": "123"
}
```
- **Status:** `422 Unprocessable Content`
- **Response Body:**
```json
{
    "message": "Title wajib diisi. (and 1 more error)",
    "errors": {
        "title": [
            "Title wajib diisi."
        ],
        "isbn": [
            "ISBN harus terdiri dari 13 karakter."
        ]
    }
}
```
*Analisis:* `StoreBookRequest` memvalidasi bahwa `title` wajib diisi (`required`) dan `isbn` harus memiliki panjang tepat 13 karakter (`size:13`). Jika aturan dilanggar, Laravel secara otomatis mengembalikan pesan error dalam format JSON dengan status HTTP 422.

### Kasus Error 3: Duplikasi ISBN (`422 Unprocessable Content`)
- **Request:** `POST http://127.0.0.1:8000/api/books`
- **Headers:** `Accept: application/json`, `Content-Type: application/json`
- **Payload:**
```json
{
    "title": "Buku Duplikat",
    "isbn": "9780132350884"
}
```
- **Status:** `422 Unprocessable Content`
- **Response Body:**
```json
{
    "message": "ISBN sudah terdaftar.",
    "errors": {
        "isbn": [
            "ISBN sudah terdaftar."
        ]
    }
}
```
*Analisis:* Rule `unique:books,isbn` mencegah input data dengan ISBN yang telah tercatat sebelumnya pada database.

---

## 7. Struktur File Terkait

1. **Routing:** `routes/api.php`
2. **Controller:** `app/Http/Controllers/BookController.php`
3. **Model:** `app/Models/Book.php`
4. **Migration:** `database/migrations/2026_10_08_045509_create_books_table.php`
5. **Seeder:** `database/seeders/BookSeeder.php`
6. **Form Requests:**
   - `app/Http/Requests/StoreBookRequest.php`
   - `app/Http/Requests/UpdateBookRequest.php`
7. **API Resource:** `app/Http/Resources/BookResource.php`
8. **Postman Collection:** `postman/tugas5_collection.json`

---

## 8. Kesimpulan

Implementasi Tugas 5 pada project Laravel telah berhasil diselesaikan dengan memenuhi seluruh kriteria:
1. Operasi CRUD untuk resource `Book` telah berfungsi penuh sesuai standar RESTful API.
2. Validasi data masuk terisolasi dengan rapi pada Form Request layer (`StoreBookRequest` dan `UpdateBookRequest`) dengan pesan validasi yang jelas.
3. Transformasi data keluar terstandarisasi melalui `BookResource`, menjamin konsistensi struktur dan tipe data response.
4. Respon sukses dan kasus error (404 dan 422) telah diuji dan menghasilkan struktur yang konsisten.
5. Koleksi pengujian Postman telah diperbarui dan siap digunakan untuk verifikasi.

---

## 9. Referensi

1. Dokumentasi Resmi Laravel 11 — Routing: [https://laravel.com/docs/11.x/routing](https://laravel.com/docs/11.x/routing)
2. Dokumentasi Resmi Laravel 11 — Controllers: [https://laravel.com/docs/11.x/controllers](https://laravel.com/docs/11.x/controllers)
3. Dokumentasi Resmi Laravel 11 — Eloquent ORM: [https://laravel.com/docs/11.x/eloquent](https://laravel.com/docs/11.x/eloquent)
4. Dokumentasi Resmi Laravel 11 — Validation & Form Requests: [https://laravel.com/docs/11.x/validation](https://laravel.com/docs/11.x/validation)
5. Dokumentasi Resmi Laravel 11 — Eloquent: API Resources: [https://laravel.com/docs/11.x/eloquent-resources](https://laravel.com/docs/11.x/eloquent-resources)
6. Postman Documentation — Sending Requests: [https://learning.postman.com/docs/sending-requests/requests/](https://learning.postman.com/docs/sending-requests/requests/)

---

## 10. Deklarasi Penggunaan AI

AI digunakan hanya sebagai alat bantu sedikit untuk merapikan format tata letak dokumentasi markdown dan pengecekan sintaks. Seluruh logika kode, konfigurasi routing, pengujian pada Postman, dan analisis pengerjaan dilakukan sendiri.
