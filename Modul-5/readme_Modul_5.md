# Laporan Penyelesaian Tugas Modul 5: Implementasi Jaringan Enterprise HQ–Branch

## 1. Topologi Jaringan

Topologi ini mensimulasikan jaringan *enterprise* yang menghubungkan Kantor Pusat (HQ) di Jakarta dan Kantor Cabang (Branch) di Surabaya. Kedua lokasi dihubungkan melalui infrastruktur *Internet Service Provider* (ISP) menggunakan teknologi *Generic Routing Encapsulation* (GRE) Tunnel, yang dikombinasikan dengan *Open Shortest Path First* (OSPF) untuk pertukaran rute secara dinamis. Jaringan HQ juga mengimplementasikan redundansi *gateway* menggunakan *Virtual Router Redundancy Protocol* (VRRP) dan layanan *Dynamic Host Configuration Protocol* (DHCP) terpusat.

![Topologi Jaringan Enterprise HQ-Branch](images/tumod/topologtumod5.png)
*(Catatan: Sesuaikan gambar dengan hasil *screenshot* topologi akhir dari *workspace* PNETLab)*

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

![Screenshot show vlan brief Switch Jakarta](images/tumod/tumod1/1.png)
![Screenshot show interfaces trunk Switch Jakarta](images/tumod/tumod1/2.png)

### 3.2. Cisco Router Jakarta
Pembuatan *subinterface*, konfigurasi IP fisik, penetapan VRRP (sebagai Master untuk VLAN 10 dan 60), serta konfigurasi `ip helper-address` (DHCP Relay) menuju Ubuntu Server `192.168.60.10`. Diakhiri dengan pengaturan *default route* menuju FortiGate Jakarta (`10.10.100.1`).

![Screenshot show ip interface brief Cisco JKT](images/tumod/tumod2/1.png)
![Screenshot show vrrp brief Cisco JKT](images/tumod/tumod2/2.png)
*(Screenshot pendukung ping ke FortiGate: Masukkan di sini)*

### 3.3. MikroTik Router Jakarta
Pembuatan interface VLAN, konfigurasi IP fisik, penetapan VRRP (sebagai Master untuk VLAN 20), dan konfigurasi DHCP Relay. Penetapan *default route* diarahkan ke FortiGate Jakarta (`10.10.101.1`).

![Screenshot IP Address MikroTik JKT](images/placeholder_mikrotik_ip.png)
![Screenshot VRRP MikroTik JKT](images/placeholder_mikrotik_vrrp.png)
![Screenshot DHCP Relay MikroTik JKT](images/placeholder_mikrotik_dhcp_relay.png)
![Screenshot Routing MikroTik JKT](images/placeholder_mikrotik_route.png)
*(Catatan: Ganti placeholder gambar di atas dengan screenshot sesungguhnya dari terminal MikroTik)*

### 3.4. Ubuntu Server Jakarta
Konfigurasi *static IP* `192.168.60.10` via Netplan dengan gateway IP Virtual VRRP (`192.168.60.1`). Instalasi `isc-dhcp-server` dan konfigurasi *pool* untuk VLAN 10 dan 20. Selanjutnya dilakukan instalasi Nginx sebagai layanan web server internal.

![Screenshot IP & Route Ubuntu Server](images/placeholder_ubuntu_ip_route.png)
![Screenshot dhcpd.conf Ubuntu Server](images/placeholder_ubuntu_dhcp.png)

### 3.5. FortiGate Jakarta
Konfigurasi interface mengarah ke LAN (Cisco & MikroTik) dan WAN (ISP). Pembuatan *Firewall Policy* untuk *allow traffic* dari LAN menuju Internet dengan NAT aktif. Konfigurasi GRE Tunnel (`172.16.0.1`) menuju IP publik FortiGate Surabaya, dan menjalankan OSPF di atas GRE Tunnel dengan *Redistribute Static* diaktifkan.

![Screenshot Interface FortiGate JKT](images/placeholder_fg_jkt_interface.png)
![Screenshot Routing OSPF FortiGate JKT](images/placeholder_fg_jkt_ospf.png)

### 3.6. MikroTik ISP
Konfigurasi IP penghubung antar FortiGate dan DHCP Client mengarah ke Internet (Cloud PNETLab). Konfigurasi `NAT Masquerade` pada `ether1` (Internet).

![Screenshot IP MikroTik ISP](images/placeholder_isp_ip.png)
![Screenshot Routing MikroTik ISP](images/placeholder_isp_route.png)

### 3.7. Cisco Switch & MikroTik Surabaya
Pembuatan VLAN 30 dan 40 di Cisco Switch Surabaya (*trunk* mengarah ke MikroTik). Pada MikroTik Surabaya, dikonfigurasi IP Gateway, layanan DHCP Server lokal untuk VLAN 30, serta *default route* menuju FortiGate Surabaya (`10.10.200.1`).

