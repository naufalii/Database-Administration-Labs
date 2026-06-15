# Journey: System Administrator

---

## TAHAP 1 : Manajemen User dan Hak Akses

### 1. Mengecek Identitas Saat Ini

```bash
whoami
```

(Biasanya akan muncul nama user yang sedang kamu pakai sekarang).

---

### 2. Membuat User Baru

```bash
sudo adduser volunteer
```

(Sistem akan memintamu memasukkan password untuk user baru tersebut, dan mengisi beberapa detail seperti nama lengkap. Tekan **Enter** saja untuk mengosongkan detail yang tidak penting).

---

### 3. Berpindah ke User Baru

```bash
su - volunteer
```

![Tampilan terminal setelah berpindah user](imager/imager1.png)

Coba perhatikan perubahan kecil tapi sangat penting di terminalmu:

- Saat kamu menjadi **Root**, simbol di akhir baris adalah `#` (`[root@exaprimary oracle]#`).
  Ini adalah tanda bahwa kamu memiliki kuasa penuh atas sistem.
- Saat kamu menjadi **volunteer**, simbolnya berubah menjadi `$` (`[volunteer@exaprimary ~]$`).
  Ini menandakan kamu adalah user biasa dengan hak akses terbatas.

---

### Menguji Hak Akses (Permissions)

**1. Mencoba masuk ke direktori milik Root:**

```bash
ls /root
```

(Seharusnya sistem akan menolak dengan pesan `"Permission denied"` karena direktori ini sangat rahasia).

**2. Mencoba "meminjam" kekuatan Root dengan sudo:**

```bash
sudo ls /root
```

(Sistem akan meminta password volunteer. Setelah dimasukkan, kemungkinan besar sistem akan marah dan memunculkan pesan bahwa volunteer tidak ada di dalam kelompok `"sudoers"`).

![Hasil error sudo belum terdaftar di sudoers](imager/imager2.png)

**sudo gagal:** Ada dua alasan kenapa ini terjadi.

Pertama, karena kamu menggunakan Oracle Linux (yang berbasis keluarga RedHat), perintah `adduser` di awal tadi ternyata belum menyetel password sama sekali untuk volunteer.

Kedua, user volunteer belum terdaftar di dalam grup khusus yang diizinkan memakai sudo.

---

### Memberikan "Izin Resmi"

**1. Kembali Menjadi Root:**

```bash
exit
```

**2. Atur Password untuk Volunteer:**

```bash
passwd volunteer
```

**3. Masukkan ke Grup Admin (Sudoers)** — grup admin ini bernama `wheel`:

```bash
usermod -aG wheel volunteer
```

**4. Uji Coba Ulang:**

```bash
su - volunteer
```

**5. Lalu jalankan ulang perintah ini dan masukkan password yang baru saja kamu buat:**

```bash
sudo ls /root
```

![Hasil sudo berhasil setelah masuk grup wheel](imager/imager3.png)

---

### Memahami Hak Akses File (File Permissions)

Sebagai SysAdmin, kamu harus memastikan file konfigurasi sensitif tidak bisa dibaca atau diubah oleh sembarang orang.

Di Linux, hak akses dibagi menjadi tiga kelompok:

1. **User (Owner):** Pemilik file.
2. **Group:** Kelompok user tertentu.
3. **Others:** Semua orang selain pemilik dan grup.

Setiap kelompok memiliki tiga jenis izin (permissions):

| Simbol | Nama    | Nilai | Keterangan                                      |
|--------|---------|-------|-------------------------------------------------|
| `r`    | Read    | 4     | Bisa melihat isi file                           |
| `w`    | Write   | 2     | Bisa mengubah isi file                          |
| `x`    | Execute | 1     | Bisa menjalankan file (jika berupa program/script) |

Angka-angka di atas dijumlahkan untuk memberikan hak akses.
Misalnya, akses `6` berarti Read (4) + Write (2). Akses `7` berarti Read (4) + Write (2) + Execute (1).

---

### Simulasi Pembuatan File Rahasia

**1. Membuat file teks baru bernama `rahasia.txt`:**

```bash
> rahasia.txt
```

**2. Menginput isi file nya:**

```bash
vi rahasia.txt
```

**3. Mengecek hak akses bawaan file tersebut:**

```bash
ls -l rahasia.txt
```

(Awalnya `-rw-rw-r--`: Pemilik (volunteer) dan kelompoknya bisa membaca/mengubah, dan orang lain di luar kelompok masih bisa membaca file tersebut).

**4. Mengubah hak akses agar HANYA pemilik yang bisa membaca dan menulis (Akses 600):**

```bash
chmod 600 rahasia.txt
```

**5. Cek lagi perubahannya:**

```bash
ls -l rahasia.txt
```

