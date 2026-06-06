# Tugas Pendahuluan
1. buat konfigurasi pada server PNET Lab berdasarkan topologi berikut ini **menggunakan node mikrotik**:
![topologi](images/topology.jpg)

2. lakukan konfigurasi singkat menggunakan terminal pada server PNET Lab dengan urutan berikut:\
a. konfigurasi IP Address.\
b. konfigurasi DHCP Client agar terhubung dengan internet.\
c. konfigurasi DHCP Server router untuk PC A dan PC B.\
d. konfigurasi NAT agar PC A dan PC B bisa terhubung dengan internet.\
e. konfigurasi routing statis agar PC A dan PC B bisa saling terhubung (ping).\
*tidak boleh konfigurasi menggunakan GUI atau winbox.

4. lakukan test koneksi dengan ping antar pc dan koneksi internet dari masing masing pc.

lampirkan screenshot hasil topologi, konfigurasi, dan hasil test koneksi pada laporan.

**JANGAN LUPA MEMATIKAN LAB SERVER PNET SETELAH KALIAN SELESAI!**

bagi praktikan yang lupa mematikan lab server, **akan mendapatkan pengurangan poin.**

# Modul Firewall & NAT

## 1. Pengenalan Firewall
### 1.1 Apa itu Firewall?
"Coba bayangin firewall itu kayak satpam digital buat jaringan komputer kamu. Dia yang berdiri di gerbang jaringan buat ngecek siapa yang boleh masuk atau keluar."

Jadi, kalau ada data yang mau masuk atau keluar, firewall bakal lihat dulu aturannya: boleh lewat, ditolak sambil ngasih pesan error, atau langsung diabaikan kayak nggak pernah ada. Intinya, firewall ini bantu jagain komputer dari hal-hal yang nggak diinginkan kayak hacker atau virus.

Sebelum ada firewall, keamanan jaringan cuma pakai Access Control List (ACL), tapi ACL ini nggak bisa bedain isi dari data yang lewat. Jadinya, masih banyak celah yang bisa dimanfaatin sama orang jahat. Apalagi sekarang internet udah kayak kebutuhan pokok organisasi. Sayangnya, koneksi ke internet juga buka celah buat serangan dari luar. Nah, firewall hadir buat nutup celah itu dan jagain jaringan internal biar tetap aman.

### 1.2 Jenis-Jenis Firewall
1. **Packet Filtering**
![Packet Filtering](images/1.1.1_PacketFiltering.png)
Cek satu per satu data yang lewat berdasarkan IP, port, dan protokol. Tapi dia nggak tahu ini bagian dari komunikasi yang mana, jadi agak kaku.

2. **Stateful Inspection**
![Stateful Inspection](images/1.1.1_StatefulInspection.webp)
Lebih canggih, bisa tahu ini data udah bagian dari koneksi yang sah atau belum.

3. **Application Layer Firewall**
Bisa ngintip sampai ke isi aplikasi (kayak HTTP, FTP), bahkan bisa blokir konten tertentu. Biasanya pakai proxy.

4. **Next Generation Firewall (NGFW)**
![NGFW](images/1.1.1_NGFW.webp)
Ini firewall zaman now! Bisa cek isi data lebih dalam (deep packet inspection), termasuk enkripsi SSL.

5. **Circuit Level Gateway**
Kerja di level koneksi (session). Cuma lihat apakah koneksi udah sah atau belum, tapi nggak cek isi datanya. Jadi bisa lolos tuh malware.

6. **Software Firewall**
![Software Firewal](images/1.1.1_SoftwareFirewall.webp)
Dipasang di komputer atau server. Fleksibel tapi kadang berat dan makan waktu buat setting-nya.

7. **Hardware Firewall**
![Hardware Firewall](images/1.1.1_HardwareFirewall.webp)
Bentuknya kayak perangkat fisik. Dipasang di antara internet dan jaringan internal, jadi bisa tahan serangan sebelum masuk lebih jauh.

8. **Cloud Firewall**
Firewall yang dijalankan di cloud. Cocok buat organisasi yang udah banyak pakai layanan cloud.

### 1.3 Cara Kerja Firewall
Firewall punya semacam buku aturan. Tiap data yang mau masuk atau keluar dicek dulu, sesuai atau nggak sama aturan itu. Misalnya, ada aturan yang bilang pegawai HRD nggak boleh akses server programmer—nah firewall bakal blokir tuh akses. Aturan bisa beda-beda tergantung kebutuhan dan kebijakan tiap organisasi.

**Kebijakan Akses di Firewall**
| Kebijakan | Penjelasan |
| --------- | ---------- |
| **Accept** | Dia yang memberikan izin lalu lintas data untuk bisa lewat. |
| **Reject** | Memblokir lalu lintas tapi memberikan balasan berupa *unreachable error* atau error yang tidak dapat dijangkau. |
| **Drop** | Memblokir lalu lintas tanpa memberikan balasan sama sekali. |


## 2. Network Address Translation (NAT)
### 2.1 Apa itu NAT?
![NAT](images/1.2_NAT.jpg)

