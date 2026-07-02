# Oracle DBA Portfolio - Naufal Ali Hilmi

Selamat datang di repositori portofolio teknis saya. Repositori ini berisi dokumentasi praktik laboratorium dan *troubleshooting* pada **Oracle Database 19c Enterprise Edition** yang saya kerjakan di lingkungan Linux (Oracle Linux), mulai dari instalasi hingga skenario *High Availability* & *Disaster Recovery* tingkat lanjut.

## 🛠 Tech Stack
- **Database:** Oracle 19c Enterprise Edition
- **Tools:** SQL*Plus, RMAN (Recovery Manager), Data Guard Broker (DGMGRL), Fast Start Failover (Observer), Snapshot Standby
- **OS:** Linux (Oracle Linux)
- **Networking:** Oracle Net Services (Listener, TNS)

## 📂 Learning Path & Projects

### Foundation
| Proyek | Deskripsi |
| :--- | :--- |
| [Instalasi Oracle 19c](docs/installation-oracle-19c.md) | Panduan instalasi dan konfigurasi dasar pada Linux. |
| [Backup & Restore (Single)](docs/backup-restore-single.md) | Implementasi strategi backup RMAN pada satu server. |
| [Network Recovery](docs/backup-restore-cross.md) | Restore database antar server menggunakan Oracle Net Services. |

### High Availability & Disaster Recovery (Data Guard)
| Proyek | Deskripsi |
| :--- | :--- |
| [Data Guard (HA)](docs/dataguard-setup.md) | Konfigurasi *Active Data Guard* untuk *High Availability* & *Disaster Recovery*. |
| [Switchover & Switchback](docs/switchover-and-switchback.md) | Manajemen *Role Transition* terencana untuk pemeliharaan rutin (*planned maintenance*). |
| [Failover Labs](docs/failover-manual-and-reinstatement.md) | Simulasi penanganan insiden darurat (*unplanned outage*) untuk memindahkan peran ke standby database, beserta proses *reinstatement*. |
| [Automasi Data Guard Broker (DGMGRL)](docs/automasi-oracle-data-guard-broker.md) | Konfigurasi DGMGRL agar *role transition* cukup dieksekusi dengan satu perintah. |
| [Fast Start Failover (FSFO)](docs/fast-start-failover.md) | Konfigurasi *Observer* agar failover ke standby berjalan otomatis tanpa intervensi manual saat primary down. |
| [Snapshot Standby](docs/snapshot-standby.md) | Mengubah database standby menjadi *read-write* untuk kebutuhan testing tanpa mengorbankan sinkronisasi data. |


### Linux System Administration
| Proyek | Deskripsi |
| :--- | :--- |
| [Journey: System Administrator](docs/journey-system-administrator.md) | Dasar-dasar administrasi Linux (manajemen user & hak akses) sebagai fondasi sebelum mengelola server Oracle. |

## 🚀 Highlights & Troubleshooting
Dalam setiap lab, saya fokus tidak hanya pada langkah "Happy Path", tetapi juga melakukan dokumentasi terhadap error riil yang ditemui, seperti:
- **`ORA-19573`** (Datafile Lock issues)
- **`ORA-19909`** (Orphan Incarnation issues)
- **`RMAN-05146`** (Network connectivity troubleshooting)

*Tujuan utama saya adalah memahami cara kerja database di level infrastruktur.*

*Dibuat oleh Naufal Ali Hilmi | Information Systems Student, Gunadarma University*