(Perhatikan bahwa hurufnya sekarang berubah menjadi `rw-------`, artinya semua akses untuk Group dan Others sudah ditutup rapat).

**6. Memindahkan kepemilikan file tersebut ke tangan Root (chown):**

```bash
sudo chown root:root rahasia.txt
```

**7. Menguji keamanan: Coba baca isi file tersebut sebagai user biasa:**

```bash
cat rahasia.txt
```

> **Hasil Akhir** (`cat: rahasia.txt: Permission denied`): Ketika kamu (volunteer) mencoba membaca file tersebut, sistem langsung memblokirnya.
> Mengapa? Karena file itu sekarang milik root, dan hak aksesnya terkunci rapat (600). Kamu tidak punya hak lagi atas file yang kamu buat sendiri.

**8. Mengecek kembali hak akses file tersebut:**

```bash
ls -l rahasia.txt
```

![Hasil akhir ls -l setelah chown ke root](imager/imager4.png)

---

## TAHAP 2 : Setup Web Server

### 1. Membuat Folder untuk "Hosting" (user volunteer)

```bash
mkdir ~/web_kalian
cd ~/web_kalian
```

### 2. Membuat File Web Statis (HTML)

```bash
vi index.html
```

```html
<!DOCTYPE html>
<html>
<head>
    <title>Server Volunteer SysAdmin</title>
</head>
<body>
    <h1>Sistem Berhasil Dihosting!</h1>
    <p>Ini adalah halaman web internal yang dikelola oleh SysAdmin.</p>
</body>
</html>
```

```
esc, wq!, enter
```

### 3. Masuk ke User Root, Lalu Nyalakan Firewall

```bash
systemctl start firewalld
```

### 4. Beri Izin Port yang Ingin Kita Gunakan (misal: 8080) di Firewall

```bash
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload
```

![Konfigurasi firewall-cmd add-port](imager/imager5.png)

### 5. Cek Apakah Port Sudah Terdaftar atau Belum

```bash
firewall-cmd --list-all
```

![Hasil firewall-cmd list-all](imager/imager6.png)

### 5. Masuk ke User Volunteer Lagi, Lalu Nyalakan Web Server Bawaan

```bash
python -m SimpleHTTPServer 8080
```

Jika berhasil, terminalmu akan "terkunci" dan memunculkan pesan seperti:

```
Serving HTTP on 0.0.0.0 port 8080 ...
```

![Terminal terkunci menampilkan Serving HTTP](imager/imager7.png)

### 6. Menguji Server dari Dalam (Local Request)

```bash
curl http://localhost:8080
```

Jika berhasil, terminal akan memuntahkan kode HTML (`<!DOCTYPE html>...`) yang kamu ketik di vi tadi.

![Hasil curl localhost:8080](imager/imager8.png)

### 7. Mencari Tahu IP Address Server

```bash
ip addr show
```

![Hasil ip addr show](imager/imager9.png)

### 8. Buka Alamat Server yang Kamu Hosting di Browser

```
http://192.168.56.10:8080
```

> Sesuaikan dengan alamat kalian sendiri.

---

## TAHAP 3 : Manajemen Servis & Otomatisasi

Tugas kita di Tahap 3 ini adalah mengubah server Python tadi menjadi sebuah **Background Service (Daemon)** menggunakan sistem pengontrol Linux yang bernama `systemd`. Dengan cara ini:

1. Aplikasi akan berjalan diam-diam di latar belakang (background).
2. Terminal kamu akan terbebas kembali dan bisa dipakai kerja.
3. Aplikasi akan otomatis menyala sendiri meskipun server habis mati lampu atau di-restart.

---

### Langkah-langkah:

**1. Matikan Server Python yang Sekarang (yang dilakukan cara manual):**

```
Ctrl + C
```

**2. Pindah ke Akun Root, lalu buat file service:**

```bash
vi /etc/systemd/system/web-volunteer.service
```

```ini
[Unit]
Description=Web Server Python Volunteer SysAdmin
After=network.target

[Service]
Type=simple
User=volunteer
WorkingDirectory=/home/volunteer/web_kalian
ExecStart=/usr/bin/python -m SimpleHTTPServer 8080
Restart=always

[Install]
WantedBy=multi-user.target
```

> **Catatan:** `WorkingDirectory` sesuaikan dengan nama folder web yang kalian buat tadi untuk menyimpan HTML. Jika kalian tidak di akun root (misal di user volunteer), tambahkan `sudo` di depannya.

**3. Jalankan tiga perintah ini:**

```bash
systemctl daemon-reload
systemctl start web-volunteer
systemctl status web-volunteer
```

Lihat baris ini di log terminalmu:

```
Active: active (running) ...
```

**Sekarang kelebihan servermu adalah:**