Pernah bingung kenapa semua orang bisa internetan padahal IP publik di dunia ini terbatas? Nah, di sinilah NAT jadi penyelamat. 

"NAT itu semacam trik pintar yang bikin banyak perangkat di rumah atau kantor kamu bisa akses internet pakai satu IP publik aja."

Jadi meskipun cuma punya satu “alamat rumah” di dunia maya, banyak “penghuni” tetap bisa kirim dan terima data. Coba bayangin: alamat IPv4 yang tersedia cuma sekitar 4,3 miliar. Padahal perangkat yang nyambung ke internet udah lebih dari itu. Kalau tiap perangkat butuh satu IP publik, alamat bakal cepat habis. Nah, dengan NAT, cukup satu IP publik buat satu jaringan lokal, terus semua perangkat di jaringan itu bisa internetan bareng lewat IP publik yang sama.

### 2.2 Jenis-Jenis NAT
1. **Static NAT**
Satu IP lokal dihubungkan ke satu IP publik (one-to-one). Jarang dipakai karena mahal dan boros IP publik. Cocok buat server yang butuh alamat tetap, misalnya buat hosting website.

2. **Dynamic NAT**
IP lokal diubah ke IP publik dari kumpulan (pool) IP yang tersedia. Kalau IP di pool habis, permintaan koneksi ditolak. Tetap butuh banyak IP publik.

3. **Port Address Translation (PAT)**
Ini yang paling sering dipakai. Banyak IP lokal bisa pakai satu IP publik dengan membedakan tiap koneksi berdasarkan port. Hemat dan efisien!

### 2.3 Cara Kerja NAT
Biasanya, NAT ini ada di router yang jadi penghubung antara jaringan lokal dan internet. Kalau ada perangkat di dalam jaringan lokal kirim data ke internet, alamat IP-nya bakal diubah jadi alamat IP publik dulu sama router. Pas data dari internet mau balik ke perangkat tadi, NAT akan ganti lagi alamat publik itu jadi IP lokal si pengirim. Semuanya dicatat rapi di "tabel NAT" biar nggak bingung.

Bayangin dua orang dari satu rumah (misalnya laptop A dan B) buka website yang sama di waktu yang sama, pakai port yang sama. Kalau cuma IP yang diubah, pas server balikin datanya, router bingung: data ini buat A atau B? Makanya, NAT juga ngubah nomor port, jadi bisa bedain mana data buat siapa.

### 2.4 Istilah Penting di NAT
![Istilah NAT](images/1.2.3_IstilahNAT.png)

| Istilah | Penjelasan |
| ------- | ---------- |
| **Inside Local Address** | IP lokal perangkat di jaringan dalam (biasanya IP privat kayak 192.168.x.x) |
| **Inside Global Address** | IP publik yang mewakili perangkat dari dalam jaringan ke dunia luar |
| **Outside Local Address** | IP tujuan dari sisi luar yang udah diterjemahin di dalam jaringan |
| **Outside Global Address** | IP asli dari tujuan di luar jaringan |

## 3. Connection Tracking
### 3.1 Apa itu Connection Tracking?
**Connection Tracking** (pelacakan koneksi) adalah fitur **"pengamat lalu lintas jaringan"** yang cerdas. Ia mencatat siapa yang sedang ngobrol dengan siapa, kapan mulai ngobrol, lewat jalur mana (IP & port), dan apakah obrolan itu masih aktif atau sudah selesai.

> Bayangkan kamu punya resepsionis jaringan yang mencatat semua pengunjung (paket data) yang masuk dan keluar. Kalau ada pengunjung yang balik lagi (paket balasan), resepsionis akan mengenalinya dan langsung mengizinkannya masuk, tanpa perlu tanya-tanya lagi.

Connection Tracking ini melakukan **manajemen trafik** dengan cara menyimpan informasi penting dari koneksi tersebut seperti:
- Source Address
- Destination Address
- Source Port
- Destination Port
- Protocol
- Connection State

Label ini sangat penting terutama untuk proses **firewall filtering** dan **NAT** karena memungkinkan router **mengenali status dari setiap paket data** yang lewat.

### 3.2 Cara Kerja Connection Tracking
Saat kamu **mengakses website**:
1. Komputer kamu mengirim permintaan ke server (misalnya IP `8.8.8.8`).
2. Connection tracking mencatat: "Oke, koneksi baru dari `192.168.1.10` ke `8.8.8.8:80`".
3. Server membalas → sistem tahu ini **balasan sah** dan langsung diizinkan.
4. Kalau ada koneksi aneh dari luar yang belum pernah dicatat → **langsung ditolak** (state: `invalid`).

### 3.3 Manfaat Connection Tracking
Connection Tracking memberikan banyak keuntungan dalam pengelolaan dan pengamanan jaringan. Berikut beberapa manfaat utamanya:
1. Keamanan yang Lebih Baik (Stateful Firewall)
2. Mendukung NAT Secara Efisien
3. Mengurangi Beban Router
4. Kontrol Lebih Detail terhadap Lalu Lintas Jaringan
5. Mendeteksi dan Menghentikan Koneksi Tidak Sah

