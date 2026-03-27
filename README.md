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
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create SubscriberRequest model struct.`
    -   [ ] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Notification repository.`
    -   [ ] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Commit: `Implement receive_notification function in Notification service.`
    -   [ ] Commit: `Implement receive function in Notification controller.`
    -   [ ] Commit: `Implement list_messages function in Notification service.`
    -   [ ] Commit: `Implement list function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1
1. Dalam tutorial ini, diperlukan mekanisme untuk mengatur akses ke data yang digunakan oleh banyak thread agar tidak terjadi data race. Dua mekanisme yang bisa digunakan adalah Mutex (Mutual Exclusion) dan RwLock (Read-Write Lock).

Mutex memastikan bahwa hanya satu thread yg dapat mengakses data pada satu waktu, baik untuk membaca maupun menulis, tetapi kurang efisien karena semua thread harus menunggu meskipun hanya ingin membaca data. Adapun RwLock membedakan antara operasi membaca dan menulis sehingga anyak thread dapat membaca data secara bersamaan (concurrent read), namun operasi penulisan tetap bersifat eksklusif (hanya satu thread yang boleh menulis dan semua thread lain harus menunggu), hal ini membuat RwLock lebih efisien untuk kasus di mana operasi membaca lebih sering terjadi dibandingkan penulisan.

static ref NOTIFICATIONS: RwLock<Vec<Notification>> = RwLock::new(vec![]);

Potongan kode ini menunjukkan bahwa data notifikasi disimpan sebagai shared in memory state yang dapat diakses oleh banyak thread. Hal ini relevan karena pada bagian sebelumnya, sistem notifikasi menggunakan multithreading utk kirim notifikasi ke banyak subscriber secara paralel.

Ada dua jenis operasi utama terhadap data ini:

Write (menambahkan notifikasi)
NOTIFICATIONS.write().unwrap()
    .push(notification.clone());

Read (membaca semua notifikasi)
NOTIFICATIONS.read().unwrap()
    .iter().map(|f| format!("{}", f.clone())).collect();

Dengan menggunakan RwLock, maka banyak thread dapat membaca daftar notifikasi secara bersamaan tanpa saling menghambat, sementara operasi perasi penulisan tetap aman karena dilakukan secara eksklusif. Adapun jika kita menggunakan Mutex, maka semua operasi baik write ataupun read harus dilakukan secara bergantian. Misalkan ada banyak thread yg mau membaca, mereka juga harus tetap megantre. Hal ini menyebabkan bottleneck dan menurunkan performa, terutama dalam kondisi dimana operasi read banyak dilakukan seperti di tutorial ini.

2. Dalam tutorial ini, kita menggunakan library eksternal lazy_static untuk mendefinisikan Vec dan DashMap sebagai variabel static. Hal ini dilakukan karena Rust tidak mengizinkan variabel static yang bersifat mutable untuk didefinisikan dan dimodifikasi secara langsung seperti pada Java. Pada Java, variabel static dapat dimodifikasi dengan bebas melalui static function. Hal ini dimungkinkan karena Java mengandalkan garbage collector dan menangani masalah concurrency pada saat runtime. Namun, pendekatan ini berisiko menyebabkan race condition jika tidak dikelola dengan baik oleh developer.

Rust menjamin memory safety dan thread safety sejak compile time melalui sistem ownership dan borrowing yang ketat. Oleh karena itu, Rust tidak mengizinkan mutable static secara langsung, terutama untuk struktur data seperti Vec dan DashMap yang bersifat mutable. Jika diperbolehkan, hal ini dapat menyebabkan data race, undefined behavior, memory corruption

Dalam tutorial ini, digunakan lazy_static untuk menginisialisasi variabel global secara aman pada saat runtime, serta mekanisme sinkronisasi seperti RwLock atau struktur data thread-safe seperti DashMap untuk memastikan akses data tetap aman dalam multithreading.

lazy_static! {
    static ref NOTIFICATIONS: RwLock<Vec<Notification>> = RwLock::new(vec![]);
}

Pada contoh tersebut:

Vec dibungkus dengan RwLock agar aman diakses oleh banyak thread. Rust tidak mengizinkan mutable static seperti Java karena ingin menjamin keamanan memori dan mencegah data race sejak compile time. Karena Vec dan DashMap bersifat mutable, Rust mengharuskan penggunaan pendekatan yang aman seperti lazy_static yang dikombinasikan dengan mekanisme sinkronisasi (RwLock) atau struktur thread-safe (DashMap) untuk mengelola shared state secara aman dalam lingkungan concurrent.

#### Reflection Subscriber-2
1. Saya telah mencoba utk membuka file src/lib.rs, dan saya melihat bahwa sepertinya file ini berfungsi sebagai pusat konfigurasi global dalam aplikasi. DI file tsb digunakan lazy_static untuk membuat instance global seperti REQWEST_CLIENT, yang digunakan sebagai HTTP client untuk mengirim request ke Main App, APP_CONFIG, yang menyimpan konfigurasi aplikasi seperti instance_root_url, publisher_root_url, dan instance_name. Konfigurasi ini diambil dari file .env menggunakan dotenv, lalu diproses menggunakan Figment dengan pendekatan default values yang dapat dioverride oleh environment variable. Hal ini membuat aplikasi menjadi fleksibel dan mudah dikonfigurasi tanpa perlu mengubah kode secara langsung. Selain itu, file ini juga menyediakan sistem error handling misal menggunakan struct ErrorResponse, fungsi compose_error_response, dst.

2. Observer Pattern mempermudah dalam menambahkan subscriber baru ke dalam sistem. Dengan pola ini, publisher (Main App) tidak perlu mengetahui detail dari setiap subscriber (Receiver). Publisher hanya mengirimkan notifikasi ke semua subscriber yang terdaftar.

Ketika menjalankan beberapa instance Receiver pada port yang berbeda, proses penambahan subscriber menjadi sangat mudah. Setiap instance cukup melakukan subscribe ke Main App, dan secara otomatis akan menerima notifikasi yang sesuai tanpa perlu mengubah kode di sisi publisher.

Namun, jika kita mencoba menjalankan lebih dari satu instance Main App, kompleksitas sistem akan meningkat. Hal ini krn kita akan perlu mekanisme sinkronisasi antar Main App (misalnya shared database). Sehingga, menambahkan banyak subscriber (Receiver) tetap mudah, tetapi menambahkan banyak publisher (Main App) membutuhkan arsitektur tambahan agar sistem tetap terkoordinasi.

3. Saya menggunakan Postman untuk menguji endpoint seperti subscribe, create product, publish, dan delete. Dengan Postman, saya dapat melihat secara langsung response dari setiap request dan memastikan bahwa alur sistem berjalan dengan benar. Penggunaan Postman membantu karena postman dapat mempermudah pengujian API secara manual.