- **Terminal Bebas:** Kita bisa mengetikkan perintah lain di terminal root atau volunteer tanpa takut mematikan websitenya.
- **Aman dari Putus Koneksi:** Kita bisa menutup aplikasi VirtualBox atau software SSH kita sekarang, dan website tersebut akan tetap aktif bisa diakses dari browser laptop luar lewat `http://192.168.56.10:8080`.

Ada satu detail kecil lagi di log statusmu: `; disabled; vendor preset: disabled`.

Artinya, kalau server `exaprimary` ini kita shutdown atau restart, servis web ini tidak akan menyala otomatis secara bawaan. Kita harus masuk ke server lagi dan mengetik `systemctl start` secara manual.

**4. Jalankan perintah ini untuk membuatnya otomatis menyala sejak server pertama kali melakukan booting:**

```bash
systemctl enable web-volunteer
```

![Hasil systemctl enable web-volunteer](imager/imager10.png)

---

> **Catatan Penting — Perbedaan `web_kalian` vs `web-volunteer`:**
>
> Alasan kenapa di `systemctl` kita memanggil `web-volunteer` dan bukan `web_kalian` adalah karena `systemctl` hanya membaca nama file konfigurasi `.service` yang kita buat di dalam folder `/etc/systemd/system/`.
>
> **1. `web_kalian` (Nama Folder Tempat File)**
> Ini hanyalah sebuah folder biasa di dalam direktori home kamu (`/home/volunteer/web_kalian`). Fungsinya cuma sebagai kontainer/wadah untuk menyimpan file `index.html` milikmu. Linux atau `systemd` tidak akan pernah tahu kalau folder ini berisi sebuah web server sampai kita memberi tahu mereka lewat file konfigurasi.
>
> **2. `web-volunteer.service` (Nama Servis/KTP Aplikasi)**
> Ini adalah file konfigurasi (bisa dibilang seperti "KTP" atau identitas) yang kita buat di jalur sistem: `/etc/systemd/system/web-volunteer.service`. Di dalam file inilah kita mengetik perintah `WorkingDirectory=/home/volunteer/web_kalian`. Di baris itulah kita menjembatani atau memberi tahu Linux: *"Eh Linux, kalau servis ini dinyalakan, tolong buka folder `web_kalian` ya!"*

---

## TAHAP 4 : Jaringan Dasar & Troubleshooting Log

Sekarang, untuk menyelesaikan Tahap 4, mari kita lakukan simulasi tantangan troubleshooting yang sesungguhnya.

Jangan matikan terminal `journalctl` tersebut, biarkan dia tetap memantau secara real-time.

Buka kembali browser di laptop utamamu, lalu sengaja ketik URL asal-asalan yang filenya tidak pernah kita buat di server. Contohnya:

```
http://192.168.56.10:8080/rahasia-negara.html
```

![Log terminal menampilkan error 404](imager/imager11.png)

Log di terminalmu pasti langsung memuntahkan status yang berubah dari `200` menjadi `"GET /rahasia-negara.html HTTP/1.1" 404 -`.

---

### === SIMULASI === Memblokir IP Laptop Utamamu Sendiri

**1. Masuk ke user root — Pantau log secara real-time:**

```bash
journalctl -u web-volunteer -f
```

**2. Jika sudah tahu IP Laptopmu yang berjalan, lakukan pemblokiran** (sesuaikan IP-nya dengan IP kalian):

```bash
firewall-cmd --permanent --add-rich-rule="rule family='ipv4' source address='192.168.56.1' reject"
```

**3. Setelah muncul tulisan success, terapkan aturannya dengan melakukan reload:**

```bash
firewall-cmd --reload
```

![Hasil pemblokiran IP dengan firewall-cmd](imager/imager12.png)

**4. Uji Coba Efeknya di Browser:**

```
http://192.168.56.10:8080
```

(sesuaikan alamat IP kalian)

![Browser menampilkan error This site can't be reached](imager/imager13.png)

(Browser kamu sekarang pasti akan langsung mogok dan memuntahkan eror `"This site can't be reached"` atau `Connection Refused` — kalian sudah berhasil melakukan pemblokiran pada laptop kalian sendiri).

**5. Lalu kita kembalikan aksesnya:**

```bash
firewall-cmd --permanent --remove-rich-rule="rule family='ipv4' source address='192.168.56.1' reject"
```

**6. Jangan lupa untuk melakukan reload kembali agar Satpam memperbarui catatannya:**

```bash
firewall-cmd --reload
```

![Hasil remove-rich-rule dan reload firewall](imager/imager14.png)

**7. Verifikasi Akhir:**

Refresh browser kalian sekarang maka sudah seperti semula.

---

*Dibuat oleh **Naufal Ali Hilmi** | Information Systems Student, Gunadarma University*