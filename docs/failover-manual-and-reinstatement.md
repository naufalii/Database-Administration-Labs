# Failover Manual and Reinstatement



## Page 1


FAIL OVER 
Ini terjadi Jika tiba-tiba gedung data center tempat server utama kita (primary) berada 
mengalami mati lampu total, korsleting, atau tersambar petir. Server utama mati total dan tidak 
bisa dihubungi sama sekali. 
 
Sebagai DBA yang bertugas, kita tidak punya waktu untuk melakukan Switchover (karena server 
utama tidak bisa merespons perintah serah terima jabatan). Di sinilah kita harus melakukan 
Failover. 
Mari kita simulasikan bencana ini : 
 
1. Kita akan mematikan paksa server utama seolah-olah hardware nya rusak mendadak. Buka 
terminal server primary kita, lalu hancurkan operasinya dengan perintah: 
- shut abort; 
 
Setelah itu, tutup terminal primary kita. Jangan disentuh lagi, anggap server ini sudah hangus 
terbakar. 
Lalu diproses inilah kita mengganti server standby kita menjadi primary dengan paksa : 
 
2. Cek status internal standby 
- select database_role, switchover_status from v$database; 
 
Karena Primary mati mendadak, statusnya pasti tidak akan menunjukkan ‘TO PRIMARY’ secara 
normal. 
 
3. Hentikan proses Managed Recovery (MRP) yang sedang kebingungan mencari Primary 
- alter database recover managed standby database cancel; 
 
Jika hasilnya ‘ORA-16136: Managed Standby Recovery not active’ tidak apa-apa karena server 
Standby kita memang sudah otomatis mati atau tidak sedang aktif saat kamu mematikan paksa 
server Primary tadi. 
 
4. Paksa Standby mengambil alih (Eksekusi Failover): 
- alter database activate physical standby database; 
 
5. Buka database agar aplikasi bisa bertransaksi Kembali 
- alter database open;


## Page 2


6. Lalu cek hasilnya, apakah server Standby nya sudah berhasil mengambil alih menjadi 
PRIMARY yang baru atau belum 
- select database_role, switchover_status from v$database; 
 
Jika hasilnya sudah menjadi PRIMARY, maka kita BERHASIL!!!. Sudah melakukan Failover tetapi 
belum selesai karena kita masih harus menyalakan primary kita Kembali, dan jadikan primary 
kita yang asli Kembali jadi server primary utama. 
 
Ada 2 Skenario disini, dengan cara Manual (karena belum melakukan Flashback) dan dengan 
cara Reinstatement (sudah melakukan Flashback sebelumnya). 
Disini kita akan melakukan ke 2 nya karena untuk belajar, pertama kita gunakan cara manual 
karena kemungkinan kita lupa/belum tau apa sebelumnya itu Flashback. 
 
Cek status sudah Flashback atau belum: 
- select flashback_on from v$database; 
 
Jika belum maka kita hidupkan, agar selanjutnya kita tidak perlu dengan cara Manual lagi untuk 
buang-buang waktu : 
- alter database flashback on; 
 
Cara Manual (No Flashback) : 
1. Masuk ke server primary, buka SQL 
- startup mount; 
- alter database convert to physical standby;  
- shut immediate;  
- startup mount; 
 
2. Sinkronisasi Data 
- alter database recover managed standby database disconnect from session; 
 
3. Turunkan pangkat di Server Primary Saat Ini (Mantan Standby) 
- alter database commit to switchover to standby with session shutdown; 
- startup mount; 
 
4. Cek statusnya di server utama kita (primary yang saat ini belum menjadi primary lagi) 
- select database_role, switchover_status from v$database; 
 
Hasil seharusnya : TO PRIMARY  |  SESSIONS ACTIVE.


## Page 3


5. Matikan proses recovery (MRP) manualnya terlebih dahulu agar tidak memicu error 
- alter database recover managed standby database cancel; 
 
6. Ubah status sql menjadi nomount 
- shut immediate; 
- startup nomount; 
 
7. Masuk ke rman untuk mengambil data langsung dari server sebelah (Primary yang sekarang 
aktif) untuk menimpa dan memperbaiki exaprimary. 
- rman TARGET sys/password_kamu@TNS_PRIMARY_SEBELAH AUXILIARY 
sys/password_kamu@TNS_EXAPRIMARY 
 