# TAHAPAN PRAKTIKUM

## Praktikum 1: Firewall NAT Dasar

**Langkah 0: buat topologi seperti berikut:**
![topologi](images/baru/topologi.png)

**Langkah 1: Reset Konfigurasi Router**
Pastikan router telah di-reset ke kondisi awal (tanpa konfigurasi) agar tidak terjadi konflik pada konfigurasi berikutnya.
1. Gunakan aplikasi **Winbox** untuk mengakses router.
2. Masuk ke menu **System** -> **Reset Configuration**.
3. Ceklis pada kotak **No Default Configuration**.
4. Klik tombol **Reset Configuration**.

![Reset Router](images/baru/1.png)

**Langkah 2: Konfigurasi DHCP Client pada Router A (ether1)**
Sambungkan port **ether1** pada Router A ke jaringan Lab/Internet, lalu atur DHCP Client agar router mendapat IP otomatis.
1. Masuk ke menu **IP** -> **DHCP Client**.
2. Klik tombol **+** (Add) untuk menambahkan entri baru.
3. Pilih **ether1** pada kolom **Interface**.
4. Klik **Apply** lalu **OK**.
5. Pastikan status koneksi berubah menjadi **bound**.

![DHCP Client Router A](images/baru/2.png)

**Langkah 3: Konfigurasi IP Address di Router A (ether2)**
Tambahkan IP Address pada interface **ether2** Router A sebagai jalur penghubung ke Router B.
1. Masuk ke menu **IP** -> **Addresses**.
2. Klik tombol **+** (Add).
3. Masukkan **Address**: `10.10.10.1/30`.
4. Pilih **Interface**: `ether2`.
5. Klik **Apply** lalu **OK**.

![IP Address Router A](images/baru/3.png)

**Langkah 4: Konfigurasi Routing Statis di Router A**
Atur routing statis agar Router A tahu rute menuju jaringan lokal PC Client yang ada di belakang Router B.
1. Masuk ke menu **IP** -> **Routes**.
2. Klik tombol **+** (Add).
3. Isi **Dst. Address**: `192.168.10.0/24`.
4. Isi **Gateway**: `10.10.10.2`.
5. Klik **Apply** lalu **OK**.

![Routing Router A](images/baru/4.png)

**Langkah 5: Konfigurasi IP Address di Router B (ether2)**
Pada Router B, tambahkan IP Address di **ether2** yang mengarah ke PC Client.
1. Buka Winbox untuk Router B.
2. Masuk ke menu **IP** -> **Addresses**.
3. Klik tombol **+** (Add).
4. Masukkan **Address**: `192.168.10.1/24`.
5. Pilih **Interface**: `ether2`.
6. Klik **Apply** lalu **OK**.
7. Abaikan ip pada ether3.
![IP ether2 Router B](images/baru/5.png)

**Langkah 6: Konfigurasi IP Address di Router B (ether1)**
Tambahkan juga IP Address di **ether1** Router B yang mengarah ke Router A.
1. Masih di menu **IP** -> **Addresses**, klik tombol **+** (Add).
2. Masukkan **Address**: `10.10.10.2/30`.
3. Pilih **Interface**: `ether1`.
4. Klik **Apply** lalu **OK**.

![IP ether1 Router B](images/baru/6.png)

**Langkah 7: Konfigurasi Default Route di Router B**
Atur *default route* agar semua paket dari jaringan lokal Router B dikirim ke Router A (menuju internet).
1. Masuk ke menu **IP** -> **Routes**.
2. Klik tombol **+** (Add).
3. Biarkan **Dst. Address** tetap `0.0.0.0/0`.
4. Isi **Gateway**: `10.10.10.1`.
5. Klik **Apply** lalu **OK**.

![Default Route Router B](images/baru/7.png)

**Langkah 8: Konfigurasi Firewall NAT di Router B**
Bikin aturan NAT (Network Address Translation) di Router B agar jaringan lokal bisa ikut internetan pakai IP publik router.
1. Kembali ke Winbox Router B , masuk ke menu **IP** -> **Firewall** -> tab **NAT**.
2. Klik tombol **+** (Add) untuk membuat aturan baru.
3. Pada tab **General**, ubah **Chain** menjadi `srcnat`.
4. Atur **Out. Interface** menjadi `ether1`.

![NAT General](images/baru/8.png)

5. Geser ke tab **Action**, ubah **Action** menjadi `masquerade`.
6. Klik **Apply** lalu **OK**.

![NAT Action](images/baru/9.png)

7. Pastikan aturan NAT sudah masuk ke dalam daftar.

![NAT Verify](images/baru/10_nat%20verify.png)

**Langkah 9: Konfigurasi IP Statis di PC Client**
Atur IP statis pada laptop/PC Client agar bisa terhubung ke Router B.
1. Buka pengaturan Network (di Windows: **Control Panel -> Network and Sharing Center -> Change adapter settings**).
2. Klik kanan pada Ethernet, pilih **Properties**.
3. Buka **Internet Protocol Version 4 (TCP/IPv4)**.
4. Pilih **Use the following IP address** dan isi:
   - **IP address**: `192.168.10.2`
   - **Subnet mask**: `255.255.255.0`
   - **Default gateway**: `192.168.10.1`
