# Laporan Penyelesaian Tugas Modul 5: Implementasi Jaringan Enterprise HQ–Branch

## 1. Topologi Jaringan

Topologi ini mensimulasikan jaringan *enterprise* yang menghubungkan Kantor Pusat (HQ) di Jakarta dan Kantor Cabang (Branch) di Surabaya. Kedua lokasi dihubungkan melalui infrastruktur *Internet Service Provider* (ISP) menggunakan teknologi *Generic Routing Encapsulation* (GRE) Tunnel, yang dikombinasikan dengan *Open Shortest Path First* (OSPF) untuk pertukaran rute secara dinamis. Jaringan HQ juga mengimplementasikan redundansi *gateway* menggunakan *Virtual Router Redundancy Protocol* (VRRP) dan layanan *Dynamic Host Configuration Protocol* (DHCP) terpusat.

(<img width="768" height="548" alt="Screenshot 2026-06-11 055102" src="https://github.com/user-attachments/assets/60237d05-e09d-48a3-8980-d0fec996c66c" />)


---

## 2. Tabel IP Address

Berikut adalah skema pengalamatan IP (*IP Addressing*) yang digunakan pada infrastruktur jaringan:

### 2.1 Addressing Jakarta / HQ

**Tabel VLAN Jakarta**
| VLAN | Nama VLAN | Network | Gateway Virtual | Keterangan |
| :--- | :--- | :--- | :--- | :--- |
| 10 | FINANCE | 192.168.10.0/24 | 192.168.10.1 | DHCP dari Ubuntu Server Jakarta |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 | DHCP dari Ubuntu Server Jakarta |
| 60 | SERVER-HQ | 192.168.60.0/24 | 192.168.60.1 | VLAN server Ubuntu Jakarta |

**Tabel Interface Router (Cisco & MikroTik) Jakarta**
| Perangkat | Interface | IP Address | Keterangan |
| :--- | :--- | :--- | :--- |
| **Cisco** | Gi0/1.10 (VLAN 10) | 192.168.10.2/24 | IP fisik Cisco |
| **Cisco** | Gi0/1.20 (VLAN 20) | 192.168.20.2/24 | IP fisik Cisco |
| **Cisco** | Gi0/1.60 (VLAN 60) | 192.168.60.2/24 | IP fisik Cisco |
| **Cisco** | Gi0/0 | 10.10.100.2/30 | Link transit ke FortiGate |
| **MikroTik**| vlan10-finance | 192.168.10.3/24 | IP fisik MikroTik |
| **MikroTik**| vlan20-it | 192.168.20.3/24 | IP fisik MikroTik |
| **MikroTik**| vlan60-server | 192.168.60.3/24 | IP fisik MikroTik |
| **MikroTik**| ether1 | 10.10.101.2/30 | Link transit ke FortiGate |

**Tabel VRRP Jakarta**
| VLAN | Virtual IP | Master | Backup |
| :--- | :--- | :--- | :--- |
| 10 | 192.168.10.1 | Cisco Router Jakarta | MikroTik Router Jakarta |
| 20 | 192.168.20.1 | MikroTik Router Jakarta | Cisco Router Jakarta |
| 60 | 192.168.60.1 | Cisco Router Jakarta | MikroTik Router Jakarta |

### 2.2 Addressing ISP dan FortiGate

**Tabel FortiGate Jakarta & Surabaya**
| Perangkat | Interface | IP Address | Terhubung Ke |
| :--- | :--- | :--- | :--- |
| **FG Jakarta** | port1 | 10.10.100.1/30 | Cisco Router Jakarta |
| **FG Jakarta** | port2 | 10.10.101.1/30 | MikroTik Router Jakarta |
| **FG Jakarta** | port3 (WAN) | 10.0.12.2/30 | MikroTik ISP |
| **FG Jakarta** | GRE-JKT-SBY | 172.16.0.1/32 | Tunnel ke Surabaya |
| **FG Surabaya**| port1 (WAN) | 10.0.13.2/30 | MikroTik ISP |
| **FG Surabaya**| port2 | 10.10.200.1/30 | MikroTik Surabaya |
| **FG Surabaya**| GRE-SBY-JKT | 172.16.0.2/32 | Tunnel ke Jakarta |

