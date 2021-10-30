# Praktikum Modul 2

- Andika Nugrahanto (0511194000031)
- Muhammad Rayhan Raffi Pratama (05111940000110)
- Fadhil Dimas Sucahyo (05111940000212)

## Pendahuluan

### Setting Topologi

![0.1](imgs/0.1.JPG)

### Edit Konfigurasi Network

#### Foosha (Router)

![0.2](imgs/0.2.JPG)

#### EniesLobby (DNS Master)

![0.3](imgs/0.3.JPG)

#### Water7 (DNS Slave)

![0.4](imgs/0.4.JPG)

#### Skypie (Web Server)

![0.5](imgs/0.5.JPG)

#### Loguetown (Client)

![0.6](imgs/0.6.JPG)

#### Alabasta (Client)

![0.7](imgs/0.7.JPG)

## no. 1

EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet

### Jawab

Menjalankan command `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.194.0.0/16` yang digunakan supaya dapat terhubung ke jaringan luar pada router `Foosha`
![1.1](imgs/1.1.JPG)

Kemudian menambahkan command `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.194.0.0/16` ke `/root/.bashrc` agar dijalankan setiap kali project distart dengan command `echo "iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.194.0.0/16" >> /root/.bashrc`
![1.2](imgs/1.2.JPG)

Setelah itu pada semua node lainnya ditambahkan command `echo "nameserver 192.168.122.0"` untuk setting IP DNS ke `/root/.bashrc` agar dijalankan setiap kali project distart dengan command `echo 'echo "nameserver 192.168.122.1" > /etc/resolv.conf' >> /root/.bashrc`
![1.3](imgs/1.3.JPG)

## no. 2

membuat website utama dengan mengakses franky.yyy.com dengan alias www.franky.yyy.com pada folder kaizoku

### Jawab

pada EniesLobby jalankan command `apt-get update` dan `apt-get install bind9 -y` untuk menginstall bind9
![2.1](imgs/2.1.JPG)

Kemudian mengedit `/etc/bind/named.conf.local` dengan menambahkan :

```bash
    zone "franky.d05.com" {
            type master;
            file "/etc/bind/kaizoku/franky.d05.com";
    };
```

![2.2](imgs/2.2.JPG)

lalu dibuat folder kaizoku di `/etc/bind`. Lalu copy `/etc/bind/db.local` menjadi `/etc/bind/kaizoku/franky.d05.com`. Lalu konfigurasi file tersebut agar memiliki SOA `franky.d05.com.`, NS `franky.d05.com.`, record A yang mengarah ke `IP Skypie`, dan CNAME `www` pada `franky.d05.com.`.
![2.3](imgs/2.3.JPG)
![2.4](imgs/2.4.JPG)

## no. 3

Setelah itu buat subdomain super.franky.yyy.com dengan alias www.super.franky.yyy.com yang diatur DNS nya di EniesLobby dan mengarah ke Skypie.

### Jawab

Mengedit file `/etc/bind/kaizoku/franky.d05.com` dengan menambahkan:

```bash
        super   IN      A       192.194.2.4
        www.super       IN      CNAME   super.franky.d05.com.
```

![3.1](imgs/3.1.JPG)

## no. 4

Buat juga reverse domain untuk domain utama

### jawab

Pertama, menambahkan zone pada `/etc/bind/named.conf.local` dengan menambahkan:

```bash
    zone "2.194.192.in-addr.arpa" {
            type master;
            file "/etc/bind/kaizoku/2.194.192.in-addr.arpa";
    };
```

![4.1](imgs/4.1.JPG)

Lalu copy `/etc/bind/db.local` menjadi `/etc/bind/kaizoku/2.194.192.in-addr.arpa`. Lalu konfigurasi file tersebut agar memiliki SOA `franky.d05.com.`,`2.194.192.in-addr.arpa.` yang memiliki NS `franky.d05.com.`, dan `4` yang merupakan byte ke-4 IP EniesLobby memiliki PTR `franky.d05.com.`.

![4.2](imgs/4.2.JPG)

## no. 5

