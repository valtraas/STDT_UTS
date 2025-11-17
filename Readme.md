# UTS SISTEM TERDISTRIBUSI DAN TERDESENTRALISASI
## NIM : 245410091
## Nama : Noval Putra Asmara


## 1. Jelaskan teorema CAP dan BASE dan keterkaitan keduanya. Jelaskan menggunakan contoh yang pernah anda gunakan. 
 ### CAP 
 Merupakan jenis jaminan yang ada di sistem terdistribusi diantaranya : 
 1. Consistency
 
    Setiap operasi baca yang mengembalikan penulisan terbaru, memastikan semua klien memiliki tampilan data yang sama. 

 2. Availability
    
    Setiap permintaan menerima respons, meskipun bukan data terbaru. 

 3. Partition Tolerance
 
    Sistem tetap beroperasi dan berfungsi meskipun komunikasi antar node terputus atau terfragmnentasi.

Sistem terdesentralisasi tidak mungkin memiliki semua jaminan tersebut harus ada salah satu yang dikorbankan. 

 ### BASE

Base atau singkatan dari Basically Available, Soft state, Eventual consistency adalah  model desain yang sering digunakan untuk sistem yang memilih AP (Availability + Partition Tolerance).

1. Basically Available 

 sistem menjamin ketersediaan dasar.

2. Soft state 

 keadaan internal sistem dapat berubah seiring waktu tanpa input baru.

3. Eventual consistency 

 akhirnya semua node akan konsisten jika tidak ada update baru .

### Keterkaitan antara CAP dan BASE

Keterkaitan antar keduanya adalah CAP adalah jaminan yang ada di sistem terdistribusi dan tidak mungkin menerapan semuanya dan BASE merupakan model yang sering dipakai sisem dengan memilih sifat AP ( availability dan Partition tolerance) atau dapat diartikan bahwa CAP adalah teorinya dan BASE adalah implementasinya.

### Contoh
Sistem bank menggunakan CP atau Consistency dan Partition tolerance. Hal ini penting karena data saldo di bank harus akurat, namun saat pratisi jaringan sistem ini memungkinkan untuk menolak transaksi untuk menghindari inkonsistensi ( mengorbankan availability )


## 2. Jelaskan keterkaitan antara GraphQL dengan komunikasi antar proses pada sistem terdistribusi. Buat diagramnya.

GraphQL adalah lapisan query yang biasanya berjalan sebagai API gateway / orchestration layer.

 Dalam arsitektur terdistribusi, GraphQL memfasilitasi komunikasi antar-proses dengan menjadi titik koordinasi  antara klien dan banyak layanan backend. Komunikasi antar-proses di sini bisa terjadi lewat HTTP, gRPC, message queue (Kafka/RabbitMQ), atau WebSocket (mis. untuk subscription).

  GraphQL menyederhanakan panggilan multiple round-trips pada klien menjadi satu endpoint yang melakukan banyak panggilan internal (fan-out), batching, caching, dan agregasi data.

  ## Diagram
 ![This is an alt text.](/images/diagram.png "Diagram graphql")

## 3. Dengan menggunakan Docker / Docker Compose, buatlah streaming replication di PostgreSQL yang bisa menjelaskan sinkronisasi. Tulislah langkah-langkah pengerjaannya dan buat penjelasan secukupnya.

