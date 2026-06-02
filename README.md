# Oracle DBA Portfolio - Naufal Ali Hilmi

Selamat datang di repositori portofolio teknis saya. Repositori ini berisi dokumentasi praktik laboratorium dan *troubleshooting* pada **Oracle Database 19c Enterprise Edition** yang saya kerjakan di lingkungan Linux (Oracle Linux).

## 🛠 Tech Stack
- **Database:** Oracle 19c Enterprise Edition
- **Tools:** SQL*Plus, RMAN (Recovery Manager), Data Guard Broker, Switchover, Switchback, & Failover
- **OS:** Linux (Oracle Linux)
- **Networking:** Oracle Net Services (Listener, TNS)

## 📂 Learning Path & Projects

| Proyek | Deskripsi |
| :--- | :--- |
| [Instalasi Oracle 19c](docs/installation-oracle-19c.md) | Panduan instalasi dan konfigurasi dasar pada Linux. |
| [Backup & Restore (Single)](docs/backup-restore-single.md) | Implementasi strategi backup RMAN pada satu server. |
| [Network Recovery](docs/backup-restore-cross.md) | Restore database antar server menggunakan Oracle Net Services. |
| [Data Guard (HA)](docs/dataguard-setup.md) | Konfigurasi *Active Data Guard* untuk *High Availability* & *Disaster Recovery*. |
| [Switchover & Switchback](docs/switchover-and-switchback.md) | Manajemen *Role Transition* terencana untuk pemeliharaan rutin (*planned maintenance*). |
| [Failover Labs](docs/failover-manual-and-reinstatement.md) | Simulasi penanganan insiden darurat (*unplanned outage*) untuk memindahkan peran ke standby database. |



## 🚀 Highlights & Troubleshooting
Dalam setiap lab, saya fokus tidak hanya pada langkah "Happy Path", tetapi juga melakukan dokumentasi terhadap error riil yang ditemui, seperti:
- **`ORA-19573`** (Datafile Lock issues)
- **`ORA-19909`** (Orphan Incarnation issues)
- **`RMAN-05146`** (Network connectivity troubleshooting)

*Tujuan utama saya adalah memahami cara kerja database di level infrastruktur.*

*Dibuat oleh Naufal Ali Hilmi | Information Systems Student, Gunadarma University*
