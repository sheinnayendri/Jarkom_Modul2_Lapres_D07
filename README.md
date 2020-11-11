# Jarkom_Modul2_Lapres_D07
Laporan Resmi Modul 2 Praktikum Jaringan Komputer 2020

Sebelum mengerjakan soal 1-17, perlu dilakukan setting topologi terlebih dahulu dengan membuat file **topologi.sh** yang berisi:
```
# Switch
uml_switch -unix switch1 > /dev/null < /dev/null &
uml_switch -unix switch2 > /dev/null < /dev/null &

# Router
xterm -T SURABAYA -e linux ubd0=SURABAYA,jarkom umid=SURABAYA eth0=tuntap,,,10.151.78.33 eth1=daemon,,,switch2 eth2=daemon,,,switch1 mem=96M &

# Server
xterm -T MALANG -e linux ubd0=MALANG,jarkom umid=MALANG eth0=daemon,,,switch2 mem=128M &
xterm -T MOJOKERTO -e linux ubd0=MOJOKERTO,jarkom umid=MOJOKERTO eth0=daemon,,,switch2 mem=128M &
xterm -T PROBOLINGGO -e linux ubd0=PROBOLINGGO,jarkom umid=PROBOLINGGO eth0=daemon,,,switch2 mem=128M &

# Klien
xterm -T SIDOARJO -e linux ubd0=SIDOARJO,jarkom umid=SIDOARJO eth0=daemon,,,switch1 mem=96M &
xterm -T GRESIK -e linux ubd0=GRESIK,jarkom umid=GRESIK eth0=daemon,,,switch1 mem=96M &
```

