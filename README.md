# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

1. Interface (Trait) vs Model Struct 
Dalam pola Observer tradisional (seperti dalam buku Head First), Interface digunakan karena Subscriber bisa berupa objek yang sangat berbeda-beda (misalnya satu mengirim email, satu mengirim SMS, satu memperbarui log). Dalam kasus BambangShop ini, sebuah Model struct sudah cukup karena semua subscriber memiliki perilaku yang seragam. Mereka adalah entitas eksternal yang dihubungi melalui HTTP request ke sebuah URL. Namun, jika ke depannya kita ingin mendukung tipe subscriber yang berbeda (misalnya Internal Dashboard Subscriber yang tidak menggunakan HTTP, atau Mobile Push Notification Subscriber), maka menggunakan Trait di Rust akan menjadi pilihan yang lebih baik untuk menjaga prinsip Open-Closed Principle.

2. Vec (List) vs DashMap (Map)
Meskipun Vec bisa digunakan untuk menyimpan daftar subscriber, penggunaan DashMap (Map/Dictionary) jauh lebih efisien dan tepat untuk kasus ini karena Keunikan (Uniqueness), DashMap secara otomatis mempermudah kita memastikan bahwa satu url (sebagai key) hanya terdaftar satu kali. Jika menggunakan Vec, kita harus melakukan pengecekan manual (iterasi) setiap kali ingin menambah data agar tidak duplikat. Performa Pencarian & Penghapusan, Pencarian atau penghapusan subscriber berdasarkan url di DashMap memiliki kompleksitas waktu rata-rata O(1), sedangkan pada Vec adalah O(n) karena harus menyisir seluruh list.

3. DashMap vs Singleton Pattern 
DashMap dan Singleton bukan pilihan yang saling menggantikan, mereka bekerja bersama. Di Rust, variabel statis seperti SUBSCRIBERS yang dibungkus lazy_static! sebenarnya sudah mengimplementasikan konsep Singleton (hanya ada satu instance database di memori selama aplikasi berjalan).Namun, masalah utama di Rust bukan hanya soal Singleton, melainkan Thread Safety. Karena aplikasi Rocket berjalan secara multi-threaded, beberapa thread bisa mencoba mengakses/mengubah daftar subscriber secara bersamaan.Singleton saja tidak menjamin keamanan data dari race condition.Kita tetap membutuhkan DashMap (atau kombinasi Mutex/RwLock dengan HashMap) untuk memastikan bahwa akses konkuren ke data tersebut aman secara memori (thread-safe), sesuai dengan aturan ownership dan borrowing yang ketat di Rust.

#### Reflection Publisher-2

1. Mengapa perlu memisahkan Service dan Repository dari Model?
Pemisahan ini mengikuti prinsip Single Responsibility. Model harusnya hanya merepresentasikan struktur data. Repository bertanggung jawab atas cara data disimpan/diambil (abstraksi database). Service menangani logika bisnis (seperti validasi atau pengolahan string). Jika digabung, satu file Model akan menjadi sangat besar, sulit diuji (test), dan sulit dikelola jika aplikasi berkembang.

2. Apa yang terjadi jika hanya menggunakan Model?
Kompleksitas kode akan meledak di dalam satu struct. Sebagai contoh, struct Subscriber harus tahu cara memvalidasi dirinya sendiri, cara menyimpan dirinya ke database, sekaligus cara mengirim notifikasi. Hal ini menyebabkan ketergantungan antar model (tight coupling) yang sangat tinggi, membuat perubahan kecil di satu fitur berisiko merusak fitur lainnya.

3. Manfaat Postman?
Postman sangat membantu dalam memvalidasi API tanpa harus membuat front-end terlebih dahulu. Fitur seperti Collections membantu mengorganisir berbagai skenario pengujian, dan variabel lingkungan (Environments) mempermudah perpindahan antar server lokal dan produksi.

#### Reflection Publisher-3
