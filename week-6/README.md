<div align="center">
  <h1 style="text-align: center;font-weight: bold">Laporan<br>Workshop Administrasi Jaringan</h1>
  <h4 style="text-align: center;">Dosen Pengampu : Dr. Ferry Astika Saputra, S.T., M.Sc.</h4>
</div>
<br />
<div align="center">
  <img src="https://upload.wikimedia.org/wikipedia/id/4/44/Logo_PENS.png" alt="Logo PENS">
  <h3 style="text-align: center;">Disusun Oleh :</h3>
  <p style="text-align: center;">
    <strong>Maula Shahihah Nur Sa'adah</strong><br>
    <strong>3123500008</strong>
  </p>

<h3 style="text-align: center;line-height: 1.5">Politeknik Elektronika Negeri Surabaya<br>Departemen Teknik Informatika Dan Komputer<br>Program Studi Teknik Informatika<br>2024/2025</h3>
  <hr><hr>
</div>

## Daftar Isi

1. [Konfigurasi pada VM 1 (Server)](#konfigurasi-vm-1)
2. [Konfigurasi NTPSec](#konfigurasi-ntpsec-vm-1)
3. [Konfigurasi File Samba](#konfigurasi-samba-vm-1) 
4. [Konfigurasi DNS Server](#konfigurasi-dns-server-vm-1)
5. [Konfigurasi pada VM 2 (Client)](#konfigurasi-vm-2)
6. [Kesimpulan](#kesimpulan)

Dalam praktikum ini, diperlukan 2 Virtual Machine (VM) yang akan berperan sebagai **Client** dan **Server**.

### Konfigurasi VM 1 (Server)

VM 1 akan berperan sebagai server. Konfigurasi jaringan nya menggunakan dua network adapter, yaitu:

- Bridge Adapter yang digunakan untuk koneksi internet.

- Internal Network yang digunakan untuk komunikasi langsung dengan VM Client.

VM 1 difungsikan sebagai gateway untuk menghubungkan client ke jaringan internet. Selain itu, server ini juga disiapkan dengan beberapa layanan seperti **Samba** untuk berbagi file, **NTPSec** untuk sinkronisasi waktu, dan **Bind9** untuk pengelolaan domain lokal.

#### Konfigurasi Network Adapter

![App Screenshot](img/vm1-bridged-adapter.png)

![App Screenshot](img/vm1-internal-network.png)

Server dikonfigurasi dengan dua network adapter, yaitu **Bridged Adapter** dan **Internal Network**. **Bridged Adapter** digunakan untuk mengakses jaringan internet melalui koneksi fisik host, sedangkan **Internal Network** digunakan untuk membangun koneksi lokal langsung dengan VM 2 yang berperan sebagai client.

#### Konfigurasi IP Address untuk Koneksi Internal antar VM

1. Cek IP Address

   ![App Screenshot](img/cek-ip.png)

2. Akses terminal VM 1 menggunakan SSH

   ![App Screenshot](img/ssh-student.png)

   Untuk melakukan konfigurasi pada VM 1 yang berperan sebagai server, koneksi dapat dilakukan secara remote dari host menggunakan protokol SSH (Secure Shell).
   
3. Tambahkan file `/etc/network/interfaces`

   ![App Screenshot](img/etc-network-interfaces.png)

   Setiap network adapter yang terpasang pada sistem akan secara otomatis diberi nama unik. Pada VM yang digunakan, adapter teridentifikasi sebagai enp0s3 untuk Bridged Adapter dan enp0s8 untuk Internal Network. Adapter enp0s3 dikonfigurasi menggunakan metode DHCP agar dapat memperoleh alamat IP secara otomatis dari jaringan internet. Sementara itu, adapter enp0s8 dikonfigurasi secara manual dan akan berperan sebagai gateway untuk VM client.

#### Aktifkan forwarding di Server

1. `/etc/sysctl.conf`

   ![App Screenshot](img/etc-sysctl-conf.png)

   Untuk membuat sistem berfungsi sebagai router atau gateway, konfigurasi IP forwarding harus diaktifkan. Langkah ini dilakukan dengan mengedit file konfigurasi dan mengaktifkan opsi `net.ipv4.ip_forward`.

2. Lakukan Validasi

   ![App Screenshot](img/sysctl-p.png)

#### Konfigurasi Iptables

1. Install Paket iptables dan iptables-persistant

   ![App Screenshot](img/install-iptables-iptables-persistent.png)

2. Konfigurasi iptables pada file `/etc/iptables/rules.v4`

   ![App Screenshot](img/etc-iptables-rules-v4.png)

3. Jalankan perintah `iptables-restore < /etc/iptables/rules.v4` untuk menyimpan konfigurasi iptables

    ![App Screenshot](img/iptables-restore.png)

#### Reboot

Setelah menyelesaikan seluruh proses konfigurasi, jalankan perintah `reboot` untuk me-restart VM.

### Konfigurasi NTP (NTPSec) pada VM 1

1. Install paket ntpsec

   ![App Screenshot](img/install-ntpsec.png)

2. Konfigurasi server NTP

   ![App Screenshot](img/etc-ntpsec-ntp-conf.png)

3. Restart service ntpsec, jalankan perintah `systemctl restart ntpsec`

   ![App Screenshot](img/systemctl-restart-ntpsec.png)

4. Kemudian cek menggunakan perintah `systemctl status ntpsec`

   ![App Screenshot](img/systemctl-status-ntpsec.png)

5. Validasi NTP Server
   
   ![App Screenshot](img/ntpq-p.png)

### Konfigurasi File Samba pada VM 1

1. Install paket samba

   ![App Screenshot](img/install-samba.png)

2. Buat direktori `mkdir /home/share` dan ubah permission agar bisa diakses `chmod 777 /home/share`

   ![App Screenshot](img/mkdir-home-share.png)

3. Konfigurasi samba pada file `/etc/samba/smb.conf`

   ![App Screenshot](img/etc-samba-smb-conf-global.png)

   ![App Screenshot](img/etc-samba-smb-conf-share.png)

4. Restart service dengan perintah `systemctl restart smbd`

   ![App Screenshot](img/systemctl-restart-smbd.png)

### Konfigurasi DNS Server pada VM 1 

1. Instalasi paket dengan menjalankan perintah `apt -y install bind9 bind9utils`

   ![App Screenshot](img/install-bind9-bind9utils.png)

2. Lakukan modifikasi pada file `/etc/bind/named.conf` dengan menambahkan `include "/etc/bind/named.conf.internal-zones";`

   ![App Screenshot](img/etc-bind-named-conf.png)

3. Modifikasi file `/etc/bind/named.conf.options`

   ![App Screenshot](img/etc-bind-named-conf-options.png)

4. Konfigurasi internal zone pada file `/etc/bind/named.conf.internal-zones`

   ![App Screenshot](img/etc-bind-named-conf-internal-zones.png)

5. Modifikasi file `/etc/default/named` dengan menambahkan `-4`

   ![App Screenshot](img/etc-default-named.png)

6. Membuat file sesuai dengan domain lokal

   ![App Screenshot](img/etc-bind-kelompok3-home-lan.png)

7. Membuat file sesuai dengan IP Address

   ![App Screenshot](img/etc-bind-3-168-192-db.png)

### Konfigurasi pada VM 2

VM 2 dikonfigurasi dengan alamat IP yang berada dalam satu jaringan dengan interface internal VM 1 (enp0s3), sehingga keduanya dapat saling terhubung dan berkomunikasi.

![App Screenshot](img/etc-network-interfaces-vm2.png)

#### Cek koneksi dengan melakukan ping ke gateway

![App Screenshot](img/ping-gateway.png)

#### Cek koneksi dengan melakukan ping ke DNS server

![App Screenshot](img/ping-dns-server.png)

#### Cek koneksi dengan melakukan ping ke domain lokal

![App Screenshot](img/ping-kelompok3-home.png)

#### Tes akses folder dari Samba (Client)

![App Screenshot](img/tes-akses-file-dari-samba.png)

Jika folder **Share** sudah muncul di tampilan file manager client seperti yang terlihat pada gambar, itu berarti konfigurasi Samba di server sudah berjalan dengan benar dan client berhasil mengakses resource sharing yang disediakan oleh server.

#### Cek DNS Server (Client) menggunakan nama domain

![App Screenshot](img/cek-menggunakan-nama-domain.png)

Hasil pengecekan menggunakan perintah `dig` menunjukkan bahwa domain `ns.kelompok3.home` berhasil ter-resolve ke alamat IP `192.168.3.1` tanpa adanya error. Hal ini menandakan bahwa konfigurasi DNS pada server sudah berjalan dengan benar dan client dapat mengenali nama domain lokal yang telah ditentukan.

#### Cek DNS Server (Client) menggunakan IP address

![App Screenshot](img/cek-menggunakan-ip-addr.png)

Hasil perintah `dig -x 192.168.3.1` menunjukkan bahwa IP tersebut berhasil di-resolve menjadi nama domain `ns.kelompok3.home`. Ini menandakan bahwa konfigurasi reverse DNS (PTR record) pada server telah berhasil dilakukan dengan benar dan sistem DNS sudah dapat menerjemahkan alamat IP menjadi hostname sesuai konfigurasi.

## Kesimpulan

Dalam praktikum ini, saya berhasil mengkonfigurasi dua Virtual Machine, yaitu VM1 sebagai server dan VM2 sebagai client. Saya mengatur server dengan dua adapter jaringan (Bridged Adapter dan Internal Network), kemudian mengaktifkan fitur IP forwarding agar server dapat berfungsi sebagai gateway. Saya juga memberikan IP statis pada masing-masing VM dalam jaringan yang sama agar keduanya dapat saling terhubung. Selain itu, saya menggunakan SSH untuk mengakses server dari client secara remote. Praktikum ini membantu saya memahami cara membangun koneksi jaringan lokal serta mengelola komunikasi antar perangkat virtual secara langsung.