# Snapshot Standby

Dengan fitur **Snapshot Standby**, kita akan menyulap database cadangan ini menjadi **READ-WRITE**.

- Kita bisa membuat tabel baru, menghapus data, atau menjalankan testing aplikasi secara brutal di `orcl_standby`.
- Sementara itu di latar belakang, Oracle akan tetap menerima Redo Log dari Primary (sehingga data aslinya tetap aman dan tidak tertinggal), tapi log tersebut **ditahan** dan tidak diaplikasikan ke Standby.
- Begitu testing selesai, kamu tinggal mengetik satu perintah. Oracle akan langsung menghapus semua data eksperimen kita, mengembalikan database ke detik sebelum diubah, dan langsung melanjutkan aplikasi Redo Log yang sempat tertahan tadi.

> **Catatan:** Fitur ini sangat bergantung pada fitur **Flashback Database**, sehingga Fast Recovery Area / FRA di server Standby-mu wajib dalam kondisi aktif.

---

## Langkah-Langkah Menggunakan Snapshot Standby

### 1. Masuk ke DGMGRL

Bisa di server mana saja, tapi karena kita sudah menggunakan server standby sebagai observer, lebih baik dijalankan di **Primary**.

```bash
dgmgrl sys/123
```

---

### 2. Matikan FSFO

```sql
disable fast_start failover;
```

---

### 3. Ubah Standby Menjadi Snapshot Standby

```sql
convert database orcl_standby to snapshot standby;
```

---

### 4. Verifikasi Perubahan

```sql
show configuration;
```

---

### 5. Cek Status Database (di SQL Standby)

```sql
select name, database_role, open_mode from v$database;
```

Jika terjadi error koneksi, jalankan:

```sql
connect / as sysdba
```

---

### 6. Bereksperimen di Snapshot Standby

Jika statusnya sudah **Snapshot**, kita bisa mulai bereksperimen:

```sql
-- Membuat tabel contoh
CREATE TABLE tabel_test (id NUMBER, pesan VARCHAR2(50));

-- Memasukkan data simulasi aplikasi
INSERT INTO tabel_test VALUES (1, 'Testing Aplikasi Aman di Snapshot Standby');

COMMIT;

-- Tampilkan datanya untuk memastikan data tersimpan
SELECT * FROM tabel_test;
```

---

### 7. Verifikasi di SQL Primary

Coba lihat apakah tabel tersebut ada di Primary:

```sql
SELECT * FROM tabel_test;
```

Hasilnya akan error:
```
ORA-00942: table or view does not exist
```

Karena kita melakukannya di Snapshot Standby — itulah gunanya fitur ini!

---

### 8. Kembalikan ke Physical Standby

Kalau sudah selesai eksperimen, ubah status Standby menjadi **Mount** terlebih dahulu (di SQL Standby):

```sql
shut immediate;
startup mount;
```

Lalu kembalikan status standby seperti semula melalui DGMGRL:

```bash
dgmgrl sys/123
```

```sql
convert database orcl_standby to physical standby;
```

---

### 9. Cek Status Setelah Konversi (di SQL Standby)

```sql
select name, database_role, open_mode from v$database;
```

---

### 10. Aktifkan Kembali FSFO

```sql
enable fast_start failover;
```

---

### 11. Validasi Status Akhir

```sql
show configuration;
```

---

*Dibuat oleh **Naufal Ali Hilmi** | Information Systems Student, Gunadarma University*