5. Isi DNS server (misal: `8.8.8.8`).
6. Klik **OK**.

![IP PC Client](images/baru/11_konfigurasi%20ip%20address%20client.png)

**Langkah 10: Pengujian Jaringan dari PC Client**
Coba tes ping dari Command Prompt (CMD) di PC Client untuk memastikan routing dan NAT sudah berjalan sukses.
1. Buka CMD.
2. Ping ke Gateway Router B: `ping 192.168.10.1`.
3. Ping ke Gateway Router A: `ping 10.10.10.1`.
4. Ping ke IP internet/Lab (contoh IP DHCP Router A): `ping 10.4.89.134`.
5. Pastikan semuanya mendapatkan *Reply*!

![Pengujian Ping](images/baru/12_pengujian%20percobaan%201.png)
## Praktikum 2: Firewall Filter (Input vs Forward)
*(Praktikum ini adalah lanjutan dari Praktikum 1. Pastikan konfigurasi sebelumnya sudah berjalan dengan baik)*

**Langkah 1: Konfigurasi Firewall Filter (Chain Input)**
Kita akan mencoba memblokir trafik *ping* dari Router A yang ditujukan ke Router B.
1. Buka Winbox **Router B**.
2. Masuk ke menu **IP** -> **Firewall** -> tab **Filter Rules**.
3. Klik tombol **+** (Add) untuk membuat aturan baru.
4. Pada tab **General**, atur:
   - **Chain**: `input`
   - **Src. Address**: `10.10.10.1` (IP Router A)
   - **Protocol**: `icmp`

   ![Filter Input General](images/baru/percobaan%202/1_input.png)

5. Geser ke tab **Action**, atur **Action**: `drop`.

   ![Filter Input Action](images/baru/percobaan%202/2_input.png)

6. Klik **Apply** lalu **OK**. Pastikan aturan sudah ditambahkan ke dalam daftar.

   ![Verify Input](images/baru/percobaan%202/3_verify_filter_input.png)

**Langkah 2: Pengujian Chain Input dari Router A**
1. Buka **New Terminal** di Winbox **Router A**.
2. Lakukan ping ke IP Router B:
   ```bash
   ping 10.10.10.2
   ```
   *Hasilnya harus Timeout, karena paket masuk yang ditujukan langsung ke Router B diblokir oleh chain input.*
3. Lakukan ping ke IP PC Client yang berada di belakang Router B:
   ```bash
   ping 192.168.10.2
   ```
   *Hasilnya harus Reply, karena paket ini hanya numpang lewat Router B (chain forward).*

   ![Hasil Ping Input](images/baru/percobaan%202/4_tes%20ping%20input.png)

**Langkah 3: Konfigurasi Firewall Filter (Chain Forward)**
Sekarang kita biarkan Router A nge-ping Router B, tapi memblokir Router A jika nge-ping PC Client.
1. Di Winbox **Router B**, matikan (disable) aturan `input` sebelumnya. Klik aturan tersebut lalu tekan tombol **Silang (X)**.
2. Klik tombol **+** (Add) untuk membuat aturan baru.
3. Pada tab **General**, atur:
   - **Chain**: `forward`
   - **Src. Address**: `10.10.10.1`
   - **Protocol**: `icmp`

   ![Filter Forward General](images/baru/percobaan%202/1_forward.png)

4. Geser ke tab **Action**, atur **Action**: `drop`.

   ![Filter Forward Action](images/baru/percobaan%202/2_forward.png)

5. Klik **Apply** lalu **OK**. Pastikan aturan forward aktif dan aturan input lama berstatus silang abu-abu (disable).

   ![Verify Forward](images/baru/percobaan%202/3_verify%20filter%20forward.png)

**Langkah 4: Pengujian Chain Forward dari Router A**
1. Kembali ke **New Terminal** di **Router A**.
2. Lakukan ping ke IP Router B:
   ```bash
   ping 10.10.10.2
   ```
   *Hasilnya harus Reply, karena aturan input sudah dimatikan.*
3. Lakukan ping ke IP PC Client:
   ```bash
   ping 192.168.10.2
   ```
   *Hasilnya harus Timeout, karena paket yang melewati Router B menuju PC Client diblokir oleh chain forward.*

   ![Hasil Ping Forward](images/baru/percobaan%202/4_testing%20result%20forward.png)
## Praktikum 3: Firewall NAT Forwarding (Port Forwarding)
*(Praktikum ini menggunakan topologi dasar Praktikum 1. Sebelum mulai, pastikan kamu menghapus/disable semua aturan Firewall Filter Rules dari Praktikum 2 agar trafik tidak terblokir)*

Pada praktikum ini, kita akan membuat PC Client bertindak sebagai Web Server. Karena PC Client berada di jaringan lokal (privat), komputer dari luar (seperti Router A) tidak bisa mengaksesnya secara langsung. Kita butuh **Destination NAT (DstNAT)** di Router B untuk membelokkan trafik yang masuk.

