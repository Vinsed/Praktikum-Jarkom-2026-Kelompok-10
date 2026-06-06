# LAPORAN AKHIR MODUL 4
Berikut adalah laporan akhir pada modul 4 Firewall and NET
# Topologi
Topologi jaringan pada praktikum ini mengimplementasikan segmentasi jaringan yang memisahkan antara jaringan internal (LAN), zona demiliterisasi (DMZ), dan jaringan eksternal (WAN/Internet). 

![Topologi Jaringan Kelompok](images/topologi_kelompok.png)
(Catatan: Ganti tautan gambar di atas dengan screenshot topologi jaringan dari workspace PNETLab kelompok Anda)

*Keterangan Segmentasi:*
* *Jaringan Lab / Internet:* Sumber koneksi luar melalui DHCP.
* *ISP ke FortiGate (10.10.10.0/30):* Tautan penghubung antara MikroTik ISP dan FortiGate.
* *Client WAN (172.16.100.0/24):* Jaringan klien dari sisi luar (publik).
* *FortiGate ke Cisco (10.20.20.0/30):* Tautan penghubung firewall ke router internal.
* *LAN (192.168.10.0/24):* Jaringan internal (privat).
* *DMZ (192.168.20.0/24):* Zona untuk penempatan server web (Ubuntu) agar dapat diakses dari luar dengan pengawasan firewall.

---

## 2. Tabel IP Address

Berikut adalah pemetaan IP Address untuk setiap antarmuka (interface) perangkat jaringan yang digunakan:

| Perangkat | Interface | IP Address | Gateway | Keterangan |
| :--- | :--- | :--- | :--- | :--- |
| *MikroTik ISP* | ether1 | DHCP Client | DHCP dari Lab | Terhubung ke Cloud / Lab |
| *MikroTik ISP* | ether2 | 10.10.10.1/30 | - | Terhubung ke FortiGate port1 |
| *MikroTik ISP* | ether3 | 172.16.100.1/24 | - | Gateway untuk Client-WAN |
| *FortiGate* | port1 | 10.10.10.2/30 | 10.10.10.1 | Interface WAN |
| *FortiGate* | port2 | 10.20.20.1/30 | - | Interface INSIDE ke Cisco |
| *FortiGate* | port3 | 192.168.20.1/24 | - | Interface DMZ |
| *Cisco Router* | G0/0 | 10.20.20.2/30 | - | Terhubung ke FortiGate port2 |
| *Cisco Router* | G0/1 | 192.168.10.1/24 | - | Gateway LAN |
| *Client LAN* | eth0 | 192.168.10.10/24 | 192.168.10.1 | Tinycore Linux (Internal) |
| *Client WAN* | eth0 | 172.16.100.10/24 | 172.16.100.1 | Tinycore Linux (Eksternal) |
| *Ubuntu DMZ* | eth0/ens3 | 192.168.20.10/24 | 192.168.20.1 | Web Server Nginx |

---

## 3. Konfigurasi Tiap Perangkat

### A. Konfigurasi MikroTik ISP
*1. Setup IP Address & DHCP Client*
bash
/ip dhcp-client add interface=ether1 disabled=no
/ip address add address=10.10.10.1/30 interface=ether2
/ip address add address=172.16.100.1/24 interface=ether3

*2. Setup NAT & Routing Statis*
bash
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade
/ip route add dst-address=192.168.10.0/24 gateway=10.10.10.2
/ip route add dst-address=192.168.20.0/24 gateway=10.10.10.2


![Konfigurasi MikroTik](images/konfigurasi_mikrotik.png)
(Screenshot: Konfigurasi IP, Route, dan NAT pada MikroTik)

### B. Konfigurasi FortiGate
*1. Setup IP Address Interface*
bash
config system interface
  edit port1
    set ip 10.10.10.2 255.255.255.252
    set allowaccess ping
  next
  edit port2
    set ip 10.20.20.1 255.255.255.252
    set allowaccess ping
  next
  edit port3
    set ip 192.168.20.1 255.255.255.0
    set allowaccess ping
  next
end

*2. Setup Routing Statis*
bash
config router static
  edit 1
    set dst 0.0.0.0 0.0.0.0
    set gateway 10.10.10.1
    set device port1
  next
  edit 2
    set dst 192.168.10.0 255.255.255.0
    set gateway 10.20.20.2
    set device port2
  next
end

*3. Setup Policy dan VIP (Port Forwarding)*
Konfigurasi ini disarankan menggunakan GUI FortiGate:
* *Virtual IP (VIP):* Buat VIP mapping IP Eksternal 10.10.10.2 ke IP Internal 192.168.20.10.
* *Policy LAN_to_WAN:* Source LAN, Destination WAN, Action ACCEPT, NAT ON.
* *Policy LAN_to_DMZ:* Source LAN, Destination DMZ, Action ACCEPT, NAT OFF.
* *Policy WAN_to_DMZ_HTTP:* Source WAN, Destination VIP_DMZ, Action ACCEPT, NAT OFF.

![Konfigurasi FortiGate](images/konfigurasi_fortigate.png)
(Screenshot: Konfigurasi antarmuka, tabel routing, dan Firewall Policy FortiGate)

### C. Konfigurasi Cisco Router
*1. Setup IP Address & Routing*
bash
enable
configure terminal
interface GigabitEthernet0/0
 ip address 10.20.20.2 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet0/1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 10.20.20.1
copy running-config startup-config


![Konfigurasi Cisco](images/konfigurasi_cisco.png)
(Screenshot: Hasil perintah show ip interface brief dan show ip route pada Cisco)

