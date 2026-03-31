# BambangShop Receiver App
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
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create SubscriberRequest model struct.`
    -   [x] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [x] Commit: `Implement add function in Notification repository.`
    -   [x] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Commit: `Implement receive_notification function in Notification service.`
    -   [x] Commit: `Implement receive function in Notification controller.`
    -   [x] Commit: `Implement list_messages function in Notification service.`
    -   [x] Commit: `Implement list function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1
1. Kita butuh sinkronisasi (seperti `RwLock` atau `Mutex`) karena `Vec` ini bersifat global (static) dan bisa diakses oleh banyak thread secara bersamaan (konkuren). Kalau dibiarkan tanpa sinkronisasi, bisa terjadi data race. Nah, alasan kita pakai `RwLock` dibanding `Mutex` adalah karena `RwLock` mengizinkan banyak thread buat membaca data secara bersamaan (asalkan ngga ada yang lagi nulis). Sedangkan kalau pakai `Mutex`, proses baca maupun nulis bakal di-lock dan cuma bisa diakses oleh satu thread di satu waktu. Di kasus aplikasi kita, operasi melihat/membaca list notifikasi pastinya bakal lebih sering dipanggil dibanding menambah notifikasi baru, jadi `RwLock` bakal lebih efisien dari sisi performance.

2. Ini karena prinsip ketat memory safety-nya Rust. Di Rust, variabel `static` secara umum itu di-load saat compile time dan diakses secara global. Kalau kita bisa ngubah isinya sebebas di Java, itu bakal rentan banget memicu data race saat ada multithreading. Makanya, kalau mau punya mutable static variables dengan tipe kompleks di Rust, kita butuh semacam `lazy_static` buat memastikan inisialisasinya tertunda secara aman (thread-safe runtime initialization). Selanjutnya datanya juga biasanya perlu dibungkus `RwLock` atau `Mutex` agar proses mutasinya bener-bener aman pas ada concurrent access.

#### Reflection Subscriber-2
1. Iya, sempet baca-baca file kayak `src/lib.rs`. Dari situ aku jadi tahu gimana cara setup App State dan setup awal routing untuk framework Rocket. Di file itu ditunjukin gimana nyiapin respon error handler secara global, dan juga inisiasi variabel global pakai `lazy_static` contohnya `REQWEST_CLIENT`. Klien reqwest itu kan dipake buat nembak API keluar (ke publisher), jadi dari lib.rs ini biar bisa di-reuse sama service lain dengan efisien.

2. Observer pattern bikin proses nambah subscriber baru gampang banget (loose coupling) karena Publisher (Main app) ngga perlu pusing mikirin detail/tipe aplikasinya si subscriber. Publisher cuma nyimpen list URL endpoint buat broadcast aja. Tapi, kalau kita ngespawn lebih dari 1 instance Main app, ceritanya bakal beda dan gak gampang lagi. Soalnya masing-masing instance punya memori penyimpanan state array/DashMap sendiri buat nampung subscriber. Kalau sebuah receiver subscribe ke Main app instance A, dia ngga bakal ke-daftar di instance B. Begitu instance B mau nge-broadcast, si receiver tadi gak dikabarin. Solusinya harus ditambahin shared message-broker atau nyimpen subcriber list di shared database (misal Redis) biar tersinkron antar instance.

3. Udah coba ngerapiin Postman collection-nya. Wah, berguna banget parah! Apalagi kalau nanti di Group Project. Dengan ninggalin dokumentasi dan contoh request/response (save responses) di Postman, temen-temen yang megang frontend jadi enak pas mau integrasi API. Mereka ngga perlu nebak-nebak tipe data atau harus ubek-ubek source code backend kita, tinggal ngehit dari Postman aja buat ngecek behavior-nya.