Kemudian setelah masuk ke UML, pada router SURABAYA lakukan setting sysctl dengan mengetikkan perintah ```nano /etc/sysctl.conf```. Hilangkan tanda pagar (#) pada bagian ```net.ipv4.ip_forward=1```. Lalu mengetikkan ```sysctl -p``` untuk mengaktifkan perubahan yang ada. Dengan mengaktifkan fungsi IP Forward ini maka Linux nantinya dapat menentukan jalur mana yang dipilih untuk mencapai jaringan tujuan.

Lalu dilakukan setting IP pada setiap UML dengan mengetikkan ```nano /etc/network/interfaces``` Lalu setting sebagai berikut:

**SURABAYA (Sebagai Router)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.78.34
netmask 255.255.255.252
gateway 10.151.78.33

auto eth1
iface eth1 inet static
address 10.151.79.65
netmask 255.255.255.248

auto eth2
iface eth2 inet static
address 192.168.0.1
netmask 255.255.255.0
```

**MALANG (Sebagai DNS Server Master)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.79.66
netmask 255.255.255.248
gateway 10.151.79.65
```

**MOJOKERTO (Sebagai DNS Server Slave)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.79.67
netmask 255.255.255.248
gateway 10.151.79.65
```

**PROBOLINGGO (Sebagai Web Server)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 10.151.79.68
netmask 255.255.255.248
gateway 10.151.79.65
```

**SIDOARJO (Sebagai Klien)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.0.2
netmask 255.255.255.0
gateway 192.168.0.1
```

**GRESIK (Sebagai Klien)**
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 192.168.0.3
netmask 255.255.255.0
gateway 192.168.0.1
```

Kemudian restart network dengan mengetikkan ```service networking restart``` di setiap UML.
Setelah itu melakukan export proxy pada setiap UML dengan sintaks seperti di bawah ini:
```
export http_proxy=”http://DPTSI-563019-18a27:0be0c@proxy.its.ac.id:8080”
export https_proxy=”http://DPTSI-563019-18a27:0be0c@proxy.its.ac.id:8080”
export ftp_proxy=”http://DPTSI-563019-18a27:0be0c@proxy.its.ac.id:8080”
```
Serta memberikan perintah ```apt-get update``` juga pada setiap UML untuk melakukan update.

Setelah itu dapat dimulai pengerjaan Soal Shift.

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

1. Membuat sebuah alamat http://semeruyyy.pw
**Pada UML MALANG**
- Ketikkan ```apt-get install bind9 -y```
- Kemudian ```nano /etc/bind/named.conf.local```
- Tambahkan kode:
```
zone "semerud07.pw" {
    type master;
    file "/etc/bind/jarkom/semerud07.pw";
};
```

![image](https://user-images.githubusercontent.com/48936125/98771132-53dffa00-2416-11eb-871b-3f3bafcc9b7c.png)

- Kemudian buat folder baru: ```mkdir /etc/bind/jarkom```
- Dan copy file db.local ke folder yang baru saja dibuat dan mengganti namanya sesuai domain yang diinginkan: ```cp /etc/bind/db.local /etc/bind/jarkom/semerud07.pw```
- Kemudian buka file ```semerud07.pw``` dengan perintah: ```nano /etc/bind/jarkom/semerud07.pw```.

![image](https://user-images.githubusercontent.com/48936125/98771180-6823f700-2416-11eb-8b73-66a11e185279.png)

- Kemudian setelah itu melakukan restart service bind9 dengan perintah ```service bind9 restart```.

**Pada UML GRESIK**
- Awalnya perlu dilakukan perubahan setting nameserver pada client yang akan dicoba (GRESIK), agar mengarah pada IP MALANG, dengan cara mengedit file ```resolv.conf``` dengan mengetikkan perintah ```nano /etc/resolv.conf``` dan mengubah menjadi file tersebut:
```
#nameserver 202.46.129.2
#nameserver 10.151.36.7

nameserver 10.151.79.66
```

- Kemudian, untuk mengecek apakah domain ```semerud07.pw``` dapat diakses, mengetikkan perintah ```ping semerud07.pw``` pada UML GRESIK.

![image](https://user-images.githubusercontent.com/48936125/98771283-a8837500-2416-11eb-9bc6-cc1d48e7d1be.png)

2. Memberikan alias http://www.semeruyyy.pw
**Pada UML MALANG**
- Ketikkan ```nano /etc/bind/jarkom/semerud07.pw``` untuk mengedit file tersebut, ditambahkan perintah seperti gambar di bawah:

![image](https://user-images.githubusercontent.com/48936125/98771391-eed8d400-2416-11eb-96f5-21ce60e18dd7.png)

- Kemudian setelah itu melakukan restart service bind9 dengan perintah ```service bind9 restart```.

**Pada UML GRESIK**
- Untuk mengecek apakah domain ```www.semerud07.pw``` dapat diakses, mengetikkan perintah ```ping www.semerud07.pw``` pada UML GRESIK.

![image](https://user-images.githubusercontent.com/48936125/98771515-31021580-2417-11eb-9f32-f4aa346e599f.png)

3. Membuat subdomain http://penanjakan.semeruyyy.pw yang diatur DNS-nya pada MALANG dan mengarah ke IP Server PROBOLINGGO
**Pada UML MALANG**
- Ketikkan ```nano /etc/bind/jarkom/semerud07.pw``` untuk mengedit file tersebut, ditambahkan perintah seperti gambar di bawah:

![image](https://user-images.githubusercontent.com/48936125/98771594-67d82b80-2417-11eb-9633-f25f4f58ebe0.png)

- Kemudian setelah itu melakukan restart service bind9 dengan perintah ```service bind9 restart```.

**Pada UML GRESIK**
- Untuk mengecek apakah domain ```penanjakan.semerud07.pw``` dapat diakses, mengetikkan perintah ```ping penanjakan.semerud07.pw``` pada UML GRESIK.

![image](https://user-images.githubusercontent.com/48936125/98771632-7b839200-2417-11eb-9f01-0e567b20e074.png)

4. Membuat reverse domain untuk domain utama
- Mengedit file /etc/bind/named.conf.local dengan perintah ```nano /etc/bind/named.conf.local```, lalu menambahkan konfigurasi seperti pada gambar: 

![image](https://user-images.githubusercontent.com/48936125/98771772-c6050e80-2417-11eb-9ee4-656fb9732606.png)
- Meng-copy file ```db.local``` pada path ```/etc/bind``` ke dalam folder jarkom yang baru saja dibuat dan ubah namanya menjadi ```79.151.10.in-addr.arpa``` dengan perintah ```cp /etc/bind/db.local /etc/bind/jarkom/79.151.10.in-addr.arpa```. Keterangan 79.151.10 adalah 3 byte pertama IP MALANG yang dibalik urutan penulisannya.
- Kemudian, mengedit file ```79.151.10.in-addr.arpa``` dengan perintah menjadi seperti gambar di bawah ini:

![image](https://user-images.githubusercontent.com/48936125/98771888-095f7d00-2418-11eb-807b-d7d7bc7f4e4d.png)

- Kemudian setelah itu melakukan restart service bind9 dengan perintah ```service bind9 restart```.

**Pada UML GRESIK**
- Kemudian untuk mengecek apakah konfigurasi sudah benar, mengetikkan perintah ini pada UML GRESIK untuk melakukan update (harus mengembalikan nameserver default terlebih dahulu agar update berhasil), kemudian mengembalikannya lagi ke IP MALANG.
```
apt-get update
apt-get install dnsutils

host -t PTR 10.151.79.66
```
- Maka akan terlihat bahwa konfigurasi reverse sudah berhasil:

![image](https://user-images.githubusercontent.com/48936125/98772265-f13c2d80-2418-11eb-8a7f-c2798080e17d.png)

5. 