### D. Konfigurasi Ubuntu Server DMZ
*1. Setup IP dan Web Server*
Konfigurasi IP statis pada netplan Ubuntu (mengarah ke 192.168.20.10/24 dengan gateway 192.168.20.1). Selanjutnya, instal dan ubah halaman utama Nginx:
bash
sudo apt update && sudo apt install nginx -y
sudo nano /var/www/html/index.nginx-debian.html
# Ubah isi file menjadi: Tumod_4_DMZ_Firewall_[No.kel]-[nama]
sudo systemctl enable nginx
sudo systemctl restart nginx


![Konfigurasi Ubuntu](images/konfigurasi_ubuntu.png)
(Screenshot: Status service Nginx dan konfigurasi IP Ubuntu Server)

### E. Konfigurasi Client (Tinycore Linux)
Akses *Control Panel > Network* pada masing-masing VM Tinycore dan atur sesuai tabel IP.
* *LAN Client:* IP 192.168.10.10, Mask 255.255.255.0, Gateway 192.168.10.1, DNS 8.8.8.8
* *WAN Client:* IP 172.16.100.10, Mask 255.255.255.0, Gateway 172.16.100.1, DNS 8.8.8.8

![Konfigurasi Tinycore](images/konfigurasi_tinycore.png)
(Screenshot: Konfigurasi IP pada GUI Network Tinycore)

---

## 4. Hasil Pengujian

1. *Pengujian Client LAN ke Gateway Cisco* (Ping ke 192.168.10.1)
   ![Pengujian 1](images/pengujian_1_lan_cisco.png)

2. *Pengujian Client LAN ke FortiGate* (Ping ke 10.20.20.1)
   ![Pengujian 2](images/pengujian_2_lan_fortigate.png)

3. *Pengujian Client LAN ke DMZ* (Ping ke 192.168.20.10)
   ![Pengujian 3](images/pengujian_3_lan_dmz.png)

4. *Pengujian Client LAN Akses Web DMZ* (Akses browser ke http://192.168.20.10)
   ![Pengujian 4](images/pengujian_4_lan_webdmz.png)

5. *Pengujian Client WAN ke ISP MikroTik* (Ping ke 172.16.100.1)
   ![Pengujian 5](images/pengujian_5_wan_mikrotik.png)

6. *Pengujian Client WAN ke FortiGate* (Ping ke 10.10.10.2)
   ![Pengujian 6](images/pengujian_6_wan_fortigate.png)

7. *Pengujian Client WAN Akses HTTP VIP* (Akses browser ke http://10.10.10.2)
   ![Pengujian 7](images/pengujian_7_wan_webdmz.png)

8. *Pengujian Client WAN Ping Client LAN* (Ping ke 192.168.10.10)
   ![Pengujian 8](images/pengujian_8_wan_lan_drop.png)
   (Keterangan: Hasil Request Timed Out (RTO) / Drop sesuai konfigurasi keamanan)

9. *Pengujian Client WAN Ping IP Asli DMZ* (Ping ke 192.168.20.10)
   ![Pengujian 9](images/pengujian_9_wan_aslidmz_drop.png)
   (Keterangan: Hasil Request Timed Out (RTO) / Drop)

10. *Pengujian Server DMZ Ping LAN* (Ping ke 192.168.10.10)
    ![Pengujian 10](images/pengujian_10_dmz_lan.png)
    (Keterangan: Hasil sesuai dengan policy firewall yang diterapkan)

---

## 5. Analisis dan Kesimpulan

*Analisis:*
1. *Implementasi Routing:* Routing statis pada jaringan ini berjalan dua arah secara efektif. MikroTik mengetahui keberadaan segment LAN dan DMZ dengan menggunakan FortiGate sebagai next-hop, sementara Cisco dan FortiGate mengandalkan rute default (0.0.0.0/0) untuk meneruskan paket asing ke arah internet.
2. *Keamanan Melalui Segmentasi:* FortiGate berhasil mengisolasi jaringan menjadi tiga zona yang berbeda kredibilitasnya: Inside (LAN - sangat dipercaya), DMZ (Semipublik - tempat diletakkannya resource yang dapat diakses publik), dan Outside (WAN/Internet - tidak dipercaya).
3. *Fungsi NAT (Network Address Translation):*
   * *Source NAT (Masquerade):* Diterapkan pada policy LAN ke WAN dan MikroTik ke Cloud sehingga pengguna LAN memiliki akses menuju internet dengan meminjam IP publik antarmuka luar.
   * *Destination NAT (Port Forwarding / VIP):* Diterapkan pada FortiGate (Policy WAN to DMZ) yang memungkinkan pengguna eksternal (Client WAN) untuk mengakses server web di ruang privat (DMZ) melalui representasi IP publik (10.10.10.2). Saat IP asli DMZ (192.168.20.10) diping secara langsung oleh pihak luar, koneksi di-drop, membuktikan lapisan keamanan jaringan berfungsi menutupi topologi internal.
4. *Connection Tracking:* Berkat sistem pelacakan koneksi pada firewall stateful (FortiGate), sesi komunikasi yang diinisiasi dari dalam (LAN ke DMZ/WAN) secara otomatis diizinkan membalas ke sumber tanpa perlu mendaftarkan aturan balik secara manual.

*Kesimpulan:*
Praktikum Modul 4 membuktikan bahwa integrasi antara routing statis dan arsitektur firewall merupakan fundamental krusial untuk manajemen jaringan skala korporasi. Penerapan zona DMZ efektif mengamankan server yang melayani akses publik dari ancaman luar tanpa membahayakan data kritis yang tersimpan pada jaringan LAN. Penggunaan fitur kebijakan akses (Filter Rules) dan translasi alamat (NAT/VIP) memastikan koneksi berjalan efisien dan sesuai dengan otorisasi (least privilege).
