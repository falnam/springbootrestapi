# Tutorial Implementasi REST API Spring Boot

## Pendahuluan

Tutorial ini akan memandu Anda melalui proses pembuatan REST API menggunakan Spring Boot dan mengujinya dengan Postman. Tutorial ini mencakup semua langkah mulai dari pembuatan proyek hingga pengujian API.

## Prasyarat

- Java Development Kit (JDK)
- Eclipse IDE
- XAMPP atau MySQL Server
- Postman

## Langkah 1: Persiapan Lingkungan

1. **Instal Eclipse IDE** jika belum ada
2. **Siapkan MySQL & XAMPP**
   - Download dan instal XAMPP
   - Pastikan MySQL service dalam keadaan aktif

## Langkah 2: Membuat Project Spring Boot

### Menggunakan Spring Initializr dari Eclipse:

1. Buka Eclipse
2. Pilih menu **File > New > Spring Starter Project**
3. Isi detail proyek:
   - **Name**: rest-api
   - **Group**: com.example
   - **Artifact**: rest-api
   - **Description**: Spring Boot REST API Example
   - **Package**: com.example.restapi
   - **Type**: Maven
   - **Packaging**: Jar
   - **Java Version**: 23 (atau sesuai JDK yang diinstal)
   - **Language**: Java
   - **Spring Boot Version**: 3.5.0 (atau versi terbaru yang stabil)

4. Klik **Next**

5. Tambahkan dependencies berikut:
   - **Spring Web**
   - **Spring Boot DevTools**
   - **Spring Data JPA**
   - **MySQL Driver**

6. Klik **Finish**

### Alternatif: Menggunakan Browser

