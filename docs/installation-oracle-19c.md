# INSTALLATION ORACLE 19C (LINUX)

---

## Oracle VM Virtual Box

### 1. Klik New, untuk membuat Servernya

![VirtualBox Manager - Welcome Screen](images/image1.png)

---

### 2. Berikan nama Server, Type = Linux, Version = Oracle Linux (64-bit)

![Create Virtual Machine - Name and Operating System](images/image2.png)

---

### 3. Tentukan ukuran RAM yang ingin digunakan (opsional)

![Create Virtual Machine - Hardware / RAM](images/image3.png)

---

### 4. Tentukan Memori Hard Disk yang ingin digunakan (opsional)

![Create Virtual Machine - Virtual Hard Disk](images/image4.png)

---

### 5. Setelah selesai membuat servernya, masuk ke settingan servernya untuk mengatur beberapa, disini diharuskan mengupload file `.iso`

![Server Settings - Storage (upload ISO)](images/image5.png)

---

### 6. Pilih file `.iso` nya

![Optical Disk Selector - pilih file ISO](images/image6.png)

---

### 7. Setting pada bagian network, ke adapter 2 lalu jadikan Host-Only Adapter

![Settings - Network Adapter 2 Host-Only](images/image7.png)

---

### 8. Lalu setelah selesai semua klik **'OK'**, dan mulai servernya klik **'Start'**

### 9. Pilih **Install Oracle Linux 7.7** lalu Enter

![Oracle Linux 7.7 Boot Menu](images/image8.png)

---

### 10. Masukan bahasa

![Oracle Linux 7.7 Installation - Language Selection](images/image9.png)

**Isi Date Time sesuai dengan waktu dan tempat kalian**

![Oracle Linux 7.7 Installation Summary](images/image10.png)

**Atur User Creation menjadi 'Server with GUI'**

![Software Selection - Server with GUI](images/image11.png)

**Membuat password dan nama user creation-nya (wajib hafal passwordnya), sisanya sesuai default saja.**

![Configuration - Root Password & User Creation](images/image12.png)

Setelah selesai membuat password dan user creation-nya, tunggu instalasi selesai.

---

### 11. Selesai, klik **'Reboot'**

### 12. Accept License Information, lalu Finish Configuration

![Initial Setup - License Information](images/image13.png)

### 13. Login Servernya

### 14. Setelah Login, setting network-nya

![Applications Menu - System Tools](images/image14.png)

**Aktifkan enp0s3 dan enp0s8**

![Settings - Network (enp0s3 & enp0s8 ON)](images/image15.png)

**Pada enp0s8, aktifkan Connect Automatically dan setting Alamat servernya**

![Settings - Wired Details tab](images/image16.png)

![Settings - IPv4 Manual (192.168.56.10)](images/image17.png)

---

Setelah dari setting, masuk ke terminal dan masuk ke user root, lalu masukkan Alamat server kita yang tadi sudah kita setting agar terhubung servernya:

```bash
vi /etc/hosts
```

![Terminal - vi /etc/hosts](images/image18.png)

**Masukkan Alamat dan nama servernya**

![/etc/hosts - isi IP dan nama server](images/image19.png)

**Lalu hubungkan juga di:**

```bash
vi /etc/hostname
```

![Terminal - vi /etc/hostname](images/image20.png)

**Isi dengan nama server yang kamu buat**

![/etc/hostname - isi nama server (exaprimary)](images/image21.png)

**Setelah selesai semua, reboot**

![Terminal - reboot](images/image22.png)

---

### 15. Lalu kita mulai masuk ke server yang kita buka dengan menggunakan MobaXterm

**Klik New Session dan pilih SSH, masukkan Alamat servernya lalu 'OK'**

![MobaXterm - New SSH Session (192.168.56.10)](images/image23.png)

**Lalu masuk ke servernya dengan memasukkan nama user dan password yang sudah dibuat di awal tadi**

---

### 16. Masuk ke server root

```bash
su - root
# atau
su
```

---

### 17. Upload `dba.tar` dan file `.zip` Linux

---

### 18. AUTOMATIC SETUP (Install set up Oracle 19c)

```bash
yum install -y oracle-database-preinstall-19c
```

---

### 19. Setelah proses preinstall selesai, lakukan update pada server tersebut

```bash
yum update -y
```

> **Catatan:** Gunakan automatic setup jika hanya digunakan untuk lab.

---

### 20. AKTIFKAN X11

Setelah selesai memasukkan script di atas, selanjutnya kita install X11:

