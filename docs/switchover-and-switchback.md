# SWITCH OVER & SWITCH BACK

---

### 1. Cek status switchover di kedua server

```sql
SELECT DATABASE_ROLE, SWITCHOVER_STATUS FROM V$DATABASE;
```

Hasil di Primary : TO STANDBY
Hasil di Standby : NOT ALLOWED

---

### 2. Turunkan pangkat Primary menjadi Standby (di Primary)

```sql
ALTER DATABASE COMMIT TO SWITCHOVER TO STANDBY WITH SESSION SHUTDOWN;

STARTUP MOUNT;
```

Lalu, cek ulang statusnya:

```sql
SELECT DATABASE_ROLE, SWITCHOVER_STATUS FROM V$DATABASE;
```

Hasil seharusnya : PHYSICAL STANDBY  |  RECOVERY NEEDED

---

### 3. Pindah ke Standby — naikkan pangkat Standby menjadi Primary yang baru (di Standby)

```sql
ALTER DATABASE COMMIT TO SWITCHOVER TO PRIMARY WITH SESSION SHUTDOWN;

ALTER DATABASE OPEN;
```

Lalu, cek ulang statusnya:

```sql
SELECT DATABASE_ROLE, SWITCHOVER_STATUS FROM V$DATABASE;
```

Hasil seharusnya : PRIMARY

---

### 4. Verifikasi — buat tabel sederhana untuk memastikan sinkronisasi benar (di Primary baru)

```sql
CREATE TABLE test_switchover (id NUMBER, status VARCHAR2(20));

INSERT INTO test_switchover VALUES (1, 'Berhasil Tukar');
COMMIT;
```

Paksa log dikirim ke Standby yang baru:

```sql
ALTER SYSTEM SWITCH LOGFILE;
```

---

### 5. Cek di Standby yang baru

Aktifkan proses recovery jika belum aktif:

```sql
-- Buka database agar bisa melihat tabel yang sudah kita buat
ALTER DATABASE OPEN;

ALTER DATABASE RECOVER MANAGED STANDBY DATABASE DISCONNECT FROM SESSION;
```

Lalu cek apakah tabel tadi sudah sampai:

```sql
SELECT * FROM test_switchover;
```

Jika sudah ada, maka **Kita Sudah Berhasil melakukan Switch Over.**

---

### 6. Switch Back — kembalikan server ke kondisi semula

**Di Primary baru (exastandby):**

```sql
ALTER DATABASE COMMIT TO SWITCHOVER TO STANDBY WITH SESSION SHUTDOWN;

STARTUP MOUNT;

SELECT DATABASE_ROLE, SWITCHOVER_STATUS FROM V$DATABASE;
```

Hasil : PHYSICAL STANDBY  |  NOT ALLOWED

**Di Standby baru (exaprimary):**

```sql
ALTER DATABASE COMMIT TO SWITCHOVER TO PRIMARY WITH SESSION SHUTDOWN;

ALTER DATABASE OPEN;

SELECT DATABASE_ROLE, SWITCHOVER_STATUS FROM V$DATABASE;
```

Hasil : PRIMARY

Server yang seharusnya Primary — yang sempat menjadi Standby — sekarang sudah menjadi **Primary** lagi.

---

**SELAMAT!!!** Proses Switch Over dan Switch Back sudah berhasil…

---

*Dibuat oleh Naufal Ali Hilmi | Information Systems Student, Gunadarma University*