**Langkah 1: Konfigurasi Port Forwarding (DstNAT) di Router B**
Kita akan mengatur agar siapa pun yang mengakses IP Router B (`10.10.10.2`) melalui port `8080` akan diarahkan ke Web Server PC Client (`192.168.10.2`) port `80`.
1. Buka Winbox **Router B**.
2. Masuk ke menu **IP** -> **Firewall** -> tab **NAT**.
3. Klik tombol **+** (Add) untuk menambahkan aturan baru.
4. Pada tab **General**, atur:
   - **Chain**: `dstnat`
   - **Dst. Address**: `10.10.10.2` (IP Router B yang mengarah ke Router A)
   - **Protocol**: `6 (tcp)`
   - **Dst. Port**: `8080`

   ![DstNAT General](images/baru/percobaan%203/dstnat1.png)

5. Geser ke tab **Action**, atur:
   - **Action**: `dst-nat`
   - **To Addresses**: `192.168.10.2` (IP PC Client)
   - **To Ports**: `80` (Port web server lokal)

   ![DstNAT Action](images/baru/percobaan%203/dstnat2.png)

6. Klik **Apply** lalu **OK**. Pastikan aturan tersebut sudah muncul di daftar NAT (berdampingan dengan aturan *masquerade* sebelumnya).

   ![Verify DstNAT](images/baru/percobaan%203/3_verify%20dst-nat.png)

**Langkah 2: Menjalankan Web Server di PC Client**
Setelah aturannya siap, sekarang kita nyalakan web server-nya. Kita akan membuat web server sementara menggunakan Python.
1. Buka **Command Prompt (CMD)** di PC Client.
2. Ketikkan perintah berikut lalu tekan Enter:
   ```bash
   python -m http.server 80
   ```
3. Biarkan jendela CMD tetap terbuka agar server tetap menyala.

   ![Web Server PC Client](images/baru/percobaan%203/5_server%20http.png)

**Langkah 3: Pengujian Port Forwarding dari Router A**
Sekarang kita posisikan Router A sebagai pihak dari luar yang ingin mengakses Web Server di PC Client.
1. Buka **New Terminal** di Winbox **Router A**.
2. Ketikkan perintah *fetch* untuk mengunduh halaman dari web server melalui IP Router B (port `8080`):
   ```bash
   /tool fetch url="http://10.10.10.2:8080" keep-result=no
   ```
3. Jika berhasil, statusnya akan menunjukkan proses *connecting* dan mengunduh file tanpa error. Artinya, trafik ke port 8080 Router B berhasil dibelokkan (NAT Forwarding) ke PC Client!

   ![Fetch dari Router A](images/baru/percobaan%203/4_tool%20fetch.png)

**Langkah 4: Mengamati Connection Tracking**
Kamu juga bisa melihat bagaimana Router B melacak perubahan port dan alamat IP ini.
1. Di Winbox **Router B**, masih di menu **IP** -> **Firewall**, buka tab **Connections**.
2. Perhatikan daftar yang muncul saat Router A mencoba mengakses web server. Kamu akan melihat catatan koneksi dari IP `10.10.10.1` ke `10.10.10.2:8080` sedang berstatus *established/syn received*.
3. perhatikan pada terminal router A, akan mendapatkan response dari web server PC Client. lalu cek ulang pada firewall filter connection router B.
4. dokumentasikan hasil terminal router A dan firewall filter connection router B.  


   ![Connection Tracking DstNAT](images/baru/percobaan%203/5_connections%20dst-nat.png)

---

# Tugas Modul

# 1. Topologi Jaringan
![Topologi Modul 4](images/Topologi4baru.png)

# 2. Penjelasan Perangkat

| No. | Perangkat | Fungsi |
|:---:|-----------|--------|
| 1 | Cloud / Jaringan Lab | Menghubungkan simulasi PNETLab ke jaringan lab atau internet |
| 2 | MikroTik ISP | Berperan sebagai router ISP atau simulasi jaringan luar |
| 3 | FortiGate | Berperan sebagai firewall utama yang mengatur akses antara WAN, LAN, dan DMZ |
| 4 | Cisco Router | Berperan sebagai router internal menuju jaringan LAN |
| 6 | Client LAN **Tinycore Linux** | Client internal yang berada di belakang Cisco Router |
| 7 | Client WAN **Tinycore Linux**| Client dari sisi luar/internet untuk menguji akses ke server DMZ |
| 8 | Ubuntu Server DMZ | Server web yang diletakkan di zona DMZ |

# 3. Segmentasi Jaringan

Pada topologi ini terdapat beberapa segment jaringan:

| Segment | Network | Keterangan |
|---------|---------|-----------|
| Jaringan Lab / Internet | DHCP dari jaringan lab | Sumber koneksi luar |
| ISP ke FortiGate | 10.10.10.0/30 | Link antara MikroTik ISP dan FortiGate |
| Client WAN | 172.16.100.0/24 | Jaringan client dari sisi luar |
| FortiGate ke Cisco | 10.20.20.0/30 | Link antara FortiGate dan Cisco Router |
| LAN | 192.168.10.0/24 | Jaringan internal client |
| DMZ | 192.168.20.0/24 | Jaringan server DMZ |