Supaya tetap bisa menghubungi Franky jika server EniesLobby rusak, maka buat Water7 sebagai DNS Slave untuk domain utama

### jawab

Di EniesLobby:

Pertama edit zone `franky.d05.com` pada `/etc/bind/named.conf.local` menjadi:

```bash
    zone "franky.d05.com" {
            type master;
            notify yes;
            also-notify { 192.194.2.3; };
            allow-transfer { 192.194.2.3; };
            file "/etc/bind/kaizoku/franky.d05.com";
    };
```

![5.1](imgs/5.1.JPG)

Di Water7:
Jalankan command `apt-get update` dan `apt-get install bind9 -y` untuk menginstall bind9

Kemudian mengedit `/etc/bind/named.conf.local` dengan menambahkan :

```bash
    zone "franky.d05.com" {
        type slave;
        masters { 192.194.2.2; };
        file "/var/lib/bind/franky.d05.com";
    };
```

![5.2](imgs/5.2.JPG)

## no. 6

Setelah itu terdapat subdomain mecha.franky.yyy.com dengan alias www.mecha.franky.yyy.com yang didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder sunnygo.

### Jawab

Di EniesLobby:

Pertama, mengedit file `/etc/bind/kaizoku/franky.d05.com` dengan menambahkan:

```bash
        ns1     IN      A       192.194.2.3 ; IP Water7
        mecha   IN      NS      ns1
```

![6.1](imgs/6.1.JPG)

Kemudian mengedit file `/etc/bind/named.conf.options` dengan comment bagian `dnssec-validation auto;` dan menambahkan line `allow-query{any;};`.

![6.2](imgs/6.2.JPG)

Pastikan sudah ada line `allow-transfer { "IP Water7"; };` pada zone `franky.d05.com` di file `/etc/bind/named.conf.local`.

![6.3](imgs/6.3.JPG)

Di Water7:

Pertama edit file `/etc/bind/named.conf.options` dengan comment bagian `dnssec-validation auto;` dan menambahkan line `allow-query{any;};`.

![6.2](imgs/6.2.JPG)

Kemudian menambahkan zone pada `/etc/bind/named.conf.local` dengan menambahkan:

```bash
    zone "mecha.franky.d05.com" {
            type master;
            file "/etc/bind/sunnygo/mecha.franky.d05.com";
    };
```

![6.4](imgs/6.4.JPG)

lalu dibuat folder sunnygo di `/etc/bind`. Lalu copy `/etc/bind/db.local` menjadi `/etc/bind/sunnygo/mecha.franky.d05.com`. Lalu konfigurasi file tersebut agar memiliki SOA `mecha.franky.d05.com.`, NS `mecha.franky.d05.com.`, record A yang mengarah ke `IP Skypie`, dan CNAME `www` pada `mecha.franky.d05.com.`.

![6.5](imgs/6.5.JPG)
![6.6](imgs/6.6.JPG)

## no. 7

Untuk memperlancar komunikasi Luffy dan rekannya, dibuatkan subdomain melalui Franky dengan nama general.mecha.franky.yyy.com dengan alias www.general.mecha.franky.yyy.com yang mengarah ke Skypie.

### Jawab

Di Water7:

Mengedit file `/etc/bind/sunnygo/mecha.franky.d05.com` dengan menambahkan:

```bash
        general IN      A       192.194.2.4 ; IP Skypie
        www.general     IN      CNAME   general.mecha.franky.d05.com.
```

![7.1](imgs/7.1.JPG)

## no. 8

Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.franky.yyy.com. Pertama, luffy membutuhkan webserver dengan DocumentRoot pada /var/www/franky.yyy.com.

Di Skypie:
Pertama, install `apache2`, `php`, dan `libapache2-mod-php7.0`
![8.1](imgs/8.1.JPG)

Kemudian melakukan `wget` untuk mendownload file yang diperlukan. Setelah itu diunzip sehinggan menampilkan folder-folder seperti ini.

![8.2](imgs/8.2.JPG)

Setelah itu, pindah ke directory `/etc/apache2/sites-available`.Kemudian copy file `000-default.conf` menjadi file `franky.d05.com.conf`