```bash
yum install xorg-x11-xauth xorg-x11-fonts* -y
```

Masuk ke filenya dengan menggunakan command:

```bash
vi /etc/ssh/sshd_config
```

![sshd_config - X11 Forwarding settings](images/image24.png)

**Lalu hapus `#` pada bagian X11**

---

### 21. Setelah itu lakukan:

```bash
systemctl restart sshd
```

![Terminal - systemctl restart sshd](images/image25.png)

---

### 22. Install packages berikut ini:

```bash
yum install -y bc
yum install -y binutils
yum install -y compat-libcap1
yum install -y compat-libstdc++-33
#yum install -y dtrace-modules
#yum install -y dtrace-modules-headers
#yum install -y dtrace-modules-provider-headers
yum install -y dtrace-utils
yum install -y elfutils-libelf
yum install -y elfutils-libelf-devel
yum install -y fontconfig-devel
yum install -y glibc
yum install -y glibc-devel
yum install -y ksh
yum install -y libaio
yum install -y libaio-devel
yum install -y libdtrace-ctf-devel
yum install -y libXrender
yum install -y libXrender-devel
yum install -y libX11
yum install -y libXau
yum install -y libXi
yum install -y libXtst
yum install -y libgcc
yum install -y librdmacm-devel
yum install -y libstdc++
yum install -y libstdc++-devel
yum install -y libxcb
yum install -y make
yum install -y net-tools        # Clusterware
yum install -y nfs-utils        # ACFS
yum install -y python           # ACFS
yum install -y python-configshell  # ACFS
yum install -y python-rtslib    # ACFS
yum install -y python-six       # ACFS
yum install -y targetcli        # ACFS
yum install -y smartmontools
yum install -y sysstat
yum install -y gcc
yum install -y unixODBC
```

---

### 23. Lalu buat grup dan users

```bash
groupadd -g 54321 oinstall
groupadd -g 54322 dba
groupadd -g 54323 oper
#groupadd -g 54324 backupdba
#groupadd -g 54325 dgdba
#groupadd -g 54326 kmdba
#groupadd -g 54327 asmdba
#groupadd -g 54328 asmoper
#groupadd -g 54329 asmadmin
#groupadd -g 54330 racdba

useradd -u 54321 -g oinstall -G dba,oper oracle
```

---

### 24. ADDITIONAl Setup pada SELINUX

```bash
vi /etc/selinux/config
```

![/etc/selinux/config - set SELINUX=permissive](images/image26.png)

**Lalu set menjadi `permissive`**

---

### 25. Setelah selesai masukkan command berikut ini:

```bash
setenforce Permissive
```

---

### 26. Setelah selesai, jika firewall masih enabled, kalian harus disable dengan command berikut ini:

```bash
systemctl stop firewalld
systemctl disable firewalld
```

---

### 27. Create directory tempat Oracle software akan diinstall

```bash
mkdir -p /u01/app/oracle/product/19.0.0/dbhome_1
mkdir -p /u02/oradata
chown -R oracle:oinstall /u01 /u02
chmod -R 775 /u01 /u02
```

---

### 28. Lalu masuk ke server oracle

```bash
su - oracle
```

---

### 29. Ekstrak file `dba.tar` di home directory dari oracle user di target database host

```bash
tar -xvf dba.tar
```

---

### 30. Masuk ke folder `dba`

```bash
cd dba
```

---

### 31. Menyalin (copy) file `act` ke dalam folder tujuan `/home/oracle`

```bash
cp act /home/oracle
```

---

### 32. Membuka dan mengedit file konfigurasi lingkungan (environment) database menggunakan text editor vi

```bash
vi .db_profile
```

**Sesuaikan nama database yang akan kamu buat dan juga lokasi filenya**

![.db_profile - konfigurasi Oracle environment](images/image27.png)

---

### 33. Set ke database yang kamu buat

![Terminal - sett command (pilih SID)](images/image28.png)

---

### 34. Masuk ke server root lagi

```bash
su - root
# atau
su
```

Gunakan perintah ini untuk membuat folder dan memberi izin akses ke user oracle:

```bash
mkdir -p /u01/app/oracle/product/19.0.0/dbhome_1 && \
chown -R oracle:oinstall /u01 && \
chmod -R 775 /u01
```

---

### 35. Cari Lokasi Pasti File `.zip`

Ketik perintah ini (sebagai root atau oracle) untuk menemukan file tersebut:

```bash
su - oracle
find / -name "LINUX.X64_193000_db_home.zip" 2>/dev/null
```