# 4. Tabel IP Address

| Perangkat | Interface | IP Address | Gateway | Keterangan |
|:----------|:----------|:-----------|:--------|:-----------|
| MikroTik ISP | ether1 | DHCP Client | DHCP dari jaringan lab | Terhubung ke Cloud / jaringan lab |
| MikroTik ISP | ether2 | `10.10.10.1/30` | - | Terhubung ke FortiGate port1 |
| MikroTik ISP | ether3 | `172.16.100.1/24` | - | Gateway untuk Client-WAN |
| FortiGate | port1 | `10.10.10.2/30` | `10.10.10.1` | Interface WAN |
| FortiGate | port2 | `10.20.20.1/30` | - | Interface INSIDE ke Cisco |
| FortiGate | port3 | `192.168.20.1/24` | - | Interface DMZ |
| Cisco Router | G0/0 | `10.20.20.2/30` | - | Terhubung ke FortiGate port2 |
| Cisco Router | G0/1 | `192.168.10.1/24` | - | Gateway LAN |
| Client LAN **Tinycore Linux**| eth0 | `192.168.10.10/24` | `192.168.10.1` | Client internal |
| Client WAN **Tinycore Linux**| eth0 | `172.16.100.10/24` | `172.16.100.1` | Client luar |
| Ubuntu Server DMZ | eth0 / ens3 | `192.168.20.10/24` | `192.168.20.1` | Web server DMZ |

# 5. Instruksi Pengerjaan


Dikerjakan secara **berkelompok**. Output wajib berupa file `README.md` di GitHub dalam folder `LA/`.

---

## 5.1 Persiapan

1. Buat workspace baru di PNETLab sesuai kelompok.
2. Buat topologi sesuai gambar di modul, pastikan semua node terhubung dan di-start.
3. Node berat (Ubuntu Server, FortiGate, Cisco Router) tunggu **3–5 menit** booting. Tekan `Enter` jika terminal belum merespons.

---

## 5.2 Login Perangkat

| Perangkat | Username | Password |
|-----------|----------|----------|
| FortiGate | `admin` | *(kosong)* |
| Ubuntu Server | `root` | `root` |
| MikroTik | default image | default image |
| Cisco Router | — | — |
| Tiny Core Linux | default image | default image |

---

## 5.3 Konfigurasi MikroTik ISP

| Interface | IP |
|-----------|----|
| ke Cloud/lab | DHCP Client |
| ke FortiGate | `10.10.10.1/30` |
| ke Client WAN | `172.16.100.1/24` |

Yang harus dikonfigurasi: DHCP Client, IP tiap interface, NAT masquerade, route ke LAN (`192.168.10.0/24`) dan DMZ (`192.168.20.0/24`) via FortiGate.

**Target:** MikroTik bisa ping ke FortiGate `10.10.10.2`.

---

## 5.4 Konfigurasi FortiGate

| Interface | Peran | IP |
|-----------|-------|----|
| `port1` | WAN | `10.10.10.2/30` |
| `port2` | INSIDE | `10.20.20.1/30` |
| `port3` | DMZ | `192.168.20.1/24` |

Yang harus dikonfigurasi:
- Default route → `10.10.10.1`
- Static route → LAN `192.168.10.0/24` via `10.20.20.2`
- Address object: LAN, Server DMZ, Client WAN
- Policy: `LAN_to_WAN` (NAT on), `LAN_to_DMZ` (NAT off), `WAN_to_DMZ_HTTP`
- VIP / port forwarding: WAN `10.10.10.2` → Server DMZ `192.168.20.10`

**Target:** Client LAN bisa akses internet & DMZ. Client WAN bisa akses web DMZ via `http://10.10.10.2`.

---

## 5.5 Konfigurasi Cisco Router

| Interface | IP |
|-----------|----|
| ke FortiGate | `10.20.20.2/30` |
| ke LAN | `192.168.10.1/24` |

Yang harus dikonfigurasi: IP tiap interface, `no shutdown`, default route → `10.20.20.1`.

**Target:** Bisa ping ke FortiGate `10.20.20.1` dan Server DMZ `192.168.20.10`.

---

## 5.6 Konfigurasi Client LAN *(Tiny Core Linux)*

| Parameter | Nilai |
|-----------|-------|
| IP | `192.168.10.10/24` |
| Gateway | `192.168.10.1` |
| DNS | `8.8.8.8` |

**Target:** Ping ke router, FortiGate, dan DMZ. Akses `http://192.168.20.10`.

---

## 5.7 Konfigurasi Client WAN *(Tiny Core Linux)*

| Parameter | Nilai |
|-----------|-------|
| IP | `172.16.100.10/24` |
| Gateway | `172.16.100.1` |
| DNS | `8.8.8.8` |

