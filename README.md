# Jarkom_Modul2_Lapres_D07
Laporan Resmi Modul 2 Praktikum Jaringan Komputer 2020

**Soal**
Kalian diminta untuk membuat sebuah website utama dengan (1) alamat http://semeruyyy.pw yang
memiliki (2) alias http://www.semeruyyy.pw, dan (3) subdomain
http://www.penanjakan.semeruyyy.pw yang diatur DNS-nya pada MALANG dan mengarah ke IP
Server PROBOLINGGO serta dibuatkan (4) reverse domain untuk domain utama. Untuk mengantisipasi
server dicuri/rusak, Bibah minta dibuatkan (5) DNS Server Slave pada MOJOKERTO agar Bibah tidak
terganggu menikmati keindahan Semeru pada Website. Selain website utama Bibah juga meminta
dibuatkan (6) subdomain dengan alamat http://gunung.semeruyyy.pw yang didelegasikan pada server
MOJOKERTO dan mengarah ke IP Server PROBOLINGGO. Bibah juga ingin memberi petunjuk
mendaki gunung semeru kepada anggota komunitas sehingga dia meminta dibuatkan (7) subdomain
dengan nama http://naik.gunung.semeruyyy.pw, domain ini diarahkan ke IP Server PROBOLINGGO.
Setelah selesai membuat keseluruhan domain, kamu diminta untuk segera mengatur web server. (8)
Domain http://semeruyyy.pw memiliki DocumentRoot pada /var/www/semeruyyy.pw. Awalnya web
dapat diakses menggunakan alamat http://semeruyyy.pw/index.php/home. Karena dirasa alamat urlnya
kurang bagus, maka (9) diaktifkan mod rewrite agar urlnya menjadi http://semeruyyy.pw/home.
(10) Web http://penanjakan.semeruyyy.pw akan digunakan untuk menyimpan assets file yang

1. alamat http://semeruyyy.pw
- Pada UML MALANG, 
-- Ketikkan ```apt-get install bind9 -y```
-- Kemudian ```nano /etc/bind/named.conf.local```
-- Tambahkan kode:
```
zone "semerud07.pw" {
    type master;
    file "/etc/bind/jarkom/semerud07.pw";
};
```
[gambar1]
-- Kemudian buat folder baru: ```mkdir /etc/bind/jarkom```
-- Dan copy file db.local ke folder yang baru saja dibuat dan mengganti namanya sesuai domain yang diinginkan: ```cp /etc/bind/db.local /etc/bind/jarkom/semerud07.pw```
-- Kemudian buka file ```semerud07.pw``` dengan perintah: ```nano /etc/bind/jarkom/semerud07.pw```.
[gambar2]
