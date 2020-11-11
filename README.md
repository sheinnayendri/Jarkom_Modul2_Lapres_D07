# Jarkom_Modul2_Praktikum_D07
Laporan Resmi Modul 2 Praktikum Jaringan Komputer 2020
#
1. Naufal Adam Kuncoro (05111740000155)
2. Sheinna Yendri (05111840000038)
#

## Setting Topologi
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

## Pengerjaan Soal
1. [Soal1](#soal1)
2. [Soal2](#soal2)
3. [Soal3](#soal3)
4. [Soal4](#soal4)
5. [Soal5](#soal5)
6. [Soal6](#soal6)
7. [Soal7](#soal7)
8. [Soal8](#soal8)
9. [Soal9](#soal9)
10. [Soal10](#soal10)
11. [Soal11](#soal11)
12. [Soal11](#soal12)
13. [Soal11](#soal13)
14. [Soal11](#soal14)
15. [Soal11](#soal15)
16. [Soal11](#soal16)
17. [Soal11](#soal17)
#

### Soal1
#### Membuat sebuah alamat http://semeruyyy.pw
#
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

### Soal2
#### Memberikan alias http://www.semeruyyy.pw
#
**Pada UML MALANG**
- Ketikkan ```nano /etc/bind/jarkom/semerud07.pw``` untuk mengedit file tersebut, ditambahkan perintah seperti gambar di bawah:

![image](https://user-images.githubusercontent.com/48936125/98771391-eed8d400-2416-11eb-96f5-21ce60e18dd7.png)

- Kemudian setelah itu melakukan restart service bind9 dengan perintah ```service bind9 restart```.

**Pada UML GRESIK**
- Untuk mengecek apakah domain ```www.semerud07.pw``` dapat diakses, mengetikkan perintah ```ping www.semerud07.pw``` pada UML GRESIK.

![image](https://user-images.githubusercontent.com/48936125/98771515-31021580-2417-11eb-9f32-f4aa346e599f.png)

### Soal3
#### Membuat subdomain http://penanjakan.semeruyyy.pw yang diatur DNS-nya pada MALANG dan mengarah ke IP Server PROBOLINGGO
#
**Pada UML MALANG**
- Ketikkan ```nano /etc/bind/jarkom/semerud07.pw``` untuk mengedit file tersebut, ditambahkan perintah seperti gambar di bawah:

![image](https://user-images.githubusercontent.com/48936125/98771594-67d82b80-2417-11eb-9633-f25f4f58ebe0.png)

- Kemudian setelah itu melakukan restart service bind9 dengan perintah ```service bind9 restart```.

**Pada UML GRESIK**
- Untuk mengecek apakah domain ```penanjakan.semerud07.pw``` dapat diakses, mengetikkan perintah ```ping penanjakan.semerud07.pw``` pada UML GRESIK.

![image](https://user-images.githubusercontent.com/48936125/98771632-7b839200-2417-11eb-9f01-0e567b20e074.png)

### Soal4
#### Membuat reverse domain untuk domain utama
#
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

### Soal5
#### Membuatkan DNS Server Slave pada MOJOKERTO agar Bibah tidak terganggu menikmati keindahan Semeru pada Website.
#
**Pada UML MALANG**
- Mengedit file ```/etc/bind/named.conf.local``` menjadi:

![image](https://user-images.githubusercontent.com/48936125/98772952-75db7b80-241a-11eb-887e-be9c28534004.png)

- Kemudian setelah itu melakukan restart service bind9 dengan perintah ```service bind9 restart```.

**Pada UML MOJOKERTO**
- Melakukan update dengan perintah ```apt-get update``` dan install bind9 dengan ```apt-get install bind9 -y```. Kemudian buka file ```/etc/bind/named.conf.local``` pada MOJOKERTO dan tambahkan syntax seperti pada gambar:

![image](https://user-images.githubusercontent.com/48936125/98773059-b89d5380-241a-11eb-8072-494d321429b9.png)

- Kemudian setelah itu melakukan restart service bind9 dengan perintah ```service bind9 restart```.

**Pada UML MALANG**
- Untuk mengecek apakah konfigurasi berhasil, maka service pada UML MALANG akan dihentikan dengan perintah ```service bind9 stop```:

![image](https://user-images.githubusercontent.com/48936125/98773144-dd91c680-241a-11eb-846f-0cafd1974271.png)

**Pada UML GRESIK**
- Pada file ```/etc/bind/resolv.conf``` perlu ditambahkan nameserver yang mengarah ke IP MOJOKERTO juga, yaitu menambahkan ```nameserver 10.151.79.67```.
- Kemudian untuk mengecek apakah DNS Slave sudah berhasil, dilakukan perintah ```ping semerud07.pw```. Karena ping berhasil, berarti konfigurasi DNS Slave berhasil:

![image](https://user-images.githubusercontent.com/48936125/98773307-3d886d00-241b-11eb-94d0-e4cfdba6671f.png)

### Soal6
#### Membuatkan subdomain dengan alamat http://gunung.semeruyyy.pw yang didelegasikan pada server MOJOKERTO dan mengarah ke IP Server PROBOLINGGO.
#
**Pada UML MALANG**
- Mengedit file ```/etc/bind/jarkom/semerud07.pw``` dan mengubah menjadi seperti di bawah ini:

![image](https://user-images.githubusercontent.com/48936125/98796309-d16b3080-243d-11eb-9c7a-5ac9e78db858.png)

- Kemudian edit file ```/etc/bind/named.conf.options``` dengan perintah ```nano /etc/bind/named.conf.options```.
- Kemudian comment ```dnssec-validation auto;``` dan menambahkan baris berikut pada ```/etc/bind/named.conf.options```: ```allow-query{any;};```.

![image](https://user-images.githubusercontent.com/48936125/98796514-0b3c3700-243e-11eb-9deb-8cb1baeab3f7.png)

- Lalu mengedit file ```/etc/bind/named.conf.local``` menjadi seperti gambar di bawah:

![image](https://user-images.githubusercontent.com/48936125/98796668-37f04e80-243e-11eb-8140-2145351c0d82.png)

- Setelah itu melakukan restart service bind9 dengan perintah ```service bind9 restart```.

**Pada UML MOJOKERTO**
- Mengedit ```file /etc/bind/named.conf.options``` dengan perintah ```nano /etc/bind/named.conf.options```.
- Kemudian comment ```dnssec-validation auto;``` dan tambahkan baris berikut pada ```/etc/bind/named.conf.options```: ```allow-query{any;};```.

![image](https://user-images.githubusercontent.com/48936125/98796913-84d42500-243e-11eb-99ee-5d6120474489.png)

- Lalu mengedit file ```/etc/bind/named.conf.local``` menjadi seperti gambar di bawah:

![image](https://user-images.githubusercontent.com/48936125/98797001-9b7a7c00-243e-11eb-80d7-32f763163245.png)

- Kemudian membuat direktori dengan nama delegasi, dan mengcopy ```db.local``` ke direktori baru dan mengedit namanya menjadi ```gunung.semerud07.pw```, dengan perintah:
```
mkdir /etc/bind/delegasi
cp /etc/bind/db.local /etc/bind/delegasi/gunung.semerud07.pw
```
- Lalu mengedit file ```gunung.semerud07.pw``` menjadi seperti dibawah ini:

![image](https://user-images.githubusercontent.com/48936125/98797207-dd0b2700-243e-11eb-9ce2-e27c8a380c5d.png)

- Setelah itu melakukan restart service bind9 dengan perintah ```service bind9 restart```.

**Pada UML GRESIK**
- Kemudian untuk melakukan testing, maka akan digunakan perintah ```ping gunung.semerud07.pw``` pada UML GRESIK. Jika berhasil, berarti konfigurasi berhasil:

![image](https://user-images.githubusercontent.com/48936125/98797799-a4b81880-243f-11eb-977a-c536ecefc8b8.png)

### Soal7
#### Membuat subdomain dengan nama http://naik.gunung.semeruyyy.pw, domain ini diarahkan ke IP Server PROBOLINGGO.
#
**Pada UML MOJOKERTO**
- Mengedit file ```/etc/bind/delegasi/gunung.semerud07.pw dan menambahkan seperti konfigurasi di gambar:

![image](https://user-images.githubusercontent.com/48936125/98799245-71768900-2441-11eb-8a66-78db1157dd7c.png)

- Kemudian melakukan restart service bind9 dengan perintah ```service bind9 restart```.

**Pada UML GRESIK**
- Untuk mengecek apakah konfigurasi sudah benar, dilakukan ```ping naik.gunung.semerud07.pw```, karena berhasil berarti konfigurasi sudah benar:

![image](https://user-images.githubusercontent.com/48936125/98799384-9cf97380-2441-11eb-87bc-faf589d8ba9f.png)

### Soal8
#### Setelah selesai membuat keseluruhan domain, kamu diminta untuk segera mengatur web server. Domain http://semeruyyy.pw memiliki DocumentRoot pada /var/www/semeruyyy.pw.
#
**Pada UML PROBOLINGGO**
- Pertama-tama melakukan instalasi Apache2 dan PHP5 dengan perintah ```apt-get install apache2``` dan ```apt-get install php5```.
- Setelah terinstall, maka akan pindah ke direktori ```/etc/apache2/sites-available``` untuk membuat sebuah web server baru dengan nama ```semerud07.pw```.
- Berpindah dengan command ```cd /etc/apache2/sites-available``` dan melakukan copy file default menjadi file baru bernama domain yang ingin kita buat dengan command ```cp default semerud07.pw```.
- Kemudian kita melakukan edit file ```semerud07.pw``` dengan command ```nano semerud07.pw``` dan mengubah DocumentRoot serta menambahkan ServerName dan ServerAlias seperti gambar di bawah:

![image](https://user-images.githubusercontent.com/48936125/98800027-6ff99080-2442-11eb-8849-13bcfd1cd6a8.png)

- Kemudian kita melakukan restart apache, menggunakan command ```service apache2 restart```.
- Lalu kita berpindah direktori ke ```cd /var/www/``` yaitu tempat isi dari web server kita. Di sini membuat folder baru dengan nama ```semerud07.pw```. Kemudian kita cd ke folder baru ini.
- Di folder ```/var/www/semerud07.pw``` ini kita akan mendownload file zip yang diberikan dengan command ```wget 10.151.36.202/semeru.pw.zip```. Kemudian kita unzip (install terlebih dahulu). Setelah itu semua isi file zip akan kita pindah ke direktori tempat kita berada yaitu ```/var/www/semerud07.pw```:

![image](https://user-images.githubusercontent.com/48936125/98800325-d1216400-2442-11eb-9b39-974a07cef60f.png)

**Pada Browser**
- Maka ketika domain ```semerud07.pw``` kita ketikkan pada browser akan terlihat seperti gambar:

![image](https://user-images.githubusercontent.com/48936125/98800380-e1394380-2442-11eb-9d66-32c6a01a66ff.png)

### Soal9
#### Mengubah agar alamat http://semeruyyy.pw/index.php/home urlnya berubah menjadi http://semeruyyy.pw/home
#
**Pada UML PROBOLINGGO**
- Kita kembali ke direktori: ```cd /etc/apache2/sites-available``` dan mengedit file: ```nano semerud07.pw```, ditambahkan alias seperti gambar di bawah:

![image](https://user-images.githubusercontent.com/48936125/98800651-4725cb00-2443-11eb-9bb4-6d6aa88d3e70.png)

- Kemudian kita melakukan restart apache, menggunakan command ```service apache2 restart```.

**Pada Browser**
- Maka ketika domain ```semerud07.pw/home``` kita ketikkan pada browser akan terlihat seperti gambar:

![image](https://user-images.githubusercontent.com/48936125/98801455-653ffb00-2444-11eb-90eb-b23cb4cb3dbd.png)

### Soal10
#### Web http://penanjakan.semeruyyy.pw akan digunakan untuk menyimpan assets file yang memiliki DocumentRoot pada /var/www/penanjakan.semeruyyy.pw dan memiliki struktur folder sebagai berikut:
```
    /var/www/penanjakan.semeruyyy.pw
    /public/javascripts
    /public/css
    /public/images
```
#
**Pada UML PROBOLINGGO**
- Berpindah direktori dengan command ```cd /etc/apache2/sites-available``` dan melakukan copy file default menjadi file baru bernama domain yang ingin kita buat dengan command ```cp default penanjakan.semerud07.pw```.
- Kemudian kita melakukan edit file ```penanjakan.semerud07.pw``` dengan command ```nano penanjakan.semerud07.pw``` dan mengubah DocumentRoot serta menambahkan ServerName dan ServerAlias seperti gambar di bawah:

![image](https://user-images.githubusercontent.com/48936125/98801698-c36cde00-2444-11eb-8633-e2e18cf08145.png)

- Kemudian kita melakukan restart apache, menggunakan command ```service apache2 restart```.
- Lalu kita berpindah direktori ke ```cd /var/www/``` yaitu tempat isi dari web server kita. Di sini membuat folder baru dengan nama ```penanjakan.semerud07.pw```. Kemudian kita cd ke folder baru ini.
- Di folder ```/var/www/penanjakan.semerud07.pw``` ini kita akan mendownload file zip yang diberikan dengan command ```wget 10.151.36.202/penanjakan.semeru.pw.zip```. Kemudian kita unzip. Setelah itu semua isi file zip akan kita pindah ke direktori tempat kita berada yaitu ```/var/www/penanjakan.semerud07.pw```:

![image](https://user-images.githubusercontent.com/48936125/98801839-fa42f400-2444-11eb-81f9-d1a024441b65.png)

**Pada Browser**
- Maka ketika domain ```penanjakan.semerud07.pw``` kita ketikkan pada browser akan terlihat seperti gambar:

![image](https://user-images.githubusercontent.com/48936125/98801879-0e86f100-2445-11eb-9b51-02a2e56deeb6.png)

### Soal11
#### Pada folder /public dibolehkan directory listing namun untuk folder yang berada di dalamnya tidak dibolehkan (penanjakan.semerud07.pw).
#
**Pada UML PROBOLINGGO**
- Berpindah direktori dengan command ```cd /etc/apache2/sites-available``` dan melakukan edit file ```penanjakan.semerud07.pw``` dengan command ```nano penanjakan.semerud07.pw``` dan menambahkan konfigurasi seperti gambar berikut:

![image](https://user-images.githubusercontent.com/48936125/98802078-51e15f80-2445-11eb-9de2-f3d1c24873c6.png)

- Kemudian kita melakukan restart apache, menggunakan command ```service apache2 restart```.

**Pada Browser**
- Maka ketika domain ```penanjakan.semerud07.pw/public``` kita ketikkan di browser, kita dapat melakukan directory listing:

![image](https://user-images.githubusercontent.com/48936125/98801879-0e86f100-2445-11eb-9b51-02a2e56deeb6.png)

- Sedangkan ketika kita mengakses domain ```penanjakan.semerud07.pw/public/css``` ataupun ```penanjakan.semerud07.pw/public/javascripts``` atau ```penanjakan.semerud07.pw/public/images```:

![image](https://user-images.githubusercontent.com/48936125/98802614-227f2280-2446-11eb-84c2-42caaf3e2101.png)

![image](https://user-images.githubusercontent.com/48936125/98802646-2a3ec700-2446-11eb-9877-1d78972e4f58.png)

![image](https://user-images.githubusercontent.com/48936125/98802677-32970200-2446-11eb-9769-daef68e41439.png)

### Soal12
#### Untuk mengatasi HTTP Error code 404, disediakan file 404.html pada folder /errors untuk mengganti error default 404 dari Apache.
#
**Pada UML PROBOLINGGO**
- Berpindah direktori dengan command ```cd /etc/apache2/sites-available``` dan melakukan edit file ```penanjakan.semerud07.pw``` dengan command ```nano penanjakan.semerud07.pw``` dan menambahkan konfigurasi seperti gambar berikut:
```
ErrorDocument 404 /errors/404.html
```

![image](https://user-images.githubusercontent.com/48936125/98803791-da60ff80-2447-11eb-92e9-ae36172f6251.png)

- Kemudian kita melakukan restart apache, menggunakan command ```service apache2 restart```.

**Pada Browser**
- Maka ketika kita mengakses domain ```penanjakan.semerud07.pw/publoc``` (terdapat **typo**), yang muncul ada file 404.html dari folder /errors:

![image](https://user-images.githubusercontent.com/48936125/98803566-90781980-2447-11eb-8a92-8af49704ba3b.png)

### Soal13
#### Untuk mengakses file assets javascript awalnya harus menggunakan url http://penanjakan.semeruyyy.pw/public/javascripts. Karena terlalu panjang maka dibuatkan konfigurasi virtual host agar ketika mengakses file assets menjadi http://penanjakan.semeruyyy.pw/js.
#
**Pada UML PROBOLINGGO**
- Berpindah direktori dengan command ```cd /etc/apache2/sites-available``` dan melakukan edit file ```penanjakan.semerud07.pw``` dengan command ```nano penanjakan.semerud07.pw``` dan menambahkan konfigurasi seperti gambar berikut (alias):

![image](https://user-images.githubusercontent.com/48936125/98803711-c2897b80-2447-11eb-9e2a-8c1fcb233c17.png)

### Soal14
#### Web http://naik.gunung.semeruyyy.pw sudah bisa diakses hanya dengan menggunakan port 8888. DocumentRoot web berada pada /var/www/naik.gunung.semeruyyy.pw.
#
**Pada UML PROBOLINGGO**
- Berpindah direktori dengan command ```cd /etc/apache2/sites-available``` dan melakukan copy file default menjadi file baru bernama domain yang ingin kita buat dengan command ```cp default naik.gunung.semerud07.pw```.
- Kemudian kita melakukan edit file ```penanjakan.semerud07.pw``` dengan command ```nano penanjakan.semerud07.pw``` dan mengubah VirtualHost serta DocumentRoot serta menambahkan ServerName dan ServerAlias seperti gambar di bawah:

![image](https://user-images.githubusercontent.com/48936125/98804196-70952580-2448-11eb-930f-3c59a6e1f56f.png)

- Kemudian kita melakukan restart apache, menggunakan command ```service apache2 restart```.
- Lalu kita berpindah direktori ke ```cd /var/www/``` yaitu tempat isi dari web server kita. Di sini membuat folder baru dengan nama ```naik.gunung.semerud07.pw```. Kemudian kita cd ke folder baru ini.
- Di folder ```/var/www/naik.gunung.semerud07.pw``` ini kita akan mendownload file zip yang diberikan dengan command ```wget 10.151.36.202/naik.gunung.semeru.pw.zip```. Kemudian kita unzip. Setelah itu semua isi file zip akan kita pindah ke direktori tempat kita berada yaitu ```/var/www/naik.gunung.semerud07.pw```:

![image](https://user-images.githubusercontent.com/48936125/98804258-89054000-2448-11eb-95d3-426393699f3c.png)

- Kemudian kita juga harus mengaktifkan port 8888 dengan mengedit file: ```nano /etc/apache2/ports.conf``` menjadi:

![image](https://user-images.githubusercontent.com/48936125/98804996-88b97480-2449-11eb-8c0a-af35d3e45868.png)

- Unutuk mengaktifkan maka akan dijalankan perintah:
```
a2ensite naik.gunung.semerud07.pw
service apache2 restart
```

- Maka domain ```naik.gunung.semerud07.pw:8888``` akan dapat diakses pada browser.

### Soal15
#### Dikarenakan web http://naik.gunung.semeruyyy.pw bersifat private, Bibah meminta kamu membuat web http://naik.gunung.semeruyyy.pw agar diberi autentikasi password dengan username “semeru” dan password “kuynaikgunung” supaya aman dan tidak sembarang orang bisa mengaksesnya.
#
**Pada UML PROBOLINGGO**
- Mengedit file: ```nano /etc/apache2/jarkom/naik.gunung.semerud07.pw``` dan menambahkan konfigurasi:

![image](https://user-images.githubusercontent.com/48936125/98805309-f796cd80-2449-11eb-8ac5-a79465eb9e86.png)

- Sebelumnya, akan dibuat akun autentikasi baru dengan cara melakukan instalasi terlebih dahulu: 
```
apt-get update
apt-get install apache2-utils
```

- Kemudian menggunakan command ```htpasswd -c /etc/apache2/.htpasswd semeru``` yang berarti ingin membuat akun autentikasi baru dengan username **semeru**. Kita akan diminta untuk memasukkan password baru dan confirm password tersebut (diisi **kuynaikgunung**):

![image](https://user-images.githubusercontent.com/48936125/98805750-a1765a00-244a-11eb-97ff-1d445c310548.png)

- Lalu service akan direstart dengan perintah: ```service apache2 restart```.

**Pada Browser**
- Ketika kita mengakses domain ```naik.gunung.semerud07.pw:8888``` maka akan diminta untuk memasukkan username dan password terlebih dahulu:

![image](https://user-images.githubusercontent.com/48936125/98805870-db476080-244a-11eb-9f51-5a5ae41eac65.png)

![image](https://user-images.githubusercontent.com/48936125/98805893-e3070500-244a-11eb-80f4-499d8a6a19e7.png)

![image](https://user-images.githubusercontent.com/48936125/98805904-e8644f80-244a-11eb-802d-69bb48b82258.png)

### Soal16
#### 