---

### 36. Masuk ke Folder Tujuan

Langkah ini wajib agar hasil ekstrak tidak berantakan:

```bash
cd /u01/app/oracle/product/19.0.0/dbhome_1/
```

---

### 37. Jalankan Unzip dengan Path Lengkap

```bash
unzip -oq /home/oracle/LINUX.X64_193000_db_home.zip
```

---

### 38. Instal Paket X11 (Sebagai root)

```bash
# Pindah ke root
su - root

# Instal xclock dan library pendukungnya
yum install -y xorg-x11-apps
```

---

### 39. Pastikan X11 Forwarding Aktif di MobaXterm

1. Klik kanan pada nama Session kamu di panel kiri MobaXterm → pilih **Edit Session**.
2. Pergi ke tab **Advanced SSH settings**.
3. Pastikan kotak **X11-Forwarding** sudah dicentang (checked).
4. Klik **OK**, lalu **Restart Session** (tutup terminal dan buka lagi).

---

### 40. Set Manual Tanpa xhost (Sebagai User oracle)

Jangan gunakan root, langsung saja ke user oracle. Jalankan perintah ini:

```bash
su - oracle

# Ganti IP di bawah dengan IP yang muncul di teks sambutan MobaXterm tadi
export DISPLAY=192.168.56.1:0.0
xclock
```

---

### 41. Lalu run Installer

```bash
cd /u01/app/oracle/product/19.0.0/dbhome_1
./runInstaller
```

![Oracle 19c Installer - Step 1: Select Configuration Option (Set Up Software Only)](images/image29.png)

![Oracle 19c Installer - Step 2: Database Installation Option (Single instance)](images/image30.png)

![Oracle 19c Installer - Step 3: Select Database Edition (Enterprise Edition)](images/image31.png)

![Oracle 19c Installer - Step 4: Specify Installation Location (/u01/app/oracle)](images/image32.png)

![Oracle 19c Installer - Step 5: Create Inventory (/u01/app/oraInventory)](images/image33.png)

![Oracle 19c Installer - Step 6: Privileged Operating System Groups](images/image34.png)

![Oracle 19c Installer - Step 7: Root Script Execution Configuration](images/image35.png)

![Oracle 19c Installer - Step 8: Perform Prerequisite Checks](images/image36.png)

![Oracle 19c Installer - Step 10: Install Product (Progress)](images/image37.png)

**Duplicate tab, masuk ke user root lalu jalankan script tersebut**

![MobaXterm - Execute Root Scripts (orainstRoot.sh & root.sh)](images/image38.png)

![Terminal - Running root.sh output](images/image39.png)

---

### 42. Lalu sebelum masuk ke DBCA, atur environment variable terlebih dahulu

```bash
su - oracle
nano ~/.bash_profile
```

Tambahkan baris ini di paling bawah:

```bash
export ORACLE_HOME=/u01/app/oracle/product/19.0.0/dbhome_1
export ORACLE_SID=orcl   # Sesuaikan dengan nama database di DBCA tadi
export PATH=$PATH:$ORACLE_HOME/bin
```

![nano ~/.bash_profile - environment variables](images/image40.png)

**Simpan `Ctrl+O`, Enter dan Keluar (`Ctrl+X`).**

Lalu aktifkan dengan:

```bash
source ~/.bash_profile
```

---

### 43. Masuk ke Oracle Home

```bash
cd $ORACLE_HOME
```

---

### 44. Membuat Database

```bash
dbca
```

![DBCA - Step 1: Select Database Operation (Create a database)](images/image41.png)

![DBCA - Step 2: Select Database Creation Mode (Typical configuration)](images/image42.png)

![DBCA - Progress Page (DB Creation in Progress)](images/image43.png)

![DBCA - Finish: Database Creation Complete](images/image44.png)

**Jika sudah seperti ini langsung Close saja.**

---

> **Database Information:**
> - Global Database Name: `orcl`
> - System Identifier (SID): `orcl`
> - Server Parameter File: `/u01/app/oracle/product/19.0.0/dbhome_1/dbs/spfileorcl.ora`
>
> **Catatan:** Semua akun database kecuali SYS dan SYSTEM terkunci. Gunakan Password Management untuk membuka akun yang akan digunakan, dan segera ubah password default setelah membuka akun.

---

*Selesai — Oracle Database 19c berhasil terinstall di Oracle Linux 7.7!*

---

*Dibuat oleh Naufal Ali Hilmi | Information Systems Student, Gunadarma University*
