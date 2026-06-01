# DATA GUARD SETUP

---

### 1. Masuk ke direktori file terlebih dahulu di Primary dan Standby

```bash
cd $ORACLE_HOME/network/admin
```

---

### 2. Membuat file `tnsnames.ora` dan `listener.ora` di Primary

```bash
vi tnsnames.ora
```

**=== Script `tnsnames.ora` (Primary) ===**

```
ORCL_PRIMARY =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.56.10)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )

ORCL_STANDBY =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.56.20)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl_standby)
    )
  )
```

```bash
vi listener.ora
```

**=== Script `listener.ora` (Primary) ===**

```
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.56.10)(PORT = 1521))
    )
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = orcl)
      (ORACLE_HOME = /u01/app/oracle/product/19.0.0/dbhome_1)
      (SID_NAME = orcl)
    )
  )
```

---

### 3. Membuat file `tnsnames.ora` dan `listener.ora` di Standby

```bash
vi tnsnames.ora
```

**=== Script `tnsnames.ora` (Standby) ===**

```
ORCL_PRIMARY =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.56.10)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )

ORCL_STANDBY =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.56.20)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl_standby)
    )
  )
```

```bash
vi listener.ora
```

**=== Script `listener.ora` (Standby) ===**

```
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.56.20)(PORT = 1521))
    )
  )

SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (GLOBAL_DBNAME = orcl_standby)
      (ORACLE_HOME = /u01/app/oracle/product/19.0.0/dbhome_1)
      (SID_NAME = orcl)
    )
  )
```

---

### 4. Lalu Nyalakan Listener di Kedua Server

```bash
lsnrctl start
```

---

### 5. Jika sudah success, Tes Panggil ke Primary (di Standby)

```bash
tnsping ORCL_PRIMARY
```

---

### 6. Kembali ke terminal Primary, tes panggil ke Standby

```bash
tnsping ORCL_STANDBY
```

Pastikan keduanya menjawab dengan pesan **OK**.

---

### 7. Kirim file password dari Primary ke Standby menggunakan `scp` (orapw)

Jalankan perintah ini di terminal **exaprimary**:

```bash
scp /u01/app/oracle/product/19.0.0/dbhome_1/dbs/orapworcl \
    oracle@192.168.56.20:/u01/app/oracle/product/19.0.0/dbhome_1/dbs/
```

---

### 8. Masuk ke Tahap Konfigurasi Database (SQL)

**-- Di Primary --**

```sql
sqlplus / as sysdba

startup

-- 1. Memberi identitas unik 'orcl'
ALTER SYSTEM SET DB_UNIQUE_NAME='orcl' SCOPE=SPFILE;

-- 2. Mengaktifkan fitur pengiriman log ke standby
ALTER SYSTEM SET LOG_ARCHIVE_CONFIG='DG_CONFIG=(orcl,orcl_standby)';

-- 3. Menentukan alamat pengiriman (ke alias ORCL_STANDBY)
ALTER SYSTEM SET LOG_ARCHIVE_DEST_2='SERVICE=ORCL_STANDBY NOAFFIRM ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=orcl_standby';

-- 4. Menentukan lokasi penerimaan jika suatu saat dia jadi standby
ALTER SYSTEM SET FAL_SERVER='ORCL_STANDBY';
```

**-- Di Standby --**

```sql
sqlplus / as sysdba

startup

-- 1. Memberi identitas unik 'orcl_standby'
ALTER SYSTEM SET DB_UNIQUE_NAME='orcl_standby' SCOPE=SPFILE;

-- 2. Mengaktifkan fitur sinkronisasi
ALTER SYSTEM SET LOG_ARCHIVE_CONFIG='DG_CONFIG=(orcl,orcl_standby)';

-- 3. Menentukan alamat pengiriman balik (jika nanti switchover)
ALTER SYSTEM SET LOG_ARCHIVE_DEST_2='SERVICE=ORCL_PRIMARY NOAFFIRM ASYNC VALID_FOR=(ONLINE_LOGFILES,PRIMARY_ROLE) DB_UNIQUE_NAME=orcl';

-- 4. Menentukan sumber data utama
ALTER SYSTEM SET FAL_SERVER='ORCL_PRIMARY';
```

