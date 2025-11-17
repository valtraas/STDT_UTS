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

 sistem menjamin ketersediaan dasar (ada respons).

2. Soft state 

 keadaan internal sistem dapat berubah seiring waktu tanpa input baru (karena replikasi/propagasi).

3. Eventual consistency 

 akhirnya semua node akan konsisten jika tidak ada update baru (konsistensi bersifat akhirnya tercapai).

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

![Gambar perintah untuk eksekusi yml](/images/eksekusi_yml.png "Eksekusi yml") 

## B. Menjalankan di kontainer primary
![Gambar eksekusi primary.](/images/eksekusi_primary.png "Eksekusi primary")
Gambar tersebut merupakan perintah untuk mengeksekusi kontainer primary dengan perintah _docker exec -it pg-primary psql -U admin -d app_db_ Setelah perintah ini dijalankan maka kita diminta untuk menginputkan password yang telah di tetapkan sebelumnya di docker-compose.yml. Jika berhasil masuk maka kita akan diarahkan ke app_db. 

 ###  Membuat, mengisi, dan menampilkan tabel message
![Gambar tabel message.](/images/buat_tampil_table.png "Tabel message")


### C. Menjalankan di kontainer replica 
![Gambar eksekusi replica.](/images/eksekusi_replica.png "Eksekusi primary")

 ### Menampilkan dan mencoba mengisi tabel message
![Gambar tabel message.](/images/tampil_buat_replica.png "Tabel message")

