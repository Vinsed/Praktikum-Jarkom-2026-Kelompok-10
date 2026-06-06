
# LAPORAN AKHIR MODUL 4
Berikut adalah laporan akhir pada modul 4 Firewall and NET
# Topologi
Topologi jaringan pada praktikum ini mengimplementasikan segmentasi jaringan yang memisahkan antara jaringan internal (LAN), zona demiliterisasi (DMZ), dan jaringan eksternal (WAN/Internet). 

src="https://github.com/user-attachments/assets/1e23d046-b054-454c-8e48-93e9bbc7c023" />

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

<img width="980" height="604" alt="Dokumentasi Miktrotik IP" src="https://github.com/user-attachments/assets/13c3f1cf-22dc-495f-80f4-0efda76866fb" />
<img width="983" height="597" alt="Dokumentasi Miktrotik NAT" src="https://github.com/user-attachments/assets/55143cfe-9eb4-48ea-9f12-6b74dde03497" />
<img width="976" height="604" alt="Dokumentasi Fortinet IP" src="https://github.com/user-attachments/assets/2f770821-db43-46ba-b1e9-80fb2dc7258b" />
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

<img width="976" height="604" alt="Dokumentasi Fortinet IP" src="https://github.com/user-attachments/assets/e37c78d1-80e3-4324-bda3-7f0b7ab07bef" />
<img width="977" height="598" alt="Dokumentasi Fortinet Routing" src="https://github.com/user-attachments/assets/913b0423-1b40-460c-89a1-8d39df125251" />
<img width="986" height="464" alt="Dokumentasi Fortinet Policy 1" src="https://github.com/user-attachments/assets/97a0d767-1d4b-4d09-8f00-00b87b7eb4d7" />
<img width="978" height="598" alt="Dokumentasi Fortinet Policy 2" src="https://github.com/user-attachments/assets/a32c328f-1066-4f2e-b876-6cbc8402411d" />

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

<img width="978" height="599" alt="Dokumentasi IP   Routing Cisco" src="https://github.com/user-attachments/assets/5f36ff33-37db-4271-bd2f-5f3b92ce43a0" />
### D. Konfigurasi Ubuntu Server DMZ
*1. Setup IP dan Web Server*
Konfigurasi IP statis pada netplan Ubuntu (mengarah ke 192.168.20.10/24 dengan gateway 192.168.20.1). Selanjutnya, instal dan ubah halaman utama Nginx:
bash
sudo apt update && sudo apt install nginx -y
sudo nano /var/www/html/index.nginx-debian.html
# Ubah isi file menjadi: Tumod_4_DMZ_Firewall_[No.kel]-[nama]
sudo systemctl enable nginx
sudo systemctl restart nginx

<img width="978" height="599" alt="Dokumentasi Ubuntu IP   nginx DMZ" src="https://github.com/user-attachments/assets/60431246-255e-48fc-b2e7-2311f3889203" />
---
## 4. Hasil Pengujian

1. *Pengujian Client LAN ke Gateway Cisco* (Ping ke 192.168.10.1)
   ![Pengujian 1]<img width="1401" height="776" alt="1" src="https://github.com/user-attachments/assets/fbb8ed57-1150-4c6f-a06f-00c31c2570f2" />


2. *Pengujian Client LAN ke FortiGate* (Ping ke 10.20.20.1)
   ![Pengujian 2]<img width="1600" height="908" alt="2" src="https://github.com/user-attachments/assets/4a775f62-a886-43e3-b82f-929f731fd622" />


3. *Pengujian Client LAN ke DMZ* (Ping ke 192.168.20.10)
   ![Pengujian 3]<img width="1600" height="909" alt="3" src="https://github.com/user-attachments/assets/8cdf5aba-73cb-451f-a0f5-27a9f10a6b11" />


4. *Pengujian Client LAN Akses Web DMZ* (Akses browser ke http://192.168.20.10)
   ![Pengujian 4]<img width="1600" height="911" alt="4" src="https://github.com/user-attachments/assets/649595cb-b975-4acc-82f9-582b97af3afc" />


5. *Pengujian Client WAN ke ISP MikroTik* (Ping ke 172.16.100.1)
   ![Pengujian 5]<img width="1600" height="908" alt="5" src="https://github.com/user-attachments/assets/4a1579f0-7d44-4795-b1e3-f9898c362ddd" />


6. *Pengujian Client WAN ke FortiGate* (Ping ke 10.10.10.2)
   ![Pengujian 6]<img width="1600" height="908" alt="6" src="https://github.com/user-attachments/assets/c7396b7c-06da-488a-8bf0-691da4b2fedb" />


7. *Pengujian Client WAN Akses HTTP VIP* (Akses browser ke http://10.10.10.2)
   ![Pengujian 7]<img width="1600" height="910" alt="7" src="https://github.com/user-attachments/assets/92194301-3504-4b75-9069-0d36a34baa8a" />


8. *Pengujian Client WAN Ping Client LAN* (Ping ke 192.168.10.10)
   ![Pengujian 8]<img width="1600" height="905" alt="8" src="https://github.com/user-attachments/assets/c990fdad-02a9-4c66-b1e2-a29c7a61f162" />

   (Keterangan: Hasil Request Timed Out (RTO) / Drop sesuai konfigurasi keamanan)

9. *Pengujian Client WAN Ping IP Asli DMZ* (Ping ke 192.168.20.10)
   ![Pengujian 9]<img width="1600" height="905" alt="9" src="https://github.com/user-attachments/assets/909c017a-7fb3-4665-8a90-8f6385023224" />

   (Keterangan: Hasil Request Timed Out (RTO) / Drop)

10. *Pengujian Server DMZ Ping LAN* (Ping ke 192.168.10.10)
    ![Pengujian 10]<img width="1600" height="908" alt="10" src="https://github.com/user-attachments/assets/6f1e0835-cff2-40da-9ed0-25bd0ca821d2" />

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