---

### 9. Restart database di kedua server agar `DB_UNIQUE_NAME` yang tadi diatur di SPFILE bisa aktif

```sql
shut immediate;
startup;
```

---

### 10. Kita tinggal menyamakan kondisi terakhirnya agar mereka bisa sinkron secara otomatis.

---

### 11. Di Standby — Ubah Control File menjadi Standby

```sql
sqlplus / as sysdba

-- Matikan dulu untuk memastikan bersih
shut immediate;

-- Start ke mode NOMOUNT
startup nomount;

-- Pastikan DB_UNIQUE_NAME sudah benar (harusnya orcl_standby)
SHOW PARAMETER DB_UNIQUE_NAME;
```

---

### 12. Buat Standby Control File di Primary

```sql
sqlplus / as sysdba

ALTER DATABASE CREATE STANDBY CONTROLFILE AS '/tmp/standby_control.ctl';
```

---

### 13. Lalu kirim file tersebut ke server Standby

```bash
scp /tmp/standby_control.ctl oracle@192.168.56.20:/tmp/
```

---

### 14. Pindah ke terminal exastandby (192.168.56.20) — Timpa control file lama dengan yang baru

```sql
sqlplus / as sysdba

SHUT IMMEDIATE;
```

```bash
# Hapus file lama agar tidak ada konflik
rm /u01/app/oracle/oradata/ORCL/controlfile/o1_mf_o1g82g46_.ctl
rm /u01/app/oracle/fast_recovery_area/ORCL/controlfile/o1_mf_o1g82g8r_.ctl
```

> **Catatan:** Sesuaikan nama filenya dengan nama file kalian sendiri.

```bash
# Lalu timpa file nya
cp /tmp/standby_control.ctl /u01/app/oracle/oradata/ORCL/controlfile/o1_mf_o1g82g46_.ctl
cp /tmp/standby_control.ctl /u01/app/oracle/fast_recovery_area/ORCL/controlfile/o1_mf_o1g82g8r_.ctl
```

---

### 16. Setelah filenya diganti, masuk ke SQLPlus untuk menyalakan database dalam mode standby

```sql
sqlplus / as sysdba

-- Matikan dulu jika tadi masih menyala
SHUT IMMEDIATE;

-- Start ke mode MOUNT
STARTUP MOUNT;

-- Verifikasi peran database
SELECT DATABASE_ROLE FROM V$DATABASE;

-- Jalankan query ini untuk pembuktian
SELECT NAME, DATABASE_ROLE, CONTROLFILE_TYPE FROM V$DATABASE;
```

Hasilnya harus:

```
DATABASE_ROLE    : PHYSICAL STANDBY
CONTROLFILE_TYPE : STANDBY
```

```sql
-- Jalankan perintah ini untuk membiarkan Oracle mencari log yang dibutuhkan secara otomatis
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE DISCONNECT FROM SESSION;

-- Cek Status Proses (Monitoring)
SELECT PROCESS, STATUS, SEQUENCE# FROM V$MANAGED_STANDBY;

-- Jalankan MRP (Managed Recovery Process)
-- Perintah ini akan menyalakan "mesin" pengolah data di standby
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE DISCONNECT USING CURRENT LOGFILE;

-- Cek Statusnya
SELECT PROCESS, STATUS, SEQUENCE# FROM V$MANAGED_STANDBY;

-- Jika MRP0 masih belum terlihat, cek log error di standby
```

---

### 17. Cek Pesan Error di Alert Log

```bash
tail -n 100 /u01/app/oracle/diag/rdbms/orcl/orcl/trace/alert_orcl.log
```

Di server standby, `LOG_ARCHIVE_DEST_2` seharusnya diarahkan ke dirinya sendiri atau dikosongkan terlebih dahulu agar tidak mencoba mengirim log balik ke primary saat dia belum sinkron.

```sql
sqlplus / as sysdba

STARTUP MOUNT;
```

