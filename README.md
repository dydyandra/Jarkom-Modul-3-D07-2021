# Jarkom-Modul-3-D07-2021
### Anggota kelompok:
Anggota | NRP | 
------------- | ------------- | 
Amanda Rozi Kurnia | 05111940000094 | 
Dyandra Paramitha W. | 05111940000119 |
Daanii Nabil Ghinannafsi Kusnanta | 05111940000163 |

## Notes
[Soal Shift 3](https://docs.google.com/document/d/1hwuI5YpxiP-aboS7wGWPbaQeSOQl0HHVHLT3ws2BPUk/edit?usp=sharing) <br>
**Prefix:** 192.195


## Daftar Isi
* [no. 1](#no.-1)
* [no. 2](#no.-2)
* [no. 3](#no.-3)
* [no. 4](#no.-4)
* [no. 5](#no.-5)
* [no. 6](#no.-6)
* [no. 7](#no.-7)
* [no. 8](#no.-8)
* [no. 9](#no.-9)
* [no. 10](#no.-10)
* [no. 11](#no.-11)
* [no. 12](#no.-12)
* [no. 13](#no.-13)
* [no. 14](#no.-14)
* [no. 15](#no.-15)
* [no. 16](#no.-16)
* [no. 17](#no.-17)
* [Kendala Yang Dialami](#kendala)
* [Referensi](#referensi)


## Pendahuluan

### Setting Topologi

<image src="screenshots/1.PNG" width="700">

### Edit Knofigurasi Network

#### Pada Server
<image src="screenshots/0-1.PNG" width="700">
<image src="screenshots/0-2.PNG" width="700">
<image src="screenshots/0-3.PNG" width="700">
<image src="screenshots/0-4.PNG" width="700">


#### Pada Client (Loguetown, Alabasta, Tottoland, Skypie)

<image src="screenshots/0-5.PNG" width="700">

## no. 1

Luffy bersama Zoro berencana membuat peta tersebut dengan kriteria EniesLobby sebagai DNS Server, Jipangu sebagai DHCP Server, Water7 sebagai Proxy Server

### Jawab

#### Pendahuluan Konfigurasi

Menjalankan command `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.195.0.0/16` yang digunakan supaya dapat terhubung ke jaringan luar pada router `Foosha`

Setelah itu pada EniesLobby, Water7, Jipangu dijalankan command `echo "nameserver 192.168.122.1" > /etc/resolv.conf` untuk setting IP DNS agar dapat terhubung ke jaringan luar.

#### Instalasi 
Menjalani instalasi berikut pada setiap server:
```bash
# Pada Ennieslobby
apt-get update
apt-get install bind9 -y

#pada Jipangu
apt-get update
apt-get install isc-dhcp-server -y

#pada water7
apt-get update
apt-get install squid -y
```

#### Jipangu
Melakukan setting pada file `/etc/default/isc-dhcp-server` dengan menambahkan `eth0` pada `INTERFACES`
<image src="screenshots/1-1.PNG" width="700">

## no. 2

Foosha sebagai DHCP Relay

### Jawab

#### Foosha

Menjalankan command `apt-get update` dan `apt-get install isc-dhcp-relay -y` untuk menginstall isc-dhcp-relay
```bash
apt-get update
DEBIAN_FRONTEND=noninteractive apt-get install -y -q isc-dhcp-relay
```
`DEBIAN_FRONTEND=noninteractive` digunakan agar saat menginstall tidak meminta input dan bisa terinstall terlebih dahulu. 

Kemudian edit file `/etc/default/isc-dhcp-relay` dengan menambahkan Ip Jipangu pada input Server dan `eth1 eth2 eth3` pada input Interfaces di dalamnya. 

<image src="screenshots/2-1.PNG" width="700">

Lalu jalankan command `service isc-dhcp-relay start`

## no. 3

Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.20 - [prefix IP].1.99 dan [prefix IP].1.150 - [prefix IP].1.169

### Jawab

#### Jipangu

Mengedit file `/etc/dhcp/dhcpd.conf`:

```bash
subnet 192.195.1.0 netmask 255.255.255.0 {
    range 192.195.1.20 192.195.1.99;
    range 192.195.1.150 192.195.1.169;
    option routers 192.195.1.1;
    option broadcast-address 192.195.1.255;
    option domain-name-servers 192.195.2.2;
    default-lease-time 360;
    max-lease-time 7200;
}
```

<image src="screenshots/3-1.PNG" width="700">

Lalu jalankan command `service isc-dhcp-server restart`


#### Testing
Nyalakan `Alabasta` atau `Loguetown` dan masukkan command `ip a`

<image src="screenshots/3-2.PNG" width="700">
<image src="screenshots/3-3.PNG" width="700">


## no. 4

Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.30 - [prefix IP].3.50

### Jawab

#### Jipangu

Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan:

```bash
subnet 192.195.3.0 netmask 255.255.255.0 {
    range 192.195.3.30 192.195.3.50;
    option routers 192.195.3.1;
    option broadcast-address 192.195.3.255;
    option domain-name-servers 192.195.2.2;
    default-lease-time 720;
    max-lease-time 7200;
}
```

<image src="screenshots/4-1.PNG" width="700">

Lalu jalankan command `service isc-dhcp-server restart`

#### Testing
Nyalakan `Skypie` atau `Tottoland` dan masukkan command `ip a`

<image src="screenshots/4-2.PNG" width="700">

## no. 5

Client mendapatkan DNS dari EniesLobby dan client dapat terhubung dengan internet melalui DNS tersebut.

### Jawab

#### EniesLobby

Mengedit file `/etc/bind/named.conf.options` dengan mengubah:

```bash
forwarders {
    192.168.122.1;
};

allow-query{any;};
```

dan melakukan comment bagian

```bash
// dnssec-validation auto;
```
<image src="screenshots/5-1.PNG" width="700">


kemudian jalankan command `service bind9 restart`

#### Jipangu

Mengedit file `/etc/dhcp/dhcpd.conf` dengan menambahkan IP Ennieslobby pada `option domain-name-servers` pada `subnet 192.195.1.0` dan `subnet 192.195.3.0`

<image src="screenshots/3-1.PNG" width="700">
<image src="screenshots/4-1.PNG" width="700">

#### Testing
Mencoba melakukan `ping google.com`. Apabila bisa, artinya sudah tersambung. 
<image src="screenshots/5-2.PNG" width="700">

## no. 6

Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 6 menit sedangkan pada client yang melalui Switch3 selama 12 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 120 menit.

### Jawab

#### Jipangu

Mengedit file `/etc/dhcp/dhcpd.conf` dengan menambahkan baris ini pada `subnet 192.195.1.0`

```bash
    default-lease-time 360;
    max-lease-time 7200;
```
Dikarenakan 6 menit = 360 detik (untuk default-lease-time), dan 120 menit = 7200 detik (untuk max-lease-time)


<image src="screenshots/3-1.PNG" width="700">

dan mengedit baris ini pada `subnet 192.195.3.0`

```bash
    default-lease-time 720;
    max-lease-time 7200;
```
Dikarenakan 12 menit = 720 detik (untuk default-lease-time), dan 120 menit = 7200 detik (untuk max-lease-time)

<image src="screenshots/4-1.PNG" width="700">

#### Testing
Terlihat tulisan `lease-time` sesuai dengan konfigurasi pada client. 

<image src="screenshots/6-1.PNG" width="700">
<image src="screenshots/6-2.PNG" width="700">
<image src="screenshots/6-3.PNG" width="700">
<image src="screenshots/6-4.PNG" width="700">


## no. 7

Luffy dan Zoro berencana menjadikan Skypie sebagai server untuk jual beli kapal yang dimilikinya dengan alamat IP yang tetap dengan IP [prefix IP].3.69

### Jawab

#### Jipangu

Edit file `/etc/dhcp/dhcpd.conf` dengan menambahkan baris ini

```bash
host Skypie {
hardware ethernet 16:9d:2a:0f:ca:55;
fixed-address 192.195.3.69;
}
```

<image src="screenshots/7-1.PNG" width="700">

Lalu jalankan command `service isc-dhcp-server restart`

#### Skypie

Kemudian tambahkan `hwaddress ether hwaddress ether 16:9d:2a:0f:ca:55"` pada `/etc/network/interfaces` agar hwaddress tidak berubah/statis. 

<image src="screenshots/7-2.PNG" width="700">

#### Testing
Lakukan restart pada client, kemudian command `ip a`
<image src="screenshots/7-3.PNG" width="700">

## no. 8

Loguetown digunakan sebagai client Proxy agar transaksi jual beli dapat terjamin keamanannya, juga untuk mencegah kebocoran data transaksi.
Pada Loguetown, proxy harus bisa diakses dengan nama jualbelikapal.yyy.com dengan port yang digunakan adalah 5000

### Jawab

#### Water7

Mengedit file `/etc/squid/squid.conf` dengan menambahkan

```bash
    http_port 5000
    visible_hostname jualbelikapald07.com
    http_access allow all
```

<image src="screenshots/8-1.PNG" width="700">

kemudian jalankan command `service squid restart`

#### Ennieslobby
Membuat domain jualbelikapald07.com pada Ennieslobby seperti pada modul 2. 
Mengedit file `/etc/bind/named.conf.local`

```bash
zone "jualbelikapald07.com"{
    type master;
    file "/etc/bind/d07/jualbelikapald07.com";
};
```
<image src="screenshots/8-2.PNG" width="700">

Membuat folder baru pada bind dengan `mkdir /etc/bind/d07` dan kemudian melakukan copy `cp /etc/bind/db.local /etc/bind/d07/jualbelikapald07.com`

Mengedit file `/etc/bind/d07/jualbelikapald07.com` dengan:
```bash
$TTL    604800
@       IN      SOA     jualbelikapald07.com. root.jualbelikapald07.com. (
                                2         ; Serial
                            604800         ; Refresh
                            86400         ; Retry
                            2419200         ; Expire
                            604800 )       ; Negative Cache TTL
;
@       IN      NS      jualbelikapald07.com.
@       IN      A       192.195.2.3 ; IP Skypie
www     IN      CNAME   jualbelikapald07.com.

```

Melakukan `service bind9 restart`. 


#### Loguetown

Jalankan `apt-get update` dan `apt-get install lynx`

Aktifkan proxy dengan menjalankan command `export http_proxy="http://jualbelikapald07.com:5000"`. Untuk melihat apakah sudah tersetting, jalankan command `env | grep -i proxy`

<image src="screenshots/8-3.PNG" width="700">


#### Testing
Lakukan `lynx "website terserah` pada Loguetown. 

<image src="screenshots/8-4.PNG" width="700">

Note: Halaman terkena access denied dikarenakan saat testing belum diganti waktunya dan masih terkena konfigurasi pembatasan waktu. Terlihat bahwa proxy sudah jalan dikarenakan host_name pada halaman sama dengan konfigurasi. 