![Screenshot VLAN Switch Surabaya](images/placeholder_sw_sby_vlan.png)
![Screenshot IP & DHCP MikroTik Surabaya](images/placeholder_mt_sby_ip_dhcp.png)

### 3.8. FortiGate Surabaya
Konfigurasi interface WAN mengarah ke MikroTik ISP dan interface LAN mengarah ke MikroTik Surabaya. Pembuatan kebijakan keamanan (*Firewall Policy*) dengan NAT untuk akses internet. Konfigurasi GRE Tunnel (`172.16.0.2`) menuju FortiGate Jakarta dan aktivasi *routing* OSPF *over* GRE.

![Screenshot Interface FortiGate SBY](images/placeholder_fg_sby_interface.png)
![Screenshot OSPF Neighbor FortiGate SBY](images/placeholder_fg_sby_ospf_neighbor.png)

---

## 4. Hasil Pengujian

Pengujian dilakukan dari *end-device* (VPCs dan Tinycore) secara menyeluruh untuk memvalidasi kelancaran seluruh protokol.

**1. Verifikasi Distribusi IP DHCP (Jakarta & Surabaya)**
Klien pada VLAN 10 (Jakarta) dan VLAN 30 (Surabaya) berhasil mendapatkan IP Address dinamis dari masing-masing DHCP Server (Ubuntu dan MikroTik lokal).
![Screenshot DHCP Klien Jakarta](images/placeholder_dhcp_client_jkt.png)
![Screenshot DHCP Klien Surabaya](images/placeholder_dhcp_client_sby.png)

**2. Verifikasi Akses Internet**
Seluruh klien, baik di HQ Jakarta maupun Branch Surabaya, berhasil melakukan resolusi dan akses ICMP ke jaringan eksternal (`8.8.8.8`).
![Screenshot Ping Internet Klien](images/placeholder_ping_internet.png)

**3. Verifikasi Konektivitas Antar-Situs (Inter-Site Ping)**
Proses *routing* OSPF di atas terowongan GRE beroperasi dengan baik, terbukti dari keberhasilan koneksi *ping* lintas situs (misal: VLAN 10 Jakarta ke VLAN 40 Surabaya).
![Screenshot Ping Antar Situs](images/placeholder_ping_intersite.png)

**4. Verifikasi Akses Web Server Internal**
Klien dari *branch* Surabaya mampu mengakses layanan Nginx yang berada di VLAN 60 (Server HQ) Jakarta menggunakan IP `192.168.60.10`.
![Screenshot Akses Web Server dari SBY](images/tumod/tumod2/accesswebfromsurabaya.png)

**5. Verifikasi Tabel Routing OSPF FortiGate**
Rute dari jaringan Surabaya termuat di FortiGate Jakarta, dan sebaliknya.
![Screenshot Routing Table OSPF](images/placeholder_ospf_routing_table.png)

---

## 5. Analisis dan Kesimpulan

### Analisis
1. **Redundansi *Gateway* (VRRP):** Konfigurasi VRRP pada Cisco dan MikroTik Jakarta berhasil memberikan *High Availability* di sisi LAN. Jika Cisco mengalami kegagalan, posisi *Master* akan diambil alih oleh MikroTik secara transparan tanpa klien perlu menyadari perubahan IP *gateway*.
2. **Sentralisasi Manajemen IP (DHCP Relay):** Dengan implementasi DHCP Relay (`ip helper-address`), distribusi IP VLAN 10 dan 20 di HQ dapat dipusatkan pengelolaannya pada satu Ubuntu Server di VLAN 60. Ini mempermudah administrasi *pool* skala besar dibanding mendesentralisasi DHCP di tiap sub-jaringan router.
3. **Konektivitas *Site-to-Site* Lintas Publik:** Penggunaan *Generic Routing Encapsulation* (GRE) sukses menembus infrastruktur ISP publik dengan menciptakan tautan virtual secara *point-to-point* antar situs. Implementasi protokol *routing* dinamis (OSPF) di atas *GRE tunnel* memastikan pertukaran informasi *subnet* (*network advertisement*) dari Jakarta ke Surabaya (dan sebaliknya) berlangsung adaptif tanpa harus mengelola puluhan *static route* secara manual.

### Kesimpulan
Praktikum Modul 5 mendemonstrasikan perancangan arsitektur jaringan tingkat lanjut (Enterprise) yang mengutamakan keandalan (melalui VRRP) dan skalabilitas (melalui OSPF dan infrastruktur terpusat). Konvergensi berbagai vendor perangkat (Cisco, MikroTik, Fortinet, dan perangkat berbasis Linux) berjalan harmonis karena penggunaan *standard protocol* IEEE maupun IETF. Pengamanan lalu lintas lintas-kota pun terisolasi secara hierarkis menggunakan *Tunneling* yang melewati topologi WAN ISP dengan sangat efisien.