---

### 18. Tambahkan Standby Redo Log (SRL)

```sql
ALTER DATABASE ADD STANDBY LOGFILE GROUP 11 ('/u01/app/oracle/oradata/ORCL/standby_redo11.log') SIZE 200M;
ALTER DATABASE ADD STANDBY LOGFILE GROUP 12 ('/u01/app/oracle/oradata/ORCL/standby_redo12.log') SIZE 200M;
ALTER DATABASE ADD STANDBY LOGFILE GROUP 13 ('/u01/app/oracle/oradata/ORCL/standby_redo13.log') SIZE 200M;
ALTER DATABASE ADD STANDBY LOGFILE GROUP 14 ('/u01/app/oracle/oradata/ORCL/standby_redo14.log') SIZE 200M;
```

---

### 19. Masuk ke RMAN untuk sinkronisasi

```bash
rman target sys/123@ORCL_PRIMARY
```

---

### 20. Setelah masuk ke prompt RMAN, Reset Incarnation secara Manual untuk menata ulang strukturnya

```rman
reset database to incarnation 2;
```

---

### 21. Hentikan Semua Proses Recovery (di SQLPlus)

```sql
sqlplus / as sysdba

-- Hentikan proses managed recovery jika ada yang nyangkut
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;

-- Jika masih error, matikan database untuk melepaskan semua kunci (locks)
SHUT ABORT;
```

---

### 22. Di terminal Linux exastandby, pastikan tidak ada proses Oracle yang masih menggantung

```bash
ps -ef | grep ora_ | grep ORCL
```

---

### 23. Hapus File Fisik yang Bermasalah

```bash
rm /u01/app/oracle/oradata/ORCL/datafile/o1_mf_system_o1g7y09k_.dbf
```

---

### 24. Kill Semua Sesi & Proses

```bash
# Matikan paksa instance
sqlplus / as sysdba <<EOF
SHUT ABORT;
EOF

# Pastikan tidak ada proses background 'ora_' yang tersisa
ps -ef | grep ora_ | grep ORCL | awk '{print $2}' | xargs kill -9 2>/dev/null

# Hapus File Fisik (Eksekusi Mati)
rm -f /u01/app/oracle/oradata/ORCL/datafile/o1_mf_system_nzn824kh_.dbf
```

---

### 25. Jalankan Kembali ke Mode NOMOUNT

```sql
sqlplus / as sysdba

STARTUP NOMOUNT;

exit;
```

---

### 26. Masuk ke RMAN

```bash
rman target sys/123@ORCL_PRIMARY auxiliary sys/123@ORCL_STANDBY
```

---

### 27. Jalankan Duplicate dari RMAN

> **DUPLICATE** adalah solusi jika cara manual di poin 16 gagal karena masalah sinkronisasi file sistem.

```rman
DUPLICATE TARGET DATABASE FOR STANDBY FROM ACTIVE DATABASE NOFILENAMECHECK;
```

> **Note:** RMAN sudah melakukan semuanya dengan benar: menarik standby controlfile, melakukan restore semua datafile via jaringan, dan melakukan switch ke file-file yang baru. Masalah "hantu" lock dan orphan incarnation tadi sudah terkubur bersama file-file lama yang ditimpa oleh RMAN.
>
> Sekarang database kamu sudah dalam kondisi MOUNT dengan struktur yang benar-benar identik dengan Primary.

---

### 28. Buka Database (Active Data Guard)

```sql
sqlplus / as sysdba

ALTER DATABASE OPEN READ ONLY;
```

> **Catatan:** Menjalankan `OPEN READ ONLY` sebelum `RECOVER MANAGED` adalah kunci agar database menjadi **Active Data Guard**.

---

### 29. Tambahkan Standby Redo Log (SRL)

