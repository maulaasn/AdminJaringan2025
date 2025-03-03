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

1. [Instalasi Debian Linux Pada VirtualBox](#instalasi-debian)
2. [Kesimpulan](#kesimpulan)

## Instalasi Debian Linux di VirtualBox melalui terminal root pada OS Linux

#### Solve LAN Wired Not Connected

![App Screenshot](img/solve-lan-wired.jpeg)

Perintah `sudo dhclient -v` digunakan untuk memperbarui konfigurasi jaringan secara dinamis melalui DHCP dan menampilkan output secara detail.

#### Install Git dan Clone Github

![App Screenshot](img/install-git.png)

Command `sudo apt install git` digunakan untuk menginstall git pada komputer lab.

![App Screenshot](img/git-clone.jpeg)

Kemudian clone menggunakan git clone dengan cara copy paste link github dibawah ini:

<pre><code>https://github.com/ferryastika/unix-and-linux-sysadmin-notes.git</code></pre>

#### Download VirtualBox Linux

![App Screenshot](img/buka-virtualbox-linux.png)

Klik pada bagian Linux distributions.

![App Screenshot](img/download-debian-12.png)

Kemudian pilih Debian 12, lalu klik untuk mendownload.

Setelah proses unduhan selesai, file VirtualBox akan tersimpan di folder Downloads.

![App Screenshot](img/folder-download.jpeg)

#### Download ISO Debian 12

![App Screenshot](img/download-iso-debian.png)

ISO yang akan digunakan dalam VirtualBox.

#### Proses Instalasi VirtualBox

Di folder Downloads, klik kanan pada file VirtualBox, lalu pilih Open Terminal. Kemudian, jalankan perintah berikut untuk memulai instalasi VirtualBox.

![App Screenshot](img/install-virtualbox.png)

Dari capture diatas terdapat error yang terjadi karena dependensi yang dibutuhkan oleh VirtualBox belum terinstal, khususnya `libxcb-cursor0`. Saat menggunakan `dpkg -i`, hanya paket yang dipilih yang diinstal tanpa secara otomatis menyelesaikan dependensi yang hilang. Karena `libxcb-cursor0` tidak ditemukan di sistem, proses instalasi VirtualBox gagal dengan status "dependency problems".

Untuk mengatasi permasalahan diatas, dapat dijalankan dengan perintah berikut:

![App Screenshot](img/fixing-bug.png)

Perintah `sudo apt install -f` akan menginstal semua dependensi yang hilang dan memperbaiki konfigurasi yang belum selesai, sehingga VirtualBox dapat terinstal dengan benar.

#### Menjalankan Virtual Machine

![App Screenshot](img/cant-enumerate-usb.png)

Error `Can't enumerate USB devices` di VirtualBox terjadi karena sistem gagal mengenali USB, biasanya akibat dukungan USB belum aktif, kurangnya izin user, atau Extension Pack belum terinstal. Masalah ini sering terjadi di Linux jika user belum masuk grup `vboxusers`. 

![App Screenshot](img/sudo-usermod.jpeg)

Jika menggunakan Linux, user perlu memiliki izin untuk mengakses perangkat USB di VirtualBox. Maka jalankan perintah `sudo usermod -aG vboxusers $USER` di terminal, lalu restart sistem agar perubahan diterapkan.

#### Menjalankan New Virtual Machine

Jika tampilan sudah sesuai seperti dibawah ini, maka proses akan dilanjutkan ke langkah berikutnya.

![App Screenshot](img/tampilan-awal.jpeg)

#### Langkah - Langkah

![App Screenshot](img/langkah-1.png)

Sesuaikan nama dan OS seperti pada gambar di atas. Jika file ISO Debian 12 sudah diunduh, masukkan lokasinya pada kolom ISO Image.

![App Screenshot](img/langkah-2.png)

Melakukan setting username dan password dengan menggunakan nama `student`.

![App Screenshot](img/langkah-3.png)

Setup RAM menjadi 2 GB dan menggunakan 2 CPU untuk prosesor.

![App Screenshot](img/langkah-4.png)

Mengalokasikan 10 GB ruang penyimpanan pada HDD (Hard Disk Drive).

![App Screenshot](img/langkah-5.png)

Jika spesifikasi sudah sesuai, klik Finish untuk melanjutkan proses instalasi.

#### Error Pada Kernel

Setelah mengklik Finish, muncul pop-up error seperti yang ditampilkan di bawah ini.

![App Screenshot](img/error-kernel.png)

Error `Kernel driver not installed (rc=-1908)` terjadi karena VirtualBox tidak dapat menemukan atau memuat driver kernel (vboxdrv) yang diperlukan. Penyebabnya bisa karena Secure Boot masih aktif, modul kernel belum terinstal, atau VirtualBox perlu dikonfigurasi ulang setelah pembaruan sistem.

Untuk mengatasi masalah diatas, jalankan perintah berikut guna memuat ulang driver kernel.

![App Screenshot](img/solving-error-1.png)

![App Screenshot](img/solving-error-2.png)

![App Screenshot](img/solving-error-3.png)

#### Proses Instalasi Debian Berjalan

Jalankan dan tunggu hingga Debian pada VM terinstal dan terinisialisasi dengan benar.

![App Screenshot](img/proses-install-software.jpeg)

![App Screenshot](img/proses-install-base-system.jpeg)

Jika tampilan sudah seperti dibawah ini, maka instalasi Debian 12 di VirtualBox telah berhasil.

![App Screenshot](img/finish.jpeg)

## Kesimpulan

Kesimpulannya, error pada VirtualBox, seperti `Kernel driver not installed (rc=-1908)` dan `Can't enumerate USB devices`, umumnya disebabkan oleh konfigurasi yang belum lengkap, izin yang kurang, atau modul yang belum terinstal. Untuk mengatasinya yaitu meliputi menambahkan user ke grup `vboxusers`, serta menginstal dan mengonfigurasi ulang driver kernel. Jika menjalankan Debian 12 di VirtualBox, user harus mendownload ISO dan sistem dikonfigurasi dengan benar agar dapat berjalan tanpa ada problem.