1. Buka [Spring Initializr](https://start.spring.io/) di browser
2. Isi konfigurasi yang sama seperti di atas
3. Klik **Generate** untuk mengunduh proyek dalam format ZIP
4. Ekstrak file ZIP
5. Di Eclipse, pilih **File > Import > Existing Maven Projects**
6. Pilih folder proyek yang telah diekstrak dan klik **Finish**

## Langkah 3: Menyiapkan Database

1. Buka XAMPP Control Panel dan pastikan MySQL service berjalan
2. Buka phpMyAdmin (http://localhost/phpmyadmin/)
3. Buat database baru dengan nama `penjualan`:
   ```sql
   CREATE DATABASE penjualan;
   ```

4. Pilih database `penjualan` dan buat tabel `product`:
   ```sql
   CREATE TABLE product (
     id BIGINT AUTO_INCREMENT PRIMARY KEY,
     name VARCHAR(255) NOT NULL,
     price DOUBLE NOT NULL
   );
   ```

5. Tambahkan data awal:
   ```sql
   INSERT INTO product (name, price) VALUES
   ('Laptop', 15000000.0),
   ('Mouse', 250000.0),
   ('Keyboard', 300000.0);
   ```

## Langkah 4: Konfigurasi Database di Spring Boot

1. Buka file `application.properties` di folder `src/main/resources/`
2. Tambahkan konfigurasi berikut:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/penjualan
spring.datasource.username=root
spring.datasource.password=
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

## Langkah 5: Membuat Model Entity

1. Buat package baru di `com.example.restapi.model`
2. Buat file `Product.java` dalam package tersebut:

```java
package com.example.restapi.model;

import jakarta.persistence.*;

@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private double price;
    
    public Product() {}
    
    public Product(String name, double price) {
        this.name = name;
        this.price = price;
    }
    
    public Long getId() {
        return id;
    }
    
    public void setId(Long id) {
        this.id = id;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public double getPrice() {
        return price;
    }
    
    public void setPrice(double price) {
        this.price = price;
    }
}
```

## Langkah 6: Membuat Repository

1. Buat package baru di `com.example.restapi.repository`
2. Buat file `ProductRepository.java` dalam package tersebut:

```java
package com.example.restapi.repository;

import com.example.restapi.model.Product;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ProductRepository extends JpaRepository<Product, Long> {
}
```

## Langkah 7: Membuat Service Layer

1. Buat package baru di `com.example.restapi.service`
2. Buat file `ProductService.java` dalam package tersebut:

```java
package com.example.restapi.service;

import com.example.restapi.model.Product;
import com.example.restapi.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;
import java.util.Optional;

@Service
public class ProductService {
    @Autowired
    private ProductRepository repository;
    
    public List<Product> getAllProducts() {
        return repository.findAll();
    }
    
    public Optional<Product> getProductById(Long id) {
        return repository.findById(id);
    }
    
    public Product saveProduct(Product product) {
        return repository.save(product);
    }
    
    public Product updateProduct(Long id, Product productDetails) {
        return repository.findById(id).map(product -> {
            product.setName(productDetails.getName());
            product.setPrice(productDetails.getPrice());
            return repository.save(product);
        }).orElse(null);
    }
    
    public void deleteProduct(Long id) {
        repository.deleteById(id);
    }
}
```

## Langkah 8: Membuat Controller

1. Buat package baru di `com.example.restapi.controller`
2. Buat file `ProductController.java` dalam package tersebut:

```java
package com.example.restapi.controller;

import com.example.restapi.model.Product;
import com.example.restapi.service.ProductService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;
import java.util.Optional;

@RestController
@RequestMapping("/products")
public class ProductController {
    @Autowired
    private ProductService service;
    
    @GetMapping
    public List<Product> getAll() {
        return service.getAllProducts();
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getById(@PathVariable Long id) {
        Optional<Product> product = service.getProductById(id);
        return product.map(ResponseEntity::ok).orElseGet(() ->
            ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public Product create(@RequestBody Product product) {
        return service.saveProduct(product);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<Product> update(@PathVariable Long id, @RequestBody Product productDetails) {
        Product updatedProduct = service.updateProduct(id, productDetails);
        return updatedProduct != null ? ResponseEntity.ok(updatedProduct) : 
            ResponseEntity.notFound().build();
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        service.deleteProduct(id);
        return ResponseEntity.noContent().build();
    }
}
```

## Langkah 9: Menjalankan Aplikasi Spring Boot

1. Pastikan kode `RestApiApplication.java` sudah ada seperti berikut:

```java
package com.example.restapi;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class RestApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(RestApiApplication.class, args);
    }
}
```

2. Klik kanan file `RestApiApplication.java`
3. Pilih **Run As > Spring Boot App**
4. Tunggu sampai aplikasi berhasil dijalankan. Jika berhasil, Anda akan melihat pesan di console:
   ```
   Tomcat started on port(s): 8080 (http)
   Started RestApiApplication in X.XX seconds
   ```

## Langkah 10: Menguji REST API dengan Postman

### GET - Mendapatkan Semua Produk
1. Metode: **GET**
2. URL: `http://localhost:8080/products`
3. Klik **Send**
4. Anda akan melihat daftar produk dalam format JSON:
   ```json
   [
     {
       "id": 1,
       "name": "Laptop",
       "price": 15000000.0
     },
     {
       "id": 2,
       "name": "Mouse",
       "price": 250000.0
     },
     {
       "id": 3,
       "name": "Keyboard",
       "price": 300000.0
     }
   ]
   ```

### GET - Mendapatkan Produk Berdasarkan ID
1. Metode: **GET**
2. URL: `http://localhost:8080/products/1` (untuk mendapatkan produk dengan ID 1)
3. Klik **Send**
4. Anda akan melihat detail produk dengan ID tersebut:
   ```json
   {
     "id": 1,
     "name": "Laptop",
     "price": 15000000.0
   }
   ```

### POST - Membuat Produk Baru
1. Metode: **POST**
2. URL: `http://localhost:8080/products`
3. Pilih tab **Body**
4. Pilih **raw** dan set format ke **JSON**
5. Masukkan data JSON:
   ```json
   {
     "name": "Headset",
     "price": 500000
   }
   ```
6. Klik **Send**
7. Anda akan melihat detail produk yang baru dibuat termasuk ID yang diberikan:
   ```json
   {
     "id": 4,
     "name": "Headset",
     "price": 500000.0
   }
   ```

### PUT - Memperbarui Produk
1. Metode: **PUT**
2. URL: `http://localhost:8080/products/4` (jika produk Headset adalah ID 4)
3. Pilih tab **Body**
4. Pilih **raw** dan set format ke **JSON**
5. Masukkan data JSON yang diperbarui:
   ```json
   {
     "name": "Headset Gaming",
     "price": 750000
   }
   ```
6. Klik **Send**
7. Anda akan melihat detail produk yang sudah diperbarui:
   ```json
   {
     "id": 4,
     "name": "Headset Gaming",
     "price": 750000.0
   }
   ```

### DELETE - Menghapus Produk
1. Metode: **DELETE**
2. URL: `http://localhost:8080/products/4` (untuk menghapus produk dengan ID 4)
3. Klik **Send**
4. Anda akan mendapatkan respon status 204 No Content yang mengindikasikan penghapusan berhasil

## Verifikasi di Database

Untuk memastikan bahwa data benar-benar tersimpan di database:

1. Buka phpMyAdmin (http://localhost/phpmyadmin/)
2. Pilih database `penjualan`
3. Klik pada tabel `product`
4. Anda seharusnya bisa melihat perubahan data yang telah Anda lakukan melalui API

## Pemecahan Masalah

Jika Anda mengalami kesalahan saat menjalankan aplikasi, berikut beberapa hal yang perlu diperiksa:

1. **Koneksi Database**: Pastikan MySQL berjalan di port 3306 dan username/password di `application.properties` sudah benar.

2. **Dependencies**: Pastikan semua dependencies sudah terunduh dengan benar. Anda dapat melakukan Maven Update dengan klik kanan pada proyek > Maven > Update Project.

3. **Port yang Sudah Digunakan**: Jika muncul error "Port 8080 already in use", Anda bisa mengubah port di `application.properties`:
   ```
   server.port=8081
   ```

4. **Java Version**: Pastikan versi Java yang digunakan sesuai dengan konfigurasi proyek.

## Kesimpulan

Selamat! Anda telah berhasil membuat REST API menggunakan Spring Boot dan mengujinya dengan Postman. API ini menyediakan operasi CRUD lengkap untuk entitas Product dan terintegrasi dengan database MySQL.

Tutorial ini dapat dijadikan dasar untuk pengembangan API yang lebih kompleks. Anda dapat menambahkan fitur seperti validasi data, autentikasi, atau endpoint tambahan sesuai kebutuhan aplikasi Anda.

## Submission Praktikum

- Upload project yang kalian buat ke dalam repository github dengan nama:
  'springbootrestapi'
- Screenshoot dan sertakan link (public) repository kalian
- Submit dalam format .pdf ke link di deskripsi grup