**Target:** Akses `http://10.10.10.2`. Tidak bisa ping ke LAN `192.168.10.10` atau IP asli DMZ `192.168.20.10`.

---

## 5.8 Konfigurasi Ubuntu Server DMZ

| Parameter | Nilai |
|-----------|-------|
| IP | `192.168.20.10/24` |
| Gateway | `192.168.20.1` |
| DNS | `8.8.8.8` |

Yang harus dikonfigurasi: IP statis, install **Nginx**, ubah halaman default dengan format:

```
Tumod_4_DMZ_Firewall_No.kel-nama
```

Contoh: `Tumod_4_DMZ_Firewall_03-Kelompok3`

**Target:** Nginx aktif, halaman dapat diakses dari Client LAN dan Client WAN.

---


## 5.9 Penyimpanan Konfigurasi

- Cisco Router: `copy run start`
- MikroTik: export konfigurasi
- FortiGate: backup via CLI atau GUI
- Ubuntu Server: pastikan IP statis tersimpan dan Nginx aktif otomatis (`systemctl enable nginx`)
- Save workspace PNETLab

---

## 5.10 Dokumentasi

Simpan laporan di `LA/README.md` dengan isi:

- Topologi jaringan
- Tabel IP address
- Konfigurasi tiap perangkat *(+ screenshot)*
- Hasil pengujian *(+ screenshot)*
- Analisis dan kesimpulan

> Screenshot wajib menampilkan nama perangkat atau command yang digunakan.

# 6. Hasil yang diharapkan 
### 1. Pengujian client lan ke gateway cisco
![lan to cisco](images/lan_to_cisco.png)

### 2. Pengujian client lan ke fortigate
![lan to fortigate ](images/lan_to_fortigate.png)

### 3. Pengujian client lan ke DMZ
![lan to dmz](images/lan_to_dmz.png)

### 4. Pengujian client  lan akses ip dmz
![lan akses web dmz](images/lan_access_dmz.png)

### 5. Pengujian client wan ping ke isp mikrotik 
![client wan ping isp mikrotik](images/client_wan_to_mikrotik.png)

### 6. Penujian client wan ping ke fortigate
![client wan to fortigate](images/client_wan_to_fortigate.png)

### 7. Pengujian client wan akses http://10.10.10.2 harus muncul halaman web server.
![client wan akses web dmz](images/client_wan_access_dmz.png)

### 8. Pengujian client wan ping client lan 
![wan ping lan](images/wan_ping_lan.png)

### 9. Pengujian client wan ping IP asli DMZ
![wan ping dmz](images/wan_ping_dmz.png)

### 10. Pengujian server dmz ping lan 
![dmz ping lan](images/dmz_ping_lan.png)




# 7. Troubleshooting
#### 1. Jika GUI Tinycore tidak muncul ganti console menjadi VNC, Ram 4096Mb, dan CPU 2.
#### 2. Jika terminal ubuntu server tidak muncul, ganti console menjadi telnet.
#### 3. Untuk set ip address pada tinycore, masuk ke Control Panel > Network > Konfigurasi IP Address yang sesuai.
#### 4. Supaya Router Mikrotik dapat IP Dhcp-Client dari lab, pada saat menghubungkan network dengan mikrotik pilih ether 1.
#### 5. Jika firewall masih tembus, cek konfigurasi pada Fortinet.
#### 


# 8. Command dan Konfigurasi Dasar pada Perangkat Cisco dan Fortinet

Dokumentasi command dasar untuk konfigurasi perangkat jaringan Cisco IOS/IOS-XE dan Fortinet FortiGate.

---

## 📌 Daftar Isi