**Tabel MikroTik ISP**
| Interface | Terhubung ke | IP Address |
| :--- | :--- | :--- |
| ether2 | FortiGate Jakarta | 10.0.12.1/30 |
| ether3 | FortiGate Surabaya | 10.0.13.1/30 |
| ether1 | Cloud NAT / Internet | DHCP |

### 2.3 Addressing Surabaya / Branch

**Tabel VLAN dan MikroTik Surabaya**
| VLAN | Nama VLAN | Network | Gateway (MikroTik SBY) | Keterangan |
| :--- | :--- | :--- | :--- | :--- |
| 30 | SALES | 192.168.30.0/24 | 192.168.30.1 | DHCP Server lokal |
| 40 | OPERATIONS | 192.168.40.0/24 | 192.168.40.1 | IP Static |

---

## 3. Konfigurasi Tiap Perangkat

Berikut adalah rangkuman penyelesaian konfigurasi pada masing-masing perangkat jaringan.

### 3.1. Cisco Switch Jakarta
Penyediaan VLAN 10, 20, dan 60 serta alokasi *access port* ke klien dan *trunk port* yang mengarah ke Cisco Router dan MikroTik Router.

<img width="1145" height="746" alt="WhatsApp Image 2026-06-10 at 8 29 26 PM" src="https://github.com/user-attachments/assets/cf7bfd15-5215-4bc9-80a1-f2098affc91d" />

### 3.2. Cisco Router Jakarta
Pembuatan *subinterface*, konfigurasi IP fisik, penetapan VRRP (sebagai Master untuk VLAN 10 dan 60), serta konfigurasi `ip helper-address` (DHCP Relay) menuju Ubuntu Server `192.168.60.10`. Diakhiri dengan pengaturan *default route* menuju FortiGate Jakarta (`10.10.100.1`).
<img width="1137" height="747" alt="WhatsApp Image 2026-06-10 at 8 29 26 PM (1)" src="https://github.com/user-attachments/assets/c5b8cdeb-b6ca-41f7-a617-f13231ace2bc" />

### 3.3. MikroTik Router Jakarta
Pembuatan interface VLAN, konfigurasi IP fisik, penetapan VRRP (sebagai Master untuk VLAN 20), dan konfigurasi DHCP Relay. Penetapan *default route* diarahkan ke FortiGate Jakarta (`10.10.101.1`).

<img width="1143" height="926" alt="WhatsApp Image 2026-06-10 at 8 29 26 PM (2)" src="https://github.com/user-attachments/assets/db7a8725-dbde-49df-a3b3-d62de9abbbdb" />
<img width="1146" height="926" alt="WhatsApp Image 2026-06-10 at 8 29 27 PM" src="https://github.com/user-attachments/assets/391d2555-fc64-4e29-865c-201ce1f51b61" />

### 3.4. Ubuntu Server Jakarta
Konfigurasi *static IP* `192.168.60.10` via Netplan dengan gateway IP Virtual VRRP (`192.168.60.1`). Instalasi `isc-dhcp-server` dan konfigurasi *pool* untuk VLAN 10 dan 20. Selanjutnya dilakukan instalasi Nginx sebagai layanan web server internal.
<img width="1258" height="1175" alt="WhatsApp Image 2026-06-10 at 10 44 23 PM" src="https://github.com/user-attachments/assets/0daec42f-d18f-4693-8e4d-5855e3866dc9" />


