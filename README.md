# Praktikum 10 — REST API dengan CodeIgniter 4

## Daftar Isi

- [Apa itu REST API?](#apa-itu-rest-api)
- [Langkah-langkah Praktikum](#langkah-langkah-praktikum)
  - [1. Persiapan](#1-persiapan)
  - [2. Membuat Model](#2-membuat-model)
  - [3. Membuat REST Controller](#3-membuat-rest-controller)
  - [4. Membuat Routing REST API](#4-membuat-routing-rest-api)
  - [5. Testing REST API dengan Postman](#5-testing-rest-api-dengan-postman)

---

## Apa itu REST API?

**REST (Representational State Transfer)** adalah salah satu desain arsitektur *Application Programming Interface* (API). API sendiri merupakan interface yang menjadi perantara yang menghubungkan satu aplikasi dengan aplikasi lainnya.

REST API berisi aturan untuk membuat web service dengan membatasi hak akses client yang mengakses API. Jika dianalogikan sebagai restoran, REST API adalah **daftar menu** — pelanggan hanya bisa memesan sesuai daftar menu meskipun si koki (server) bisa membuatkan pesanan tersebut. Pembatasan ini dilakukan untuk melindungi database/resource yang ada di server.

### Cara Kerja REST API

REST API menggunakan prinsip **REST Server** dan **REST Client**:

| Komponen | Peran |
|----------|-------|
| **REST Server** | Bertindak sebagai penyedia data/resource |
| **REST Client** | Membuat HTTP request ke server menggunakan URI atau global ID |

Server akan memberikan response dengan mengirim kembali HTTP request yang diminta client. Data yang dikirim maupun diterima biasanya berformat **JSON**, sehingga REST API mudah diintegrasikan dengan berbagai platform, bahasa pemrograman, maupun framework yang berbeda.

> 💡 **Contoh:** Backend dibuat menggunakan REST API dengan PHP (CodeIgniter), lalu dihubungkan ke frontend yang menggunakan Vue.js.

---

## Langkah-langkah Praktikum

### 1. Persiapan

Unduh dan install aplikasi **Postman** sebagai REST Client untuk keperluan testing REST API.

> 🔗 Download Postman: [https://www.postman.com/downloads/](https://www.postman.com/downloads/)

---

### 2. Membuat Model

Pada modul ini kita memanfaatkan `ArtikelModel` yang sudah dibuat pada modul sebelumnya agar dapat diakses melalui API.

---

### 3. Membuat REST Controller

Masuk ke direktori `app/Controllers` dan buat file baru bernama `Post.php`, lalu isi dengan kode berikut:

**`app/Controllers/Post.php`**

```php
<?php

namespace App\Controllers;

use CodeIgniter\RESTful\ResourceController;
use CodeIgniter\API\ResponseTrait;
use App\Models\ArtikelModel;

class Post extends ResourceController
{
    use ResponseTrait;

    // Menampilkan semua data
    public function index()
    {
        $model           = new ArtikelModel();
        $data['artikel'] = $model->orderBy('id', 'DESC')->findAll();
        return $this->respond($data);
    }

    // Menambahkan data baru
    public function create()
    {
        $model = new ArtikelModel();
        $data  = [
            'judul' => $this->request->getVar('judul'),
            'isi'   => $this->request->getVar('isi'),
        ];
        $model->insert($data);
        $response = [
            'status'   => 201,
            'error'    => null,
            'messages' => [
                'success' => 'Data artikel berhasil ditambahkan.'
            ]
        ];
        return $this->respondCreated($response);
    }

    // Menampilkan data spesifik
    public function show($id = null)
    {
        $model = new ArtikelModel();
        $data  = $model->where('id', $id)->first();
        if ($data) {
            return $this->respond($data);
        } else {
            return $this->failNotFound('Data tidak ditemukan.');
        }
    }

    // Mengubah data
    public function update($id = null)
    {
        $model = new ArtikelModel();
        $id    = $this->request->getVar('id');
        $data  = [
            'judul' => $this->request->getVar('judul'),
            'isi'   => $this->request->getVar('isi'),
        ];
        $model->update($id, $data);
        $response = [
            'status'   => 200,
            'error'    => null,
            'messages' => [
                'success' => 'Data artikel berhasil diubah.'
            ]
        ];
        return $this->respond($response);
    }

    // Menghapus data
    public function delete($id = null)
    {
        $model = new ArtikelModel();
        $data  = $model->where('id', $id)->delete($id);
        if ($data) {
            $model->delete($id);
            $response = [
                'status'   => 200,
                'error'    => null,
                'messages' => [
                    'success' => 'Data artikel berhasil dihapus.'
                ]
            ];
            return $this->respondDeleted($response);
        } else {
            return $this->failNotFound('Data tidak ditemukan.');
        }
    }
}
```

**Penjelasan method:**

| Method | Fungsi |
|--------|--------|
| `index()` | Menampilkan seluruh data dari database |
| `create()` | Menambahkan data baru ke database |
| `show($id)` | Menampilkan satu data spesifik berdasarkan ID |
| `update($id)` | Mengubah data tertentu di database |
| `delete($id)` | Menghapus data dari database |

---

### 4. Membuat Routing REST API

Buka file `app/Config/Routes.php` dan tambahkan baris berikut:

```php
$routes->resource('post');
```

Untuk mengecek daftar route yang sudah terdaftar, jalankan perintah:

```bash
php spark routes
```

> Satu baris kode `resource()` akan secara otomatis menghasilkan banyak endpoint sekaligus untuk operasi GET, POST, PUT, dan DELETE.

---

### 5. Testing REST API dengan Postman

Buka aplikasi Postman, pilih **Create New → HTTP Request**, lalu lakukan pengujian berikut:

---

#### Menampilkan Semua Data

| | |
|---|---|
| **Method** | `GET` |
| **URL** | `http://localhost:8080/post` |

Klik **Send**. Jika berhasil, semua data artikel dari database akan ditampilkan dalam format JSON.

---

#### Menampilkan Data Spesifik

| | |
|---|---|
| **Method** | `GET` |
| **URL** | `http://localhost:8080/post/2` |

Klik **Send**. Response akan menampilkan data artikel dengan ID 2.

---

#### Menambahkan Data

| | |
|---|---|
| **Method** | `POST` |
| **URL** | `http://localhost:8080/post` |
| **Body** | `x-www-form-urlencoded` |

Isi kolom **KEY** dengan nama atribut (`judul`, `isi`) dan kolom **VALUE** dengan data yang ingin ditambahkan, lalu klik **Send**.

---

#### Mengubah Data

| | |
|---|---|
| **Method** | `PUT` |
| **URL** | `http://localhost:8080/post/2` |
| **Body** | `x-www-form-urlencoded` |

Isi kolom **KEY** dan **VALUE** dengan data yang ingin diperbarui, lalu klik **Send**.

---

#### Menghapus Data

| | |
|---|---|
| **Method** | `DELETE` |
| **URL** | `http://localhost:8080/post/7` |

Klik **Send**. Response akan menampilkan pesan bahwa data berhasil dihapus dari database.

---

*Laporan Praktikum — REST API dengan CodeIgniter 4 | Pemrograman Web*