![8.3](imgs/8.3.JPG)

Lalu setting file `franky.d05.com.conf` agar memiliki line `ServerName franky.d05.com`, `ServerAlias www.franky.d05.com`, dan `DocumentRoot /var/www/franky.d05.com`.

![8.4](imgs/8.4.JPG)

Kemudian bikin directory baru dengan nama `franky.d05.com` pada `/var/www/` menggunakan command `mkdir /var/www/franky.d05.com`. lalu copy isi dari folder `franky` yang telah didownload ke `/var/www/franky.d05.com`.

Setelah itu jalankan command `a2ensite franky.d05.com` dan `service apache2 restart`
![8.5](imgs/8.5.JPG)

Ketika menjalankan command `lynx www.franky.d05.com` pada client akan muncul halaman
![8.6](imgs/8.6.JPG)

## no. 9

Setelah itu, Luffy juga membutuhkan agar url www.franky.yyy.com/index.php/home dapat menjadi menjadi www.franky.yyy.com/home.

### Jawab

Pertama, jalankan perintah `a2enmod rewrite` kemudian `service apache2 restart`.

Kemudian pindah ke directory `/var/www/franky.d05.com` dan buat file `.htaccess` dengan isi file:

```bash
    RewriteEngine On
    RewriteRule ^home$ index.php/home
```

![9.1](imgs/9.1.JPG)

Kemudian buka file `/etc/apache2/sites-available/franky.d05.com.conf` dan tambahkan:

```bash
    <Directory /var/www/franky.d05.com>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>
```

![9.2](imgs/9.2.JPG)

Ketika menjalankan command `lynx www.franky.d05.com/home` pada client akan muncul halaman
![9.3](imgs/9.3.JPG)

## no. 10

Setelah itu, pada subdomain www.super.franky.yyy.com, Luffy membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/super.franky.yyy.com

### Jawab

Pertama, pindah ke directory `/etc/apache2/sites-available`.Kemudian copy file `000-default.conf` menjadi file `super.franky.d05.com.conf`

![10.1](imgs/10.1.JPG)

Lalu setting file `super.franky.d05.com.conf` agar memiliki line `ServerName super.franky.d05.com`, `ServerAlias www.super.franky.d05.com`, dan `DocumentRoot /var/www/super.franky.d05.com`.

![10.2](imgs/10.2.JPG)

Kemudian bikin directory baru dengan nama `super.franky.d05.com` pada `/var/www/` menggunakan command `mkdir /var/www/super.franky.d05.com`. lalu copy isi dari folder `super.franky` yang telah didownload ke `/var/www/super.franky.d05.com`.

Setelah itu jalankan command `a2ensite super.franky.d05.com` dan `service apache2 restart`
![10.3](imgs/10.3.JPG)

Ketika menjalankan command `lynx www.super.franky.d05.com` pada client akan muncul halaman
![10.4](imgs/10.4.JPG)

## no. 11

Akan tetapi, pada folder /public, Luffy ingin hanya dapat melakukan directory listing saja.

### Jawab

Pindah ke directory `/etc/apache2/sites-available` kemudian buka file `super.franky.d05.com` dan tambahkan:

```bash
    <Directory /var/www/super.franky.d05.com/public>
        Options +Indexes
    </Directory>
```

![11.1](imgs/11.1.JPG)

Ketika menjalankan command `lynx www.super.franky.d05.com/public` pada client akan muncul halaman
![11.2](imgs/11.2.JPG)

## no. 12

Tidak hanya itu, Luffy juga menyiapkan error file 404.html pada folder /errors untuk mengganti error kode pada apache .

### Jawab

Pindah ke directory `/etc/apache2/sites-available` kemudian buka file `super.franky.d05.com` dan tambahkan:

```bash
        ErrorDocument 404 /error/404.html
```

![12.1](imgs/12.1.JPG)

Ketika menjalankan command `lynx www.super.franky.d05.com/publi`(lokasi yang tidak ada) pada client akan muncul halaman
![12.2](imgs/12.2.JPG)