### 3.5. FortiGate Jakarta
Konfigurasi interface mengarah ke LAN (Cisco & MikroTik) dan WAN (ISP). Pembuatan *Firewall Policy* untuk *allow traffic* dari LAN menuju Internet dengan NAT aktif. Konfigurasi GRE Tunnel (`172.16.0.1`) menuju IP publik FortiGate Surabaya, dan menjalankan OSPF di atas GRE Tunnel dengan *Redistribute Static* diaktifkan.
<img width="1266" height="1243" alt="WhatsApp Image 2026-06-10 at 10 44 24 PM" src="https://github.com/user-attachments/assets/5c930d8d-a301-48a2-bbae-cae00bb3f51e" />
<img width="1248" height="1210" alt="WhatsApp Image 2026-06-10 at 10 44 25 PM (1)" src="https://github.com/user-attachments/assets/2f6d03b2-791b-42b4-915e-7a4b11261bb4" />
<img width="1243" height="1235" alt="WhatsApp Image 2026-06-10 at 10 44 25 PM" src="https://github.com/user-attachments/assets/cffebf85-34b9-49c6-95f3-df3a140d329c" />
<img width="1248" height="1244" alt="WhatsApp Image 2026-06-10 at 10 44 26 PM" src="https://github.com/user-attachments/assets/ba9ccbc3-4ffd-458e-8bf0-4cfb0d86726e" />

### 3.6. MikroTik ISP
Konfigurasi IP penghubung antar FortiGate dan DHCP Client mengarah ke Internet (Cloud PNETLab). Konfigurasi `NAT Masquerade` pada `ether1` (Internet).
<img width="1258" height="1242" alt="WhatsApp Image 2026-06-10 at 10 44 26 PM (1)" src="https://github.com/user-attachments/assets/612fae09-e3d4-4b18-a831-4975f00a28bc" />


### 3.7. Cisco Switch & MikroTik Surabaya
Pembuatan VLAN 30 dan 40 di Cisco Switch Surabaya (*trunk* mengarah ke MikroTik). Pada MikroTik Surabaya, dikonfigurasi IP Gateway, layanan DHCP Server lokal untuk VLAN 30, serta *default route* menuju FortiGate Surabaya (`10.10.200.1`).
<img width="1254" height="1227" alt="WhatsApp Image 2026-06-10 at 10 44 26 PM (2)" src="https://github.com/user-attachments/assets/f817ff65-f551-4acb-8f16-84ddd34f7c1b" />


### 3.8. FortiGate Surabaya
Konfigurasi interface WAN mengarah ke MikroTik ISP dan interface LAN mengarah ke MikroTik Surabaya. Pembuatan kebijakan keamanan (*Firewall Policy*) dengan NAT untuk akses internet. Konfigurasi GRE Tunnel (`172.16.0.2`) menuju FortiGate Jakarta dan aktivasi *routing* OSPF *over* GRE.
<img width="1263" height="1256" alt="WhatsApp Image 2026-06-10 at 10 44 27 PM (2)" src="https://github.com/user-attachments/assets/df1ec264-0424-47fc-ba68-22b91c82b0cd" />



---

## 4. Hasil Pengujian

Pengujian dilakukan dari *end-device* (VPCs dan Tinycore) secara menyeluruh untuk memvalidasi kelancaran seluruh protokol.

**1. Verifikasi Distribusi IP DHCP (Jakarta & Surabaya)**
Klien pada VLAN 10 (Jakarta) dan VLAN 30 (Surabaya) berhasil mendapatkan IP Address dinamis dari masing-masing DHCP Server (Ubuntu dan MikroTik lokal).
<img width="1270" height="1320" alt="WhatsApp Image 2026-06-10 at 10 50 28 PM (2)" src="https://github.com/user-attachments/assets/711e9b4f-e4b9-4294-901b-147e157adc5c" />
<img width="1266" height="1276" alt="WhatsApp Image 2026-06-10 at 10 44 27 PM (1)" src="https://github.com/user-attachments/assets/1dba0259-76ed-4854-a940-0afba4f86da9" />

**2. Verifikasi Akses Internet**
Seluruh klien, baik di HQ Jakarta maupun Branch Surabaya, berhasil melakukan resolusi dan akses ICMP ke jaringan eksternal (`8.8.8.8`).
<img width="1262" height="1294" alt="WhatsApp Image 2026-06-10 at 10 50 29 PM" src="https://github.com/user-attachments/assets/859d479f-2684-45e1-8d35-6d4fb04ba9b3" />
<img width="1266" height="1276" alt="WhatsApp Image 2026-06-10 at 10 44 27 PM (1)" src="https://github.com/user-attachments/assets/1dba0259-76ed-4854-a940-0afba4f86da9" />
<img width="1270" height="1284" alt="WhatsApp Image 2026-06-10 at 10 50 29 PM (1)" src="https://github.com/user-attachments/assets/cc969f34-0130-487d-bbbe-7ef6ef653e04" />