```sql
ALTER DATABASE ADD STANDBY LOGFILE GROUP 11 ('/u01/app/oracle/oradata/ORCL_STANDBY/datafile/srl_11.log') SIZE 200M;
ALTER DATABASE ADD STANDBY LOGFILE GROUP 12 ('/u01/app/oracle/oradata/ORCL_STANDBY/datafile/srl_12.log') SIZE 200M;
ALTER DATABASE ADD STANDBY LOGFILE GROUP 13 ('/u01/app/oracle/oradata/ORCL_STANDBY/datafile/srl_13.log') SIZE 200M;
ALTER DATABASE ADD STANDBY LOGFILE GROUP 14 ('/u01/app/oracle/oradata/ORCL_STANDBY/datafile/srl_14.log') SIZE 200M;
```

---

### 30. Nyalakan Managed Recovery (MRP)

```sql
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE DISCONNECT USING CURRENT LOGFILE;
```

---

### 31. Cara Verifikasi untuk memastikan "mesin" sudah menyala

```sql
SELECT PROCESS, STATUS, SEQUENCE# FROM V$MANAGED_STANDBY WHERE PROCESS LIKE 'MRP%';
```

---

### 32. Tes Pengiriman Log (DI PRIMARY)

```sql
-- Di SQL exaprimary
ALTER SYSTEM SWITCH LOGFILE;
```

---

### 33. Cek Perubahan (DI STANDBY)

```sql
-- Di SQL exastandby
SELECT PROCESS, STATUS, SEQUENCE# FROM V$MANAGED_STANDBY WHERE PROCESS LIKE 'MRP%';
```

> Angka **SEQUENCE#** — jika naik, berarti sinkronisasi log sudah berjalan otomatis.

---

### Dokumentasi: Mengecek Sinkronisasi Perubahan dari Primary ke Standby

```sql
-- Di Primary --
CREATE TABLE tes_sinkron (id NUMBER, waktu DATE, pesan VARCHAR2(50));
INSERT INTO tes_sinkron VALUES (1, SYSDATE, 'Berhasil Sinkron Gan!');
COMMIT;
ALTER SYSTEM SWITCH LOGFILE;

-- Di Standby --
SELECT * FROM tes_sinkron;
```

**Hasil:** Jika hasilnya keluar di standby, maka sinkronisasi sudah berhasil.

---

### 34. Monitoring — Cek Gap antara Primary dan Standby (jalankan di Standby)

```sql
SELECT ARCHIVED_THREAD#, ARCHIVED_SEQ#, APPLIED_THREAD#, APPLIED_SEQ#
FROM V$ARCHIVE_DEST_STATUS;

SELECT name, value FROM v$dataguard_stats WHERE name LIKE 'apply lag';
```

---

### 35. Paksa Sinkronisasi (DI PRIMARY)

```sql
ALTER SYSTEM ARCHIVE LOG CURRENT;
```

---

### 36. Cek Status Final (DI STANDBY)

```sql
SELECT ARCHIVED_THREAD#, ARCHIVED_SEQ#, APPLIED_THREAD#, APPLIED_SEQ#
FROM V$ARCHIVE_DEST_STATUS
WHERE ARCHIVED_SEQ# > 0;
```

---

### 37. Langkah Penutupan — Pengecekan "identitas" terakhir di exastandby

```sql
SELECT DATABASE_ROLE, OPEN_MODE, PROTECTION_MODE, PROTECTION_LEVEL
FROM V$DATABASE;
```

Hasil yang seharusnya muncul:

```
DATABASE_ROLE    : PHYSICAL STANDBY
OPEN_MODE        : READ ONLY WITH APPLY
                   (Artinya ini Active Data Guard — bisa query sambil sinkronisasi jalan)
PROTECTION_MODE  : MAXIMUM PERFORMANCE (Default standar)
PROTECTION_LEVEL : MAXIMUM PERFORMANCE
```

---

### 38. Apa yang harus dilakukan jika Sequence tidak naik-naik?

Jika setelah 5 menit `APPLIED_SEQ#` masih tetap 0, cukup jalankan ini di **exastandby**:

```sql
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE CANCEL;
ALTER DATABASE RECOVER MANAGED STANDBY DATABASE DISCONNECT USING CURRENT LOGFILE;
```

---

*Dibuat oleh Naufal Ali Hilmi | Information Systems Student, Gunadarma University*
