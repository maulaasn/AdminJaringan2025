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

1. [Tugas](#tugas)
2. [Hasil Percobaan](#hasil-percobaan)

## Tugas 

![App Screenshot](img/tugas.png)

1. Melakukan Instalisasi Winbox 
2. Konfigurasi Layer Network
   Percobaan awal yang dilakukan berdasarkan skema di atas adalah menghubungkan perangkat MikroTik antar kelompok. Tujuan dari langkah ini adalah agar setiap laptop yang tergabung dalam jaringan LAN dapat saling berkomunikasi, khususnya melakukan uji koneksi (ping) ke IP kelompok lainnya. 
   
   Untuk melakukan hal tersebut, salah satu laptop dari setiap kelompok perlu dihubungkan ke jaringan LAN dan menjalankan perintah ping guna menguji konektivitas. Pada percobaan ini, digunakan rentang alamat IP 10.252.108.5x, di mana x mewakili nomor kelompok masing-masing. Karena saya kelompok 3, maka IP MikroTik yang digunakan adalah 10.252.108.53. 
   
   Setelah perangkat berhasil terhubung ke jaringan melalui kabel LAN, pengujian koneksi dapat dilakukan melalui Command Prompt di sistem operasi Windows dengan menggunakan perintah ping terhadap IP tujuan.
3. Ping device kelompok lain sesuai networknya

## Hasil Percobaan

1. Install Wine
   
   ![App Screenshot](img/install_wine.png)

   Installasi ini diperlukan agar bisa melakukan install aplikasi Winbox di Linux. Kemudian, instalasi ini akan menginstal Wine, yang merupakan sebuah emulator untuk Windows. Wine ini digunakan untuk membuat aplikasi yang dibuat untuk Windows, dapat dijalankan di Linux. 

2. Install Winbox dengan sftp kelompok 5 

   ![App Screenshot](img/sftp_10_252_108_110.png)

   ![App Screenshot](img/get_winbox.png)

3. Mencoba ping network 10.252.108.53

   ![App Screenshot](img/ping.png)

   Berdasarkan hasil di atas, network milik kelompok 3 terdeteksi saat terhubung ke LAN dengan alamat IP 10.252.108.53

4. Connect Winbox ke Mikrotik dengan IP address 10.252.108.53
    
   ![App Screenshot](img/connect_network_winbox_kel3.png)

   ![App Screenshot](img/berhasil_connect.png)

5. Mencoba ping network 192.168.1.0/24 di Mikrotik (Kelompok 1)

   ![App Screenshot](img/ping_kel_7.png)

   Pada bagian IP, pilih menu Routes, kemudian tambahkan network 192.168.1.0/24 dan isi kolom gateway dengan 10.252.108.51 (pada gambar di atas merupakan konfigurasi untuk kelompok 7). Setelah konfigurasi selesai, klik button OK.

   Sembunyikan (hidden) IP yang tidak terhubung ke perangkat MikroTik untuk menghindari kebingungan. Selanjutnya, lakukan pengujian koneksi dengan menjalankan perintah ping ke IP address 10.252.108.51 melalui Command Prompt. 
   
   Sebagai contoh, gambar di bawah ini menunjukkan hasil ping ke IP 192.168.1.0
  
   ![App Screenshot](img/hasil_ping_kel_1.png)

   Jika test ping seperti pada langkah sebelumnya berhasil dilakukan, dan setiap uji koneksi tidak menunjukkan status request timed out atau pesan kesalahan lainnya, maka dapat disimpulkan bahwa koneksi antar device dari kelompok yang berbeda telah berhasil dilakukan.
   