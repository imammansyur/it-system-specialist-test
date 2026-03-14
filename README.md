# IT System Specialist Test

## 1. Tes Pemrograman & Database (Logika)

### a. Database

Saya menggunakan syntax MySQL untuk pengerjaan database.

```sql
CREATE TABLE barang (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    nama_barang VARCHAR(255) NOT NULL,
    serial_number VARCHAR(255),
    status VARCHAR(8) NOT NULL CHECK (status IN ('tersedia','rusak','dipinjam'),
    tgl_maintenance DATE,
);
```

### b. Query

```sql
SELECT *
FROM barang
WHERE status='rusak'
AND tgl_maintenance >= CURRENT_DATE - INTERVAL 6 MONTH;
```

### c. Logika Coding

Asumsi yang saya gunakan adalah:
- Database berdasarkan soal 1a
- tgl_maintenance adalah tanggal terakhir dilakukannya maintenance, bukan tanggal rencana dilakukannya maintenance
- Maintenance hanya dilakukan pada barang dengan status rusak. Barang yang tersedia memerlukan maintenance rutin dan barang yang dipinjam tidak dapat dilakukan maintenance. Namun saya memutuskan untuk hanya melakukan maintenance pada barang yang rusak sebagai kelanjutan dari soal 1b.
- Jatuh tempo maintenance berasal dari ketentuan maintenance yang dilakukan setiap 6 bulan berdasarkan soal 1b.

Environment yang digunakan:
- MySQL database
- Python untuk skrip
- Package Python mysql-connector-python

Secara garis besar, yang akan saya lakukan dengan skrip ini adalah:
- Menghubungkan dengan database MySQL
- Menjalankan query dari jawaban atas soal 1b
- Print teks output dari query
- Menutup koneksi dengan database MySQL

```python
import mysql.connector

config = {
    'user': 'user',
    'password': 'password',
    'database': 'database'
}

query = ("SELECT * FROM barang WHERE status='rusak' AND tgl_maintenance >= CURRENT_DATE - INTERVAL 6 MONTH;")

connection = mysql.connector.connect(**config)
cursor = connection.cursor()

cursor.execute(query)

for row in cursor:
    print row

cursor.close()
connection.close()
```

## 2. Tes Infrastruktur & Jaringan (Troubleshooting)

### a. Masalah Jaringan

1. Melakukan konfirmasi apakah benar-benar terjadi masalah tersebut dengan melakukan ping & traceroute dari perangkat user yang bermasalah ke internet dan jaringan uplink dari lantai kantor.
2. Memastikan tidak ada konfigurasi yang salah yang berimbas pada jaringan lantai tersebut tidak dapat terkoneksi ke internet. Konfigurasi bisa berupa routing table, ACL/firewall rule, dan konfigurasi switch.
3. Memastikan bahwa perangkat jaringan terhubung secara fisik, mulai dari access point hingga switch dan router, baik dari kabel UTP, kaber power device, dan port yang terhubung.

### b. Keamanan Fisik

Pertama, saya akan membuatkan jaringan WLAN baru khusus tamu yang berbeda dari jaringan internal perusahaan. Kemudian, saya membuat VLAN baru dari jaringan tamu. Setelah itu, saya akan membuat rule pada Access Control List (ACL) atau firewall untuk membatasi akses ke jaringan internal dan hanya membolehkan akses ke internet. Rule yang saya pakai seperti berikut:

```
deny [source: jaringan tamu] [dest: jaringan internal]
allow [source: jaringan tamu] 0.0.0.0/0
```

## 3. Tes Administrasi Server (Manajemen & Recovery)

### a. Backup & Restore

Saya akan menggunakan RAID pada disk dari server database. RAID sendiri merupakan teknologi yang menggabungkan beberapa disk menjadi satu. Seperti contoh, RAID 1 membutuhkan 2 disk untuk membuat 1 disk, kedua disk memuat data yang sama (mirroring). Rusaknya 1 disk tidak menghilangkan data yang ada sehingga hanya perlu dilakukan penggantian disk yang rusak dengan yang baru. 

Namun, RAID saja tidak cukup untuk mencegah hilangnya data dari kerusakan 1 pasang disk RAID, yang mengakibatkan downtime server database. Replikasi database diperlukan untuk menjaga database tetap dapat diakses pada saat database lain mengalami disk failure yang berujung hilangnya data. Replikasi database menggunakan sistem master-slave, master untuk akses CRUD dan slave untuk akses read saja.

### b. Optimasi

Saat server diakses oleh banyak karyawan secara bersamaan, saya akan memeriksa resource server, mulai dari CPU, RAM, Swap, Disk IO, dan Network. Untuk pengecekan, saya menggunakan `top` untuk melihat resource sekaligus melihat program yang saat ini berjalan. 
