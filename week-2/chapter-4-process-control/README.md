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

## Chapter 4: Process Control

### Daftar Isi

- [Components of a Process](komponen-proses)
- [Lifecycle of a Process](#lifecycle-of-a-process)
- [PS: Monitoring Processes](#ps-monitoring-processes)
- [Interactive monitoring with top](#interactive-monitoring-with-top)
- [Nice and renice: changing process priority](#nice-and-renice-changing-process-priority)
- [The /proc filesystem](#the-proc-filesystem)
- [Strace and truss](#strace-and-truss)
- [Runaway processes](#runaway-processes)
- [Periodic processes](#periodic-processes)

### Components of a Process

Proses terdiri dari ruang alamat (memori) dan struktur data dalam kernel. 

Struktur data dalam kernel menyimpan informasi tentang status proses, prioritas, parameter penjadwalan, dan lainnya. Proses dapat dianggap sebagai wadah yang berisi sumber daya yang dikelola kernel untuk menjalankan program, seperti halaman memori, file descriptor (mengacu pada file terbuka), serta atribut lain yang menggambarkan status proses.

Kernel mencatat berbagai informasi disetiap proses:
- Peta ruang alamat proses
- Status proses saat ini (berjalan, tidur, dll.)
- Prioritas proses
- Informasi sumber daya yang digunakan (CPU, memori, dll.)
- Informasi tentang file dan port jaringan yang dibuka
- Masker sinyal proses (sinyal yang diblokir)
- Pemilik proses (UID dari pengguna yang menjalankan proses)

Thread adalah konteks eksekusi dalam suatu proses yang memungkinkan eksekusi paralel dengan berbagi ruang alamat dan sumber daya lainnya. Dibandingkan dengan proses, thread lebih ringan dan efisien untuk dibuat serta dihancurkan. 

Contohnya, dalam server web, setiap permintaan yang masuk ditangani oleh thread baru, memungkinkan server menangani banyak permintaan secara bersamaan. Dalam kasus ini, server web berfungsi sebagai proses, sedangkan setiap thread adalah eksekusi terpisah dalam proses tersebut.

#### The PID: process ID number

Setiap proses memiliki Process ID (PID) unik yang ditetapkan oleh kernel saat dibuat dan digunakan dalam berbagai panggilan sistem, seperti mengirim sinyal ke proses tertentu. Dengan konsep namespace, beberapa proses dapat memiliki PID yang sama dalam lingkungan terisolasi, seperti container, yang memungkinkan menjalankan beberapa instance aplikasi secara independen dalam satu sistem.

#### The PPID: parent process ID number

Setiap proses memiliki Process ID (PID) unik yang ditetapkan oleh kernel saat dibuat dan digunakan dalam berbagai panggilan sistem, seperti mengirim sinyal ke proses. Dengan konsep namespace, beberapa proses dapat berbagi PID yang sama dalam lingkungan terisolasi, seperti container, sehingga memungkinkan menjalankan beberapa instance aplikasi secara independen dalam satu sistem.

#### The UID and EUID: user ID and effective user ID

- User ID (UID): ID pengguna yang menjalankan proses.
- Effective User ID (EUID): ID yang digunakan oleh proses untuk menentukan hak akses terhadap sumber daya seperti file dan port jaringan.

### Lifecycle of a Process

Proses baru dibuat dengan fork, yang menyalin proses induk dengan PID berbeda dan informasi akuntansi sendiri. Linux sebenarnya menggunakan clone, yang lebih canggih, tetapi tetap mempertahankan fork untuk kompatibilitas.

Saat sistem booting, kernel menciptakan beberapa proses, termasuk init/systemd dengan PID 1, yang bertanggung jawab menjalankan skrip startup dan menjadi induk dari semua proses lain di sistem.

#### Signals

Sinyal adalah cara untuk mengirim notifikasi ke suatu proses, memberi tahu bahwa suatu peristiwa telah terjadi. 

Terdapat sekitar 30 jenis sinyal yang digunakan dalam berbagai situasi, seperti 
- komunikasi antarproses
- interupsi dari terminal saat tombol tertentu ditekan
- perintah kill dari administrator
- peringatan dari kernel ketika terjadi pelanggaran seperti pembagian dengan nol
- pemberitahuan dari kernel tentang kondisi tertentu, misalnya matinya child process atau data yang tersedia pada saluran I/O.

![App Screenshot](img/signals.png)

Beberapa sinyal utama meliputi:

- KILL: Tidak dapat diblokir dan langsung menghentikan proses di tingkat kernel tanpa bisa ditangani oleh proses.
- INT: Dikirim oleh terminal saat pengguna menekan kombinasi tertentu (misalnya Ctrl+C) sebagai permintaan untuk menghentikan proses yang sedang berjalan.
- TERM: Permintaan agar proses mengakhiri eksekusi sepenuhnya dengan membersihkan status sebelum keluar.
- HUP: Dikirim saat terminal pengendali ditutup, awalnya digunakan untuk menunjukkan koneksi telepon yang terputus, tetapi kini sering digunakan untuk memberi tahu daemon agar memuat ulang konfigurasi dan restart.
- QUIT: Mirip dengan TERM, tetapi secara default menghasilkan core dump jika tidak ditangkap oleh proses. Beberapa program menggunakannya untuk tujuan lain.

#### Kill: send signals

Perintah kill paling sering digunakan untuk menghentikan proses. kill dapat mengirim berbagai jenis sinyal, tetapi secara default mengirimkan sinyal TERM. Pengguna biasa dapat menggunakan kill untuk menghentikan proses mereka sendiri, sementara root dapat menghentikan proses apa pun.

```bash
kill [-signal] pid
```

Di mana *sinyal* adalah nomor atau nama simbolik sinyal yang dikirim, dan pid adalah nomor identifikasi proses yang menjadi target.

Menjalankan **kill** tanpa nomor sinyal tidak menjamin proses akan mati, karena sinyal TERM dapat ditangkap, diblokir, atau diabaikan. Namun, perintah kill -9 pid dijamin akan menghentikan proses karena sinyal KILL tidak dapat ditangkap, diblokir, atau diabaikan.

Perintah **killall** digunakan untuk menghentikan proses berdasarkan nama, bukan berdasarkan PID, tetapi tidak tersedia di semua sistem.

```bash
killall firefox
```

Perintah **pkill** mirip dengan **killall** tetapi memiliki lebih banyak opsi.

```bash
pkill -u abdoufermat # kill all processes owned by user abdoufermat
```

### PS: Monitoring Processes

Perintah ps adalah alat utama administrator sistem untuk memantau proses. Meskipun versi ps berbeda dalam argumen dan tampilan, semua varian memberikan informasi yang sama secara umum.

ps dapat menampilkan PID, UID, prioritas, dan terminal kontrol dari suatu proses. Selain itu, perintah ini juga menunjukkan jumlah memori yang digunakan, waktu CPU yang dikonsumsi, serta status proses (berjalan, berhenti, tidur, dll).

Untuk melihat gambaran umum sistem, gunakan perintah:
- a: Menampilkan proses dari semua pengguna.
- u: Memberikan informasi detail tentang setiap proses.
- x: Menampilkan proses yang tidak terhubung ke terminal.

```bash
$ ps aux | head -8
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  22556  2584 ?        Ss   2019   0:02 /sbin/init
root         2  0.0  0.0      0     0 ?        S    2019   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        I<   2019   0:00 [rcu_gp]
root         4  0.0  0.0      0     0 ?        I<   2019   0:00 [rcu_par_gp]
root         6  0.0  0.0      0     0 ?        I<   2019   0:00 [kworker/0:0H-kblockd]
root         8  0.0  0.0      0     0 ?        I<   2019   0:00 [mm_percpu_wq]
root         9  0.0  0.0      0     0 ?        S    2019   0:00 [ksoftirqd/0]
```

![App Screenshot](img/process-explanation.png)

Argumen **lax** pada perintah **ps** memberikan informasi teknis lebih rinci tentang proses yang sedang berjalan. **lax** sedikit lebih cepat dibandingkan **aux** karena tidak perlu menerjemahkan nama pengguna dan grup.

```bash
$ ps lax | head -8
F   UID   PID  PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY        TIME COMMAND
4     0     1     0  20   0  22556  2584 -      Ss   ?          0:02 /sbin/init
1     0     2     0  20   0      0     0 -      S    ?          0:00 [kthreadd]
1     0     3     2  20   0      0     0 -      I<   ?          0:00 [rcu_gp]
1     0     4     2  20   0      0     0 -      I<   ?          0:00 [rcu_par_gp]
1     0     6     2  20   0      0     0 -      I<   ?          0:00 [kworker/0:0H-kblockd]
1     0     8     2  20   0      0     0 -      I<   ?          0:00 [mm_percpu_wq]
1     0     9     2  20   0      0     0 -      S    ?          0:00 [ksoftirqd/0]
```

Untuk mencari proses tertentu, gunakan **grep** untuk memfilter output dari **ps**:

```bash
$ ps aux | grep -v grep | grep firefox
```

Selain itu, juga dapat menentukan **PID** suatu proses dengan **pgrep**:

```bash
$ pgrep firefox
```

Atau dengan **pidof**, yang menunjukkan lokasi eksekusi program:

```bash
$ pidof /usr/bin/firefox
```

### Interactive monitoring with top

Perintah **top** menampilkan informasi sistem secara real-time, termasuk ringkasan sistem dan daftar proses atau thread yang dikelola kernel Linux. Pengguna dapat mengonfigurasi tampilan data sesuai kebutuhan, dan pembaruan tampilan terjadi setiap 1-2 detik secara default. 

Alternatifnya, **htop** adalah penampil proses interaktif berbasis teks yang memungkinkan pengguna menggulir secara vertikal dan horizontal untuk melihat semua proses beserta perintah lengkapnya. **htop** memiliki antarmuka yang lebih intuitif dan menyediakan lebih banyak opsi untuk pengelolaan proses dibandingkan top.

### Nice and renice: changing process priority

Niceness adalah nilai numerik yang menentukan prioritas proses dalam persaingan penggunaan CPU.Nilai niceness yang tinggi (positif) berarti prioritas rendah, sedangkan nilai yang rendah atau negatif berarti prioritas tinggi. Di Linux, rentangnya -20 hingga +19, sementara di FreeBSD -20 hingga +20.

Untuk memulai proses dengan nilai niceness tertentu, gunakan perintah nice

```bash
nice -n nice_val [command]

# Example
nice -n 10 sh infinite.sh &
```

Renice digunakan untuk mengubah prioritas proses yang sedang berjalan.

```bash
renice -n nice_val -p pid

# Example
renice -n 10 -p 1234
```

Di sistem Linux, prioritas proses berkisar dari 0 hingga 139, dengan rentang 0 hingga 99 untuk proses real-time, dan 100 hingga 139 untuk pengguna biasa.

Hubungan antara nilai nice dan prioritas proses adalah:

> priority_value = 20 + nice_value

Secara default, nilai nice adalah 0, dan semakin rendah nilainya, semakin tinggi prioritas proses.

### The /proc filesystem

Versi Linux dari **ps** dan **top** membaca informasi status proses dari direktori /proc, sebuah pseudo-filesystem di mana kernel mengekspos berbagai informasi tentang keadaan sistem. 

Meskipun namanya /proc, direktori ini tidak hanya berisi informasi tentang proses, tetapi juga statistik yang dihasilkan oleh sistem. 

Setiap proses memiliki direktori sesuai PID-nya, berisi data seperti command line, variabel lingkungan, dan file descriptor.

![App Screenshot](img/process-information.png)

### Strace and truss

Strace di Linux dan truss di FreeBSD digunakan untuk melacak panggilan sistem dan sinyal suatu proses, berguna untuk debugging atau memahami aktivitas program.

Sebagai contoh, log berikut dihasilkan oleh strace yang dijalankan pada proses top aktif dengan PID 5810:

```bash
$ strace -p 5810

gettimeofday({1197646605,  123456}, {300, 0}) = 0
open("/proc", O_RDONLY|O_NONBLOCK|O_LARGEFILE|O_DIRECTORY) = 7
fstat64(7, {st_mode=S_IFDIR|0555, st_size=0, ...}) = 0
fcntl64(7, F_SETFD, FD_CLOEXEC)          = 0
getdents64(7, /* 3 entries */, 32768)   = 72
getdents64(7, /* 0 entries */, 32768)   = 0
stat64("/proc/1", {st_mode=S_IFDIR|0555, st_size=0, ...}) = 0
open("/proc/1/stat", O_RDONLY)           = 8
read(8, "1 (init) S 0 1 1 0 -1 4202752"..., 1023) = 168
close(8)                                = 0

[...]
```

top memulai dengan memeriksa waktu saat ini, kemudian membuka dan memeriksa direktori /proc, serta membaca file /proc/1/stat untuk mendapatkan informasi tentang proses init.

### Runaway processes

Terkadang, suatu proses dapat berhenti merespons sistem dan menggunakan 100% CPU secara berlebihan. Karena proses lain hanya mendapatkan akses terbatas ke CPU, kinerja mesin menjadi sangat lambat. Hal ini disebut runaway process.

Perintah **kill** dapat digunakan untuk menghentikan proses tersebut. Jika proses tidak merespons sinyal TERM, bisa juga menggunakan sinyal KILL dengan perintah berikut:

```bash
kill -9 pid

or

kill -KILL pid
```

Penyebab runaway process dapat diselidiki menggunakan strace atau truss. Proses yang menghasilkan output tanpa henti dapat memenuhi seluruh ruang penyimpanan. Jalankan **df -h** untuk memeriksa penggunaan filesystem dan menggunakan du untuk menemukan file atau direktori terbesar.

Selain itu, perintah **lsof** dapat digunakan untuk melihat file apa saja yang sedang dibuka oleh proses tersebut:

```bash
lsof -p pid
```

### Periodic processes

#### Cron: schedule command

Cron adalah daemon yang digunakan untuk menjalankan perintah pada jadwal tertentu. Ia membaca file konfigurasi bernama crontab, yang menyimpan daftar perintah beserta waktu eksekusinya. 

#### Format of crontab

Format crontab terdiri dari lima kolom untuk menit, jam, hari, bulan, dan hari dalam seminggu. Contoh:

```bash
*     *     *     *     *  command to be executed
-     -     -     -     -
|     |     |     |     |
|     |     |     |     +----- day of week (0 - 6) (Sunday=0)
|     |     |     +------- month (1 - 12)
|     |     +--------- day of month (1 - 31)
|     +----------- hour (0 - 23)
+------------- min (0 - 59)
```

Contoh lain:

```bash
# Run a command at 2:30am every day
30 2 * * * command

# Run a command at 10:30pm on the 1st of every month
30 22 1 * * command

# Run a Python script every 1st of the month at 2:30am
30 2 1 * * /usr/bin/python3 /path/to/script.py
```

Crontab management:

- `crontab -e` → Mengedit crontab.
- `crontab -l` → Melihat daftar crontab.
- `crontab -r` → Menghapus crontab.

#### Systemd timer

Systemd timer adalah file konfigurasi unit dengan ekstensi **.timer**, yang dapat digunakan sebagai alternatif cron jobs. Systemd timer lebih fleksibel dan kuat dibandingkan cron karena dapat dipicu oleh waktu tertentu, saat sistem boot, atau berdasarkan suatu event.

Timer unit diaktifkan oleh service unit yang sesuai. Service unit akan dijalankan oleh timer unit pada waktu yang telah ditentukan dalam konfigurasi. Selain berdasarkan jadwal, timer unit juga dapat dipicu saat sistem boot atau oleh suatu event.

Untuk mengelola systemd units, gunakan perintah **systemctl**. Untuk melihat daftar timer yang sedang aktif, gunakan:

```bash
$ systemctl list-timers

NEXT                         LEFT          LAST                         PASSED       UNIT                         ACTIVATES
Fri 2021-10-15 00:00:00 UTC  1h 1min left Thu 2021-10-14 00:00:00 UTC  22h ago      logrotate.timer              logrotate.service

1 timers listed.
```

Pada contoh di atas, **logrotate.timer** dijadwalkan untuk mengaktifkan **logrotate.service** setiap tengah malam.

Contoh konfigurasi unit timer untuk rotasi log harian:

```bash
$ cat /usr/lib/systemd/system/logrotate.timer

[Unit]
Description=Daily rotation of log files
Documentation=man:logrotate(8) man:logrotate.conf(5)

[Timer]
OnCalendar=daily
AccuracySec=1h
Persistent=true

[Install]
WantedBy=timers.target

```

- OnCalendar = daily → Menjadwalkan eksekusi setiap hari.
- AccuracySec = 1h → Menentukan akurasi waktu eksekusi dengan toleransi 1 jam.
- Persistent = true → Memastikan timer tetap berjalan meskipun sistem reboot.

#### Common use for scheduled tasks

Sending mail

Mengotomatiskan pengiriman laporan harian atau hasil eksekusi perintah menggunakan cron atau systemd timers.

Contoh:

```bash
30 4 25 * * /usr/bin/mail -s "Monthly report"
    abdou@admin.com%Receive the monthly report for the month of July!%%Sincerely,%cron%
```

Cleaning up a filesystem

Menggunakan cron atau systemd timers untuk menjalankan skrip yang membersihkan filesystem, seperti menghapus isi direktori sampah setiap tengah malam.

```bash
0 0 * * * /usr/bin/find /home/abdou/.local/share/Trash/files -mtime +30 -exec /bin/rm -f {} \;
```

Rotating a log file

Rotasi log membagi file log berdasarkan ukuran atau tanggal, menjaga beberapa versi lama tetap tersedia. Karena dilakukan secara berkala, rotasi log ideal untuk dijadwalkan secara otomatis.

Running batch jobs

Perhitungan jangka panjang seperti pemrosesan pesan dalam antrian atau database dapat dijalankan sebagai batch jobs menggunakan cron, misalnya untuk ETL ke data warehouse.

Backing up and mirroring

Tugas terjadwal dapat digunakan untuk mencadangkan direktori ke sistem jarak jauh. Mirroring membuat salinan byte-per-byte dari sistem atau direktori lain, berguna untuk backup atau distribusi file. rsync dapat digunakan untuk memperbarui mirror secara berkala.

### Kesimpulan

Mengelola proses dan tugas terjadwal dalam sistem Linux sangat penting untuk menjaga kinerja dan stabilitas sistem. Perintah seperti top dan htop memungkinkan pemantauan proses secara real-time, sementara ps mengambil informasi proses dari /proc. Prioritas proses dapat diatur menggunakan nice dan renice, serta proses yang tidak merespons dapat dihentikan dengan kill.

Untuk menganalisis aktivitas proses, alat seperti strace dan truss dapat digunakan untuk menelusuri panggilan sistem. Jika terjadi proses runaway yang menghabiskan CPU, investigasi dapat dilakukan dengan lsof, df, atau du.

Tugas berkala dapat dikelola dengan cron atau systemd timers, di mana cron lebih klasik dan berbasis crontab, sedangkan systemd timers lebih fleksibel dengan kontrol berbasis unit service. Beberapa contoh penggunaan tugas terjadwal meliputi pengiriman email otomatis, pembersihan filesystem, rotasi log, eksekusi batch jobs, dan backup serta mirroring data.