- [Cisco IOS / IOS-XE](#cisco-ios--ios-xe)
  - [Ganti Hostname](#1-ganti-hostname)
  - [Set IP Address](#2-set-ip-address)
  - [Simpan Konfigurasi](#3-simpan-konfigurasi)
  - [Routing Statis](#4-routing-statis)
- [Fortinet FortiGate](#fortinet-fortigate)
  - [Ganti Hostname](#1-ganti-hostname-1)
  - [Set IP Address](#2-set-ip-address-1)
  - [Simpan Konfigurasi](#3-simpan-konfigurasi-1)
  - [Routing Statis](#4-routing-statis-1)
- [Tips Umum](#tips-umum)

---

## Cisco IOS / IOS-XE

> Berlaku untuk Router dan Switch Cisco yang menggunakan IOS atau IOS-XE.

### Urutan masuk ke mode konfigurasi

```
Router> enable
Router# configure terminal
Router(config)#
```

---

### 1. Ganti Hostname

```
Router(config)# hostname NamaBaru
```

**Contoh:**

```
Router(config)# hostname Core-SW-01
Core-SW-01(config)#
```

---

### 2. Set IP Address

```
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
Router(config-if)# exit
```

> **Catatan:**
> - Ganti `GigabitEthernet0/0` sesuai interface yang tersedia di perangkat.
> - Perintah `no shutdown` wajib dijalankan karena interface Cisco default dalam kondisi **shutdown (off)**.
> - Gunakan `show ip interface brief` untuk memverifikasi IP yang sudah di-set.

**Verifikasi:**

```
Router# show ip interface brief
```

---

### 3. Simpan Konfigurasi

```
Router# copy running-config startup-config
```

Atau menggunakan shortcut:

```
Router# write memory
```

Atau singkatnya:

```
Router# wr
```

> **Catatan:** Jika tidak disimpan, konfigurasi akan hilang setelah perangkat di-restart.

---

### 4. Routing Statis

Sintaks dasar:

```
Router(config)# ip route [network-tujuan] [subnet-mask] [next-hop / exit-interface]
```

**Contoh — via next-hop IP:**

```
Router(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
```

**Contoh — via exit interface:**

```
Router(config)# ip route 192.168.2.0 255.255.255.0 GigabitEthernet0/1
```

**Default route (gateway of last resort):**

```
Router(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.1
```

> **Catatan:**
> - `0.0.0.0 0.0.0.0` berarti "semua tujuan yang tidak dikenal" — digunakan sebagai default gateway.
> - Tambahkan parameter `permanent` di akhir agar route tidak hilang saat interface down.
> - Administrative Distance (AD) default static route Cisco adalah **1**.

**Verifikasi:**

```
Router# show ip route
Router# show ip route static
```

---

## Fortinet FortiGate

> Berlaku untuk Firewall/UTM FortiGate yang menggunakan FortiOS CLI.

### Struktur perintah FortiGate

Semua konfigurasi FortiGate berada dalam blok `config ... end`. Gunakan `show` untuk verifikasi setelah konfigurasi.

---

### 1. Ganti Hostname

```
FortiGate # config system global
FortiGate (global) # set hostname NamaBaru
FortiGate (global) # end
```

**Contoh:**

```
FortiGate # config system global
FortiGate (global) # set hostname FG-HQ-01
FG-HQ-01 (global) # end
```

---

### 2. Set IP Address

```
FortiGate # config system interface
FortiGate (interface) # edit port1
FortiGate (port1) # set ip 192.168.1.1 255.255.255.0
FortiGate (port1) # set allowaccess ping https ssh
FortiGate (port1) # next
FortiGate (interface) # end
```

> **Catatan:**
> - Ganti `port1` sesuai interface yang digunakan.
> - `allowaccess` menentukan protokol manajemen yang diizinkan masuk ke interface. Opsi umum: `ping`, `https`, `ssh`, `http`, `telnet`.
> - Gunakan `show system interface port1` untuk verifikasi.

**Verifikasi:**

```
FortiGate # show system interface port1
```

---

### 3. Simpan Konfigurasi

FortiGate **menyimpan konfigurasi secara otomatis** setiap kali perintah `end` atau `next` dijalankan. Namun, untuk backup manual ke flash/USB:

```
FortiGate # execute backup config flash backup.conf
```

Backup juga dapat dilakukan melalui GUI:

```
Dashboard → System → Backup
```

---

### 4. Routing Statis

```
FortiGate # config router static
FortiGate (static) # edit 1
FortiGate (1) # set dst 192.168.2.0 255.255.255.0
FortiGate (1) # set gateway 10.0.0.2
FortiGate (1) # set device port2
FortiGate (1) # next
FortiGate (static) # end
```

**Default route:**

```
FortiGate # config router static
FortiGate (static) # edit 1
FortiGate (1) # set dst 0.0.0.0 0.0.0.0
FortiGate (1) # set gateway 10.0.0.1
FortiGate (1) # set device port1
FortiGate (1) # next
FortiGate (static) # end
```

> **Catatan:**
> - `edit 1` adalah nomor urut route (sequence number). Gunakan angka berbeda untuk setiap route baru.
> - `set device` wajib diisi dengan nama interface yang menuju ke gateway.
> - `set dst 0.0.0.0 0.0.0.0` digunakan sebagai default route (gateway of last resort).
> - Tambahkan `set distance [1-255]` untuk mengatur administrative distance jika diperlukan.

**Verifikasi:**

```
FortiGate # get router info routing-table all
FortiGate # get router info routing-table static
```

---

## Tips Umum

| Hal | Cisco | FortiGate |
|-----|-------|-----------|
| Masuk mode config | `enable` → `conf t` | Langsung dari prompt |
| Keluar dari mode config | `end` atau `exit` | `end` atau `next` |
| Simpan config | `copy run start` atau `wr` | Otomatis tersimpan |
| Lihat konfigurasi aktif | `show running-config` | `show full-configuration` |
| Lihat IP interface | `show ip interface brief` | `get system interface physical` |
| Verifikasi hostname | `show version` | `get system status` |
| Lihat routing table | `show ip route` | `get router info routing-table all` |
| Hapus static route | `no ip route [dst] [mask] [gw]` | `config router static` → `delete [seq]` |

---

> **Referensi:**
> - [Cisco IOS Command Reference](https://www.cisco.com/c/en/us/support/ios-nx-os-software/ios-15/products-command-reference-list.html)
> - [FortiGate CLI Reference](https://docs.fortinet.com/document/fortigate/latest/cli-reference)