## A. Membuat docker-compose.yml
    services:
    primary:
        image: bitnami/postgresql:latest
        container_name: pg-primary
        ports:
        - '5432:5432'
        volumes:
        - primary_data:/bitnami/postgresql
        environment:
        - POSTGRESQL_USERNAME=admin
        - POSTGRESQL_PASSWORD=adminpass
        - POSTGRESQL_DATABASE=app_db
        - POSTGRESQL_REPLICATION_MODE=master
        - POSTGRESQL_REPLICATION_USER=replicator
        - POSTGRESQL_REPLICATION_PASSWORD=reppass

    replica:
        image: bitnami/postgresql:latest
        container_name: pg-replica
        ports:
        - '5433:5432'
        depends_on:
        - primary
        volumes:
        - replica_data:/bitnami/postgresql
        environment:
        - POSTGRESQL_REPLICATION_MODE=slave
        - POSTGRESQL_MASTER_HOST=primary   
        - POSTGRESQL_MASTER_PORT_NUMBER=5432
        
        - POSTGRESQL_REPLICATION_USER=replicator
        - POSTGRESQL_REPLICATION_PASSWORD=reppass
        - POSTGRESQL_USERNAME=admin
        - POSTGRESQL_PASSWORD=adminpass
        - POSTGRESQL_DATABASE=app_db

        volumes:
        primary_data:
        replica_data:


### Mengeksekusi docker-compose.yml
 Untuk mengeksekusi docker-compose maka gunakan perintah
   
    docker-compose up -d

Perintah tersebut berarti 
    
     docker-compose up
Memerintahkan Docker Compose untuk membaca file docker-compose.yml, kemudian menjalankan dan membuat seluruh service (container) yang terdefinisi di dalamnya. Jika image belum tersedia, Docker akan melakukan proses build atau menarik image yang diperlukan.

    -d (detach mode)
Menjalankan seluruh container secara background (detached). 

![Gambar perintah untuk eksekusi yml](/images/eksekusi_yml.png "Eksekusi yml") 

## B. Menjalankan di kontainer primary
![Gambar eksekusi primary.](/images/eksekusi_primary.png "Eksekusi primary")

Gambar tersebut merupakan perintah untuk mengeksekusi kontainer primary dengan perintah _docker exec -it pg-primary psql -U admin -d app_db_ Setelah perintah ini dijalankan maka kita diminta untuk menginputkan password yang telah di tetapkan sebelumnya di docker-compose.yml. Jika berhasil masuk maka kita akan diarahkan ke app_db. 

 ###  Membuat, mengisi, dan menampilkan tabel message
![Gambar tabel message.](/images/buat_tampil_table.png "Tabel message")

Gambar tersebut merupakan tampilan proses pembuatan tabel message pada database di dalam kontainer primary. Pada tahap ini dilakukan eksekusi perintah SQL untuk membuat tabel dengan kolom id sebagai primary key dan content sebagai kolom berisi teks pesan. Setelah tabel berhasil dibuat, dilakukan proses insert dua data contoh, yaitu “Pesan pertama” dan “Pesan kedua”. Selanjutnya, perintah SELECT digunakan untuk menampilkan isi tabel dan membuktikan bahwa data berhasil tersimpan di dalam primary database.

### C. Menjalankan di kontainer replica 
![Gambar eksekusi replica.](/images/eksekusi_replica.png "Eksekusi primary")

Gambar tersebut menunjukkan proses masuk ke dalam kontainer replica menggunakan perintah docker exec. Setelah masuk, dilakukan koneksi ke database menggunakan akun admin. Tahap ini bertujuan untuk mengamati apakah tabel dan data yang dibuat pada primary telah direplikasi dengan benar ke dalam kontainer replica

 ### Menampilkan dan mencoba mengisi tabel message
![Gambar tabel message.](/images/tampil_buat_replica.png "Tabel message")

Gambar tersebut menampilkan hasil query SELECT pada kontainer replica. Terlihat bahwa data yang sebelumnya dibuat di primary berhasil muncul juga di replica, menandakan bahwa proses replikasi berjalan dengan baik. Namun, ketika dilakukan perintah INSERT untuk menambah data baru pada replica, muncul pesan error “cannot execute INSERT in a read-only transaction”.

Hal ini menunjukkan bahwa kontainer replica bersifat read-only, sesuai dengan konsep replikasi database PostgreSQL, di mana primary bertugas melakukan operasi tulis (write) dan replica hanya digunakan untuk operasi baca (read). Dengan demikian, replica tidak mengizinkan operasi insert, update, atau delete.

