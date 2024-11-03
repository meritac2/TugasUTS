# TugasUTS
1. Membuat Database
- Buka phpMyAdmin (http://localhost/phpmyadmin)
- Buat database baru bernama sepatustore
- Pilih database sepatustore, lalu buka tab SQL
- Jalankan query SQL berikut untuk membuat tabel dan menambahkan data sampel:
  CREATE TABLE `sepatu` (
  `id` int(11) NOT NULL,
  `name` varchar(255) NOT NULL,
  `type` varchar(255) NOT NULL,
  `price` varchar(100) NOT NULL,
  `stock` int(11) NOT NULL
  );