**3. Verifikasi Konektivitas Antar-Situs (Inter-Site Ping)**
Proses *routing* OSPF di atas terowongan GRE beroperasi dengan baik, terbukti dari keberhasilan koneksi *ping* lintas situs (misal: VLAN 10 Jakarta ke VLAN 40 Surabaya).
<img width="1206" height="1294" alt="WhatsApp Image 2026-06-10 at 10 44 29 PM (2)" src="https://github.com/user-attachments/assets/da7815c7-ff45-4079-bcc2-75dbc561709c" />
<img width="1265" height="1298" alt="WhatsApp Image 2026-06-10 at 10 44 30 PM" src="https://github.com/user-attachments/assets/9db56ac8-5a6b-47c1-9d31-1fd8fe0392e8" />
**4. Verifikasi Akses Web Server Internal**
Klien dari *branch* Surabaya mampu mengakses layanan Nginx yang berada di VLAN 60 (Server HQ) Jakarta menggunakan IP `192.168.60.10`.
<img width="1274" height="1272" alt="WhatsApp Image 2026-06-10 at 10 50 30 PM" src="https://github.com/user-attachments/assets/20ffd0dc-4f8c-43f5-ba3d-c82cc444beb5" />

**5. Verifikasi Tabel Routing OSPF FortiGate**
Rute dari jaringan Surabaya termuat di FortiGate Jakarta, dan sebaliknya.
<img width="1283" height="1293" alt="WhatsApp Image 2026-06-10 at 10 50 30 PM (1)" src="https://github.com/user-attachments/assets/c8338b40-26a7-4356-9977-09af2dcefbbb" />
<img width="1287" height="1289" alt="WhatsApp Image 2026-06-10 at 10 50 30 PM (2)" src="https://github.com/user-attachments/assets/69ee8576-de0b-4f0d-9327-54137fe7bde0" />


---

## 5. Analisis dan Kesimpulan

### Analisis
1. **Redundansi *Gateway* (VRRP):** Konfigurasi VRRP pada Cisco dan MikroTik Jakarta berhasil memberikan *High Availability* di sisi LAN. Jika Cisco mengalami kegagalan, posisi *Master* akan diambil alih oleh MikroTik secara transparan tanpa klien perlu menyadari perubahan IP *gateway*.
2. **Sentralisasi Manajemen IP (DHCP Relay):** Dengan implementasi DHCP Relay (`ip helper-address`), distribusi IP VLAN 10 dan 20 di HQ dapat dipusatkan pengelolaannya pada satu Ubuntu Server di VLAN 60. Ini mempermudah administrasi *pool* skala besar dibanding mendesentralisasi DHCP di tiap sub-jaringan router.
3. **Konektivitas *Site-to-Site* Lintas Publik:** Penggunaan *Generic Routing Encapsulation* (GRE) sukses menembus infrastruktur ISP publik dengan menciptakan tautan virtual secara *point-to-point* antar situs. Implementasi protokol *routing* dinamis (OSPF) di atas *GRE tunnel* memastikan pertukaran informasi *subnet* (*network advertisement*) dari Jakarta ke Surabaya (dan sebaliknya) berlangsung adaptif tanpa harus mengelola puluhan *static route* secara manual.

### Kesimpulan
Praktikum Modul 5 mendemonstrasikan perancangan arsitektur jaringan tingkat lanjut (Enterprise) yang mengutamakan keandalan (melalui VRRP) dan skalabilitas (melalui OSPF dan infrastruktur terpusat). Konvergensi berbagai vendor perangkat (Cisco, MikroTik, Fortinet, dan perangkat berbasis Linux) berjalan harmonis karena penggunaan *standard protocol* IEEE maupun IETF. Pengamanan lalu lintas lintas-kota pun terisolasi secara hierarkis menggunakan *Tunneling* yang melewati topologi WAN ISP dengan sangat efisien.
