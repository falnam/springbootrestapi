# Spring Boot REST API Tutorial

Tutorial ini merupakan panduan lengkap untuk membangun REST API menggunakan Spring Boot berdasarkan **Modul 10**. Anda akan mempelajari cara membuat proyek Spring Boot, menghubungkannya ke MySQL, serta mengimplementasikan dan menguji operasi CRUD menggunakan Postman.

## Fitur yang Dibangun

- GET semua produk
- GET produk berdasarkan ID
- POST produk baru
- PUT untuk memperbarui produk
- DELETE untuk menghapus produk

## Teknologi yang Digunakan

- Java (JDK 17+)
- Spring Boot 3.5.0
- Spring Web
- Spring Data JPA
- MySQL
- Eclipse IDE
- Postman

---

## Langkah Cepat Instalasi

### 1. Kloning Repository

```bash
git clone https://github.com/falnam/rest-api.git
cd rest-api
````

### 2. Setup Database

1. Jalankan MySQL (via XAMPP)
2. Buat database `penjualan` dan tabel `product`:

```sql
CREATE DATABASE penjualan;

CREATE TABLE product (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  price DOUBLE NOT NULL
);

INSERT INTO product (name, price) VALUES
('Laptop', 15000000.0),
('Mouse', 250000.0),
('Keyboard', 300000.0);
```

### 3. Konfigurasi Database

Edit file `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/penjualan
spring.datasource.username=root
spring.datasource.password=
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

### 4. Jalankan Aplikasi

Jalankan melalui Eclipse dengan klik kanan `RestApiApplication.java > Run As > Spring Boot App`.

---

## Uji API dengan Postman

| Metode | Endpoint         | Deskripsi                |
| ------ | ---------------- | ------------------------ |
| GET    | `/products`      | Mendapatkan semua produk |
| GET    | `/products/{id}` | Mendapatkan produk by ID |
| POST   | `/products`      | Menambahkan produk baru  |
| PUT    | `/products/{id}` | Memperbarui produk by ID |
| DELETE | `/products/{id}` | Menghapus produk by ID   |

Contoh JSON untuk POST/PUT:

```json
{
  "name": "Headset Gaming",
  "price": 750000
}
```

---

## Pemecahan Masalah

* **Port 8080 digunakan?**
  Tambahkan di `application.properties`:
  `server.port=8081`

* **Masalah koneksi DB?**
  Pastikan `MySQL` aktif dan kredensial benar.

* **Dependency tidak terunduh?**
  Klik kanan proyek → Maven → Update Project.

---

## Lisensi

Proyek ini untuk keperluan pembelajaran. Bebas digunakan, dimodifikasi, dan dikembangkan lebih lanjut.

---

## Penulis

Tutorial disusun berdasarkan Modul 10 dan diadaptasi untuk implementasi nyata oleh \[Ade Fatkhul Anam].

```