8. eksekusi perintah duplikasi 
- DUPLICATE TARGET DATABASE FOR STANDBY FROM ACTIVE DATABASE NOFILENAMECHECK; 
 
9. Jika sudah berhasil, keluar rman dan lanjut untuk mengubah status server kita (masuk ke SQL) 
- shut immediate; 
- startup mount; 
 
10. Sinkronisasi data seperti biasa untuk mengecek apakah bisa terhubung atau tidak 
- alter database recover managed standby database disconnect from session; 
 
Lalu cek: 
-  SELECT PROCESS, STATUS, SEQUENCE# FROM V$MANAGED_STANDBY WHERE PROCESS LIKE 
'MRP%'; 
 
11. Setelah sudah tau terhubung, putuskan lagi untuk kita lanjut mengubah status servernya 
- alter database recover managed standby database cancel; 
 
12. Sekarang naikkan pangkat server kita menjadi primary 
- alter database commit to switchover to primary with session shutdown; 
- alter database open; 
 
13. Cek Statusnya 
- select database_role, switchover_status from v$database; 
 
Jika statusnya sudah PRIMARY  |  TO STANDBY  kita sudah berhasil!!!.


## Page 4


Jadi, kita sudah melakukan Failover sampai kembali seperti semula sebelum kejadian bencana 
terjadi. 
 
 
Cara Reinstatement (Flashback) : 
1. Kita mulai dari awal banget, menyiptakan simulasi bencana sendiri 
- shut abort; 
 
2. Failover (Naikkan Standby Menjadi Primary), Matikan proses standby yang sedang menunggu 
exaprimary 
- alter database recover managed standby database cancel; 
 
3. Terapkan sisa log darurat yang masih ada 
- alter database recover managed standby database finish; 
 
4. Paksa naik pangkat menjadi Primary 
- alter database commit to switchover to primary with session shutdown; 
 
5. Buka database agar aplikasi bisa jalan lagi 
- alter database open; 
 
6. Sekarang Standby sudah beralih jadi Primary, Lalu saatnya kita menggunakan cara 
Reinstatement 
- select standby_became_primary_scn from v$database; 
 
Catat angka yang muncul dari kueri, contoh:
 
 
7. Lalu sekarang masuk ke server utama kita, nyalakan sebagai mount 
- startup mount 
 
8. Lalu, gunakan kekuatan Flashback menggunakan angka SCN yang didapat tadi 
- flashback database to scn ANGKA_SCN;


![Image 1](images/page_4_img_1.png)



## Page 5


9. Setelah berhasil waktunya diputar mundur ke masa sebelum perpecahan, sekarang kita bisa 
mengubah perannya dengan aman 
- alter database convert to physical standby; 
 
10. Restart singkat untuk menyegarkan memori control file 
- shut immediate; 
- startup mount; 
 
11. Nyalakan proses sinkronisasi ke Primary yang baru 
- alter database recover managed standby database disconnect from session; 
 
12. cek apakah exaprimary berhasil terconnect dan berstatus WAIT_FOR_LOG 
- SELECT PROCESS, STATUS, SEQUENCE# FROM V$MANAGED_STANDBY WHERE PROCESS LIKE 
'MRP%'; 
 
13. Jika sudah terconnect, maka tinggal kita turunkan Pangkat exastandby (Primary Saat Ini) 
- alter database commit to switchover to standby with session shutdown; 
- startup mount; 
 
14. Naikkan Pangkat exaprimary (Standby yang Mau Jadi Primary Lagi) 
- alter database recover managed standby database cancel; 
- alter database commit to switchover to primary with session shutdown; 
- alter database open; 
 
15. Jika sudah berhasil, kembali ke server exastandby (yang sekarang sudah jadi Standby 
kembali), lalu nyalakan proses recovery-nya agar dia memantau exaprimary: 
- alter database recover managed standby database disconnect from session; 
 
16. Terakhir, Cek statusnya dikedua Server 
- select database_role, switchover_status from v$database; 
 
Jadi, kita sudah BERHASIL melakukan Failover dengan cara Reinstatement menggunakan 
Flashback. 
Jika dibandingkan dengan cara manual, cara ini lebih efisien dan simple. 
 
 
Dibuat oleh Naufal Ali Hilmi | Information Systems Student, Gunadarma University


## Page 6
