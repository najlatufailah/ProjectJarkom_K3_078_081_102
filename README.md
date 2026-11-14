# ProjectJarkom_K3_078_081_108

| Nama                          | NRP        |
| ----------------------------- | ---------- |
| Najla Tufailah                | 5027251078 |
| Anargya Shafa Setiyadi P      | 5027251081 |
| Achmad Rifqy Aqila            | 5027251108 |

## 1. Topology
<img width="1916" height="896" alt="gambartopologyk3" src="https://github.com/user-attachments/assets/bac644ae-2d37-42e9-b3b4-a7057a956ffa" />
Cara membaca gambar:
 
| Tampilan di Packet Tracer | Arti |
|---|---|
| Garis hitam lurus | Kabel Copper Straight-Through (router ke switch, switch ke PC / server / access point) |
| Garis hitam putus-putus | Kabel Copper Cross-Over (R-ISP ke R-PUSAT dan R-ISP ke SRV-INTERNET) |
| Garis merah bergelombang | Kabel Serial DCE/DTE untuk link backbone antar-router (L1 sampai L6) |
| Segitiga hijau di ujung kabel | Link sudah aktif (up) |
| Tunnel GRE R-PUSAT ke R-PELABUHAN | Link virtual, tidak tampak sebagai kabel |
 
Perangkat pada topologi:
 
| Jenis | Model di Packet Tracer | Jumlah | Nama perangkat |
|---|---|:---:|---|
| Router | Cisco 2911 | 8 | R-ISP, R-PUSAT, R-DC, R-UTARA, R-SELATAN, R-TIMUR, R-BARAT, R-PELABUHAN |
| Switch | Cisco 2960-24TT | 14 | SW-SRV-BALAIKOTA, SW-PEGAWAI, SW-DC-CCTV, SW-DC-SRV, SW-UTARA-WIFI, SW-UTARA-SNS, SW-SEL-WIFI, SW-SEL-PJU, SW-TIMUR-WIFI, SW-TIMUR-SNS, SW-BARAT-WIFI, SW-BARAT-PKM, SW-PEL-SCADA, SW-PEL-KANTOR |
| PC | PC-PT | 8 | PC-PEGAWAI, PC-CCTV-KOTA, PC-SENSOR-LALIN, PC-SENSOR-BANJIR, PC-PJU, PC-PUSKESMAS, PC-SCADA, PC-KANTOR-PEL |
| Server | Server-PT | 3 | SRV-BALAIKOTA, SRV-DC, SRV-INTERNET |
| Access Point | AccessPoint-PT | 4 | AP-UTARA, AP-TIMUR, AP-SELATAN, AP-BARAT |
 
---
 
## 2. Skema Alamat: VLSM vs CIDR dan Alasan Pemilihan
 
**Skema yang dipakai di topologi: VLSM**, dengan blok induk `10.24.0.0/16`. Perhitungan CIDR (supernetting) tetap dibuat sebagai pembanding.
 
### 2.1 Parameter
 
| Parameter | Nilai | Keterangan |
|---|---|---|
| Nomor kelompok (K) | 3 | |
| Blok private | `10.24.0.0/16` | Oktet kedua = 8 × K = 24 |
| Link R-PUSAT ke R-ISP | `203.0.113.12/30` | Oktet keempat = 4 × K = 12; R-ISP `.14`, R-PUSAT `.13` |
| Server internet | `8.8.8.0/24` | Server `8.8.8.8`, gateway `8.8.8.1` di R-ISP |
| Tunnel GRE | `192.168.3.0/30` | R-PUSAT `.1`, R-PELABUHAN `.2` |
 
### 2.2 Cara menghitung
 
Untuk tiap subnet:
 
1. Kebutuhan alamat = host + 1 gateway + 2 (Network ID dan broadcast). Link antar-router tidak memakai gateway, sehingga hanya 2 host + 2 = 4 alamat.
2. Bit host = pembulatan ke atas dari log2(kebutuhan alamat).
3. Prefix = 32 − bit host; ukuran blok = 2^(bit host).
4. VLSM: subnet diurutkan dari yang terbesar, lalu tiap subnet menempati offset berikutnya (offset sebelumnya + ukuran blok). Hal ini membuat blok tidak saling tumpang tindih dan tidak ada celah.
5. Gateway memakai alamat usable pertama (`.1` dari network) dan dipasang di interface router.
Contoh, WiFi Publik Utara: 11.000 host + 1 gateway + 2 = 11.003 alamat, butuh 14 bit host (16.384), jadi prefix **/18**, Network ID `10.24.0.0`, broadcast `10.24.63.255`.
 
### 2.3 Tabel VLSM (skema yang dipakai)
 
| No | Subnet | Router | Host | Prefix | Network ID | Range host | Broadcast | Gateway |
|---:|---|---|---:|:---:|---|---|---|---|
| 1 | WiFi Publik Utara | R-UTARA | 11.000 | /18 | 10.24.0.0 | 10.24.0.1 – 10.24.63.254 | 10.24.63.255 | 10.24.0.1 |
| 2 | WiFi Publik Selatan | R-SELATAN | 6.500 | /19 | 10.24.64.0 | 10.24.64.1 – 10.24.95.254 | 10.24.95.255 | 10.24.64.1 |
| 3 | CCTV Kota | R-DC | 6.000 | /19 | 10.24.96.0 | 10.24.96.1 – 10.24.127.254 | 10.24.127.255 | 10.24.96.1 |
| 4 | WiFi Publik Timur | R-TIMUR | 5.500 | /19 | 10.24.128.0 | 10.24.128.1 – 10.24.159.254 | 10.24.159.255 | 10.24.128.1 |
| 5 | Sensor dan SCADA Pelabuhan | R-PELABUHAN | 5.000 | /19 | 10.24.160.0 | 10.24.160.1 – 10.24.191.254 | 10.24.191.255 | 10.24.160.1 |
| 6 | WiFi Publik Barat | R-BARAT | 4.500 | /19 | 10.24.192.0 | 10.24.192.1 – 10.24.223.254 | 10.24.223.255 | 10.24.192.1 |
| 7 | Pegawai Balai Kota | R-PUSAT | 1.800 | /21 | 10.24.224.0 | 10.24.224.1 – 10.24.231.254 | 10.24.231.255 | 10.24.224.1 |
| 8 | Sensor Banjir Timur | R-TIMUR | 700 | /22 | 10.24.232.0 | 10.24.232.1 – 10.24.235.254 | 10.24.235.255 | 10.24.232.1 |
| 9 | Sensor Lalu Lintas Utara | R-UTARA | 450 | /23 | 10.24.236.0 | 10.24.236.1 – 10.24.237.254 | 10.24.237.255 | 10.24.236.1 |
| 10 | Puskesmas dan Sekolah Barat | R-BARAT | 220 | /24 | 10.24.238.0 | 10.24.238.1 – 10.24.238.254 | 10.24.238.255 | 10.24.238.1 |
| 11 | Server dan Storage DC | R-DC | 180 | /24 | 10.24.239.0 | 10.24.239.1 – 10.24.239.254 | 10.24.239.255 | 10.24.239.1 |
| 12 | Kantor Pelabuhan | R-PELABUHAN | 120 | /25 | 10.24.240.0 | 10.24.240.1 – 10.24.240.126 | 10.24.240.127 | 10.24.240.1 |
| 13 | PJU Pintar Selatan | R-SELATAN | 90 | /25 | 10.24.240.128 | 10.24.240.129 – 10.24.240.254 | 10.24.240.255 | 10.24.240.129 |
| 14 | Server Balai Kota | R-PUSAT | 20 | /27 | 10.24.241.0 | 10.24.241.1 – 10.24.241.30 | 10.24.241.31 | 10.24.241.1 |
| 15 | L1 PUSAT-DC | R-PUSAT – R-DC | 2 | /30 | 10.24.241.32 | 10.24.241.33 – 10.24.241.34 | 10.24.241.35 | — |
| 16 | L2 PUSAT-UTARA | R-PUSAT – R-UTARA | 2 | /30 | 10.24.241.36 | 10.24.241.37 – 10.24.241.38 | 10.24.241.39 | — |
| 17 | L3 PUSAT-SELATAN | R-PUSAT – R-SELATAN | 2 | /30 | 10.24.241.40 | 10.24.241.41 – 10.24.241.42 | 10.24.241.43 | — |
| 18 | L4 PUSAT-TIMUR | R-PUSAT – R-TIMUR | 2 | /30 | 10.24.241.44 | 10.24.241.45 – 10.24.241.46 | 10.24.241.47 | — |
| 19 | L5 PUSAT-BARAT | R-PUSAT – R-BARAT | 2 | /30 | 10.24.241.48 | 10.24.241.49 – 10.24.241.50 | 10.24.241.51 | — |
| 20 | L6 UTARA-PELABUHAN | R-UTARA – R-PELABUHAN | 2 | /30 | 10.24.241.52 | 10.24.241.53 – 10.24.241.54 | 10.24.241.55 | — |
| 21 | L7 DC-UTARA (cadangan OSPF) | R-DC – R-UTARA | 2 | /30 | 10.24.241.56 | 10.24.241.57 – 10.24.241.58 | 10.24.241.59 | — |
 
Ringkasan: total alamat terpakai **61.756** dari 65.536 alamat dalam `10.24.0.0/16`, yaitu **94,2%** blok terpakai dan sisa 3.780 alamat. Jumlah host sebenarnya 42.080, atau **64,2%** dari blok.
 
### 2.4 Tabel CIDR (pembanding)
 
Pada CIDR, setiap router mendapat satu blok kontigu (supernet) yang cukup untuk semua LAN dan link miliknya, lalu blok-blok itu digabung dalam pohon supernet. Aturan penggabungan: prefix sama naik 1; prefix berbeda mengikuti blok terbesar, lalu naik 1.
 
| No | Subnet | Router | Host | Prefix | Network ID | Range host | Broadcast | Gateway |
|---:|---|---|---:|:---:|---|---|---|---|
| 1 | WiFi Publik Utara | R-UTARA | 11.000 | /18 | 10.24.0.0 | 10.24.0.1 – 10.24.63.254 | 10.24.63.255 | 10.24.0.1 |
| 2 | Sensor Lalu Lintas Utara | R-UTARA | 450 | /23 | 10.24.64.0 | 10.24.64.1 – 10.24.65.254 | 10.24.65.255 | 10.24.64.1 |
| 3 | L2 PUSAT-UTARA | R-PUSAT – R-UTARA | 2 | /30 | 10.24.66.0 | 10.24.66.1 – 10.24.66.2 | 10.24.66.3 | — |
| 4 | CCTV Kota | R-DC | 6.000 | /19 | 10.24.128.0 | 10.24.128.1 – 10.24.159.254 | 10.24.159.255 | 10.24.128.1 |
| 5 | Server dan Storage DC | R-DC | 180 | /24 | 10.24.160.0 | 10.24.160.1 – 10.24.160.254 | 10.24.160.255 | 10.24.160.1 |
| 6 | L1 PUSAT-DC | R-PUSAT – R-DC | 2 | /30 | 10.24.161.0 | 10.24.161.1 – 10.24.161.2 | 10.24.161.3 | — |
| 7 | L7 DC-UTARA (cadangan OSPF) | R-DC – R-UTARA | 2 | /30 | 10.24.161.4 | 10.24.161.5 – 10.24.161.6 | 10.24.161.7 | — |
| 8 | WiFi Publik Timur | R-TIMUR | 5.500 | /19 | 10.24.192.0 | 10.24.192.1 – 10.24.223.254 | 10.24.223.255 | 10.24.192.1 |
| 9 | Sensor Banjir Timur | R-TIMUR | 700 | /22 | 10.24.224.0 | 10.24.224.1 – 10.24.227.254 | 10.24.227.255 | 10.24.224.1 |
| 10 | L4 PUSAT-TIMUR | R-PUSAT – R-TIMUR | 2 | /30 | 10.24.228.0 | 10.24.228.1 – 10.24.228.2 | 10.24.228.3 | — |
| 11 | WiFi Publik Selatan | R-SELATAN | 6.500 | /19 | 10.25.0.0 | 10.25.0.1 – 10.25.31.254 | 10.25.31.255 | 10.25.0.1 |
| 12 | PJU Pintar Selatan | R-SELATAN | 90 | /25 | 10.25.32.0 | 10.25.32.1 – 10.25.32.126 | 10.25.32.127 | 10.25.32.1 |
| 13 | L3 PUSAT-SELATAN | R-PUSAT – R-SELATAN | 2 | /30 | 10.25.32.128 | 10.25.32.129 – 10.25.32.130 | 10.25.32.131 | — |
| 14 | WiFi Publik Barat | R-BARAT | 4.500 | /19 | 10.25.64.0 | 10.25.64.1 – 10.25.95.254 | 10.25.95.255 | 10.25.64.1 |
| 15 | Puskesmas dan Sekolah Barat | R-BARAT | 220 | /24 | 10.25.96.0 | 10.25.96.1 – 10.25.96.254 | 10.25.96.255 | 10.25.96.1 |
| 16 | L5 PUSAT-BARAT | R-PUSAT – R-BARAT | 2 | /30 | 10.25.97.0 | 10.25.97.1 – 10.25.97.2 | 10.25.97.3 | — |
| 17 | Sensor dan SCADA Pelabuhan | R-PELABUHAN | 5.000 | /19 | 10.25.128.0 | 10.25.128.1 – 10.25.159.254 | 10.25.159.255 | 10.25.128.1 |
| 18 | Kantor Pelabuhan | R-PELABUHAN | 120 | /25 | 10.25.160.0 | 10.25.160.1 – 10.25.160.126 | 10.25.160.127 | 10.25.160.1 |
| 19 | L6 UTARA-PELABUHAN | R-UTARA – R-PELABUHAN | 2 | /30 | 10.25.160.128 | 10.25.160.129 – 10.25.160.130 | 10.25.160.131 | — |
| 20 | Pegawai Balai Kota | R-PUSAT | 1.800 | /21 | 10.25.192.0 | 10.25.192.1 – 10.25.199.254 | 10.25.199.255 | 10.25.192.1 |
| 21 | Server Balai Kota | R-PUSAT | 20 | /27 | 10.25.200.0 | 10.25.200.1 – 10.25.200.30 | 10.25.200.31 | 10.25.200.1 |
 
Hasil pohon supernet (akar `10.24.0.0/15`):
 
| Router | Supernet | Isi |
|---|---|---|
| R-UTARA | `10.24.0.0/17` | WiFi Utara /18 + Sensor Lalu Lintas /23 + L2 |
| R-DC | `10.24.128.0/18` | CCTV /19 + Server DC /24 + L1 + L7 |
| R-TIMUR | `10.24.192.0/18` | WiFi Timur /19 + Sensor Banjir /22 + L4 |
| R-SELATAN | `10.25.0.0/18` | WiFi Selatan /19 + PJU /25 + L3 |
| R-BARAT | `10.25.64.0/18` | WiFi Barat /19 + Puskesmas /24 + L5 |
| R-PELABUHAN | `10.25.128.0/18` | SCADA /19 + Kantor Pelabuhan /25 + L6 |
| R-PUSAT | `10.25.192.0/20` | Pegawai /21 + Server Balai Kota /27 |
 
```
10.24.0.0/15
├─ 10.24.0.0/16
│   ├─ 10.24.0.0/17    R-UTARA
│   └─ 10.24.128.0/17
│       ├─ 10.24.128.0/18  R-DC
│       └─ 10.24.192.0/18  R-TIMUR
└─ 10.25.0.0/16
    ├─ 10.25.0.0/17
    │   ├─ 10.25.0.0/18    R-SELATAN
    │   └─ 10.25.64.0/18   R-BARAT
    └─ 10.25.128.0/17
        ├─ 10.25.128.0/18  R-PELABUHAN
        └─ 10.25.192.0/20  R-PUSAT
```
 
### 2.5 Perbandingan dan alasan memilih VLSM
 
| Aspek | VLSM | CIDR (supernet per router) |
|---|---|---|
| Blok induk | `10.24.0.0/16` (65.536 alamat) | `10.24.0.0/15` (131.072 alamat) |
| Alamat terpakai | 61.756 | 61.756 |
| Pemakaian blok | **94,2%** | 47,1% |
| Pemakaian oleh host sebenarnya | **64,2%** | 32,1% |
| Muat di jatah `/16` kelompok | Ya | Tidak, perlu `10.25.0.0/16` tambahan |
| Jumlah rute ringkasan | 21 subnet | 7 supernet (satu per router) |
 
Alasan memilih **VLSM**:
 
1. **Kebutuhan host sangat timpang.** Subnet terbesar 11.000 host (WiFi Publik Utara), sedangkan yang terkecil hanya 20 host (Server Balai Kota) dan 2 host (link antar-router). VLSM memberi ukuran yang pas untuk tiap subnet. Jika semua subnet disamakan sebesar yang terbesar (/18, 16.384 alamat), 21 subnet butuh 344.064 alamat, jauh melebihi satu blok /16.
2. **Muat dalam satu blok /16 jatah kelompok.** VLSM selesai di `10.24.241.59`, tetap dalam `10.24.0.0/16`. CIDR memerlukan `/15` yang meluas ke `10.25.0.0/16`, di luar jatah kelompok.
3. **Efisiensi alamat jauh lebih baik.** VLSM memakai 94,2% blok, sedangkan CIDR hanya 47,1% karena setiap router dibulatkan ke blok pangkat dua (misalnya R-UTARA butuh 16.900 alamat tetapi mendapat blok 32.768).
4. **Link point-to-point hemat.** 7 link memakai `/30` (4 alamat per link, total 28 alamat). Dengan blok yang lebih besar, link-link ini akan memboroskan ribuan alamat.
5. **Kekurangan VLSM sudah diantisipasi.** VLSM menghasilkan lebih banyak rute (21 subnet), dan subnet satu router tidak selalu berurutan (contoh: WiFi Utara `10.24.0.0/18` dan Sensor Lalu Lintas `10.24.236.0/23`), sehingga ringkasan rute per router tidak rapi. Pada project ini hal itu ditangani oleh routing (static route dan OSPF yang dikerjakan anggota lain), sehingga keunggulan CIDR berupa tabel rute yang ringkas tidak sebanding dengan pemborosan alamatnya.
Kesimpulan: **VLSM dipilih sebagai skema utama**, dan CIDR dipakai sebagai bahan perbandingan.
 
---

## 3. Penjelasan Desain Topologi
 
### 3.1 Arsitektur: hub-and-spoke dengan R-PUSAT sebagai pusat
 
- Lima wilayah (DC, Utara, Selatan, Timur, Barat) masing-masing tersambung langsung ke R-PUSAT lewat link L1 sampai L5. Semua trafik antar-wilayah dan trafik keluar melewati satu titik pusat, sehingga mudah dikelola dan dipantau.
- **R-PELABUHAN** tersambung ke R-UTARA lewat L6, dan ke R-PUSAT lewat **tunnel GRE** virtual (`192.168.3.0/30`) sebagai jalur langsung ke pusat.
- **R-ISP** berada di atas R-PUSAT. Hanya R-PUSAT yang berhubungan dengan internet (`203.0.113.12/30`), sehingga NAT dan akses keluar terkonsentrasi di satu tempat. SRV-INTERNET (`8.8.8.8`) mewakili server di internet.
- Pada tabel alamat, link cadangan **L7** (`10.24.241.56/30`, antara R-DC dan R-UTARA) sudah dialokasikan untuk skenario OSPF. Pada Gambar 1, kabel L7 belum terpasang.
### 3.2 Peran dan isi tiap router
 
| Router | Lokasi / peran | LAN yang dilayani | Link ke router lain |
|---|---|---|---|
| R-ISP | Penyedia internet (simulasi) | `8.8.8.0/24` (SRV-INTERNET) | R-PUSAT |
| R-PUSAT | Balai Kota, hub utama dan gerbang internet | Pegawai Balai Kota `/21`, Server Balai Kota `/27` | R-ISP, L1 sampai L5, tunnel GRE |
| R-DC | Data Center dan Command Center | CCTV Kota `/19`, Server & Storage DC `/24` | L1 (L7 cadangan) |
| R-UTARA | Wilayah Utara dan transit ke pelabuhan | WiFi Publik Utara `/18`, Sensor Lalu Lintas `/23` | L2, L6 (L7 cadangan) |
| R-SELATAN | Wilayah Selatan | WiFi Publik Selatan `/19`, PJU Pintar `/25` | L3 |
| R-TIMUR | Wilayah Timur | WiFi Publik Timur `/19`, Sensor Banjir `/22` | L4 |
| R-BARAT | Wilayah Barat | WiFi Publik Barat `/19`, Puskesmas & Sekolah `/24` | L5 |
| R-PELABUHAN | Pelabuhan | Sensor & SCADA `/19`, Kantor Pelabuhan `/25` | L6, tunnel GRE |
 
Prinsip desain di tiap LAN: **satu LAN = satu port router + satu switch + satu perangkat wakil**. Router menjadi gateway (`.1` dari network). Setiap LAN berada di domain broadcast sendiri, sehingga broadcast dari WiFi publik berukuran besar tidak mengganggu LAN lain.
 
### 3.3 Pemilihan perangkat
 
| Perangkat | Alasan |
|---|---|
| **Cisco 2911** (router) | Mendukung static route, OSPF, DHCP relay, NAT, dan tunnel GRE yang dibutuhkan project. Port dapat ditambah dengan modul. |
| **Modul serial HWIC-2T** (2 port serial) | Router 2911 hanya punya 3 port GigabitEthernet bawaan. R-PUSAT membutuhkan 5 link backbone, 1 link ISP, dan 2 LAN. Link backbone dipindah ke serial supaya port Gigabit tersisa untuk LAN dan ISP. |
| **Cisco 2960-24TT** (switch) | Satu switch per LAN, tanpa konfigurasi tambahan karena LAN datar (tanpa VLAN). 24 port lebih dari cukup untuk perangkat wakil. |
| **PC-PT** | Perangkat wakil untuk CCTV, sensor, SCADA, dan kantor. |
| **Server-PT** | Menjalankan layanan: SRV-DC (DNS, DHCP CCTV, FTP), SRV-BALAIKOTA (web, email), SRV-INTERNET (server `8.8.8.8`). |
| **AccessPoint-PT** | Menyediakan WiFi Publik di Utara, Timur, Selatan, dan Barat. |
 
Kebutuhan modul serial per router (berdasarkan port yang dipakai di tabel interface):
 
| Router | Port serial dipakai | Modul 2 port |
|---|---|:---:|
| R-PUSAT | Se0/0/0, Se0/0/1, Se0/1/0, Se0/1/1, Se0/2/0 | 3 |
| R-UTARA | Se0/0/0 (L2), Se0/0/1 (L6), Se0/1/0 (L7 cadangan) | 2 |
| R-DC | Se0/0/0 (L1), Se0/0/1 (L7 cadangan) | 1 |
| R-SELATAN, R-TIMUR, R-BARAT, R-PELABUHAN | Se0/0/0 | 1 per router |
 
### 3.4 Jenis kabel dan alasan pemakaiannya
 
| Kabel | Dipakai untuk | Jumlah | Alasan |
|---|---|:---:|---|
| Copper Straight-Through | Router (Gi) ke switch, switch ke PC / server / access point | 28 | Menghubungkan perangkat yang berbeda jenis. 14 LAN × 2 kabel. |
| Copper Cross-Over | R-ISP ke R-PUSAT, R-ISP ke SRV-INTERNET | 2 | Menghubungkan router ke router, dan router ke server (perangkat sejenis). |
| Serial DCE/DTE | L1 sampai L6 (R-PUSAT ke R-DC, R-UTARA, R-SELATAN, R-TIMUR, R-BARAT; R-UTARA ke R-PELABUHAN) | 6 | Meniru link WAN point-to-point antar-wilayah, dan menghemat port Gigabit. Salah satu ujung (DCE) perlu `clock rate`. |
| Tunnel GRE | R-PUSAT ke R-PELABUHAN | 0 (virtual) | Dibuat dengan konfigurasi, bukan kabel. |
 
Total kabel fisik pada Gambar 1 adalah 36.
 
### 3.5 Penempatan IP di tiap interface router (skema VLSM)
 
| Router | Interface | Terhubung ke | Network | IP address | Prefix |
|---|---|---|---|---|:---:|
| R-ISP | Gi0/0 | R-PUSAT | 203.0.113.12/30 | 203.0.113.14 | /30 |
| R-ISP | Gi0/1 | Server 8.8.8.8 | 8.8.8.0/24 | 8.8.8.1 | /24 |
| R-PUSAT | Gi0/0 | R-ISP | 203.0.113.12/30 | 203.0.113.13 | /30 |
| R-PUSAT | Se0/0/0 | R-DC | 10.24.241.32/30 | 10.24.241.33 | /30 |
| R-PUSAT | Se0/0/1 | R-UTARA | 10.24.241.36/30 | 10.24.241.37 | /30 |
| R-PUSAT | Se0/1/0 | R-SELATAN | 10.24.241.40/30 | 10.24.241.41 | /30 |
| R-PUSAT | Se0/1/1 | R-TIMUR | 10.24.241.44/30 | 10.24.241.45 | /30 |
| R-PUSAT | Se0/2/0 | R-BARAT | 10.24.241.48/30 | 10.24.241.49 | /30 |
| R-PUSAT | Gi0/2 | Pegawai Balai Kota | 10.24.224.0/21 | 10.24.224.1 | /21 |
| R-PUSAT | Gi0/1 | Server Balai Kota | 10.24.241.0/27 | 10.24.241.1 | /27 |
| R-PUSAT | Tunnel0 | R-PELABUHAN | 192.168.3.0/30 | 192.168.3.1 | /30 |
| R-DC | Se0/0/0 | R-PUSAT | 10.24.241.32/30 | 10.24.241.34 | /30 |
| R-DC | Se0/0/1 | R-UTARA | 10.24.241.56/30 | 10.24.241.57 | /30 |
| R-DC | Gi0/1 | CCTV Kota | 10.24.96.0/19 | 10.24.96.1 | /19 |
| R-DC | Gi0/2 | Server dan Storage DC | 10.24.239.0/24 | 10.24.239.1 | /24 |
| R-UTARA | Se0/0/0 | R-PUSAT | 10.24.241.36/30 | 10.24.241.38 | /30 |
| R-UTARA | Se0/0/1 | R-PELABUHAN | 10.24.241.52/30 | 10.24.241.53 | /30 |
| R-UTARA | Se0/1/0 | R-DC | 10.24.241.56/30 | 10.24.241.58 | /30 |
| R-UTARA | Gi0/2 | WiFi Publik Utara | 10.24.0.0/18 | 10.24.0.1 | /18 |
| R-UTARA | Gi0/1 | Sensor Lalu Lintas Utara | 10.24.236.0/23 | 10.24.236.1 | /23 |
| R-SELATAN | Se0/0/0 | R-PUSAT | 10.24.241.40/30 | 10.24.241.42 | /30 |
| R-SELATAN | Gi0/1 | WiFi Publik Selatan | 10.24.64.0/19 | 10.24.64.1 | /19 |
| R-SELATAN | Gi0/2 | PJU Pintar Selatan | 10.24.240.128/25 | 10.24.240.129 | /25 |
| R-TIMUR | Se0/0/0 | R-PUSAT | 10.24.241.44/30 | 10.24.241.46 | /30 |
| R-TIMUR | Gi0/1 | WiFi Publik Timur | 10.24.128.0/19 | 10.24.128.1 | /19 |
| R-TIMUR | Gi0/2 | Sensor Banjir Timur | 10.24.232.0/22 | 10.24.232.1 | /22 |
| R-BARAT | Se0/0/0 | R-PUSAT | 10.24.241.48/30 | 10.24.241.50 | /30 |
| R-BARAT | Gi0/1 | WiFi Publik Barat | 10.24.192.0/19 | 10.24.192.1 | /19 |
| R-BARAT | Gi0/2 | Puskesmas dan Sekolah Barat | 10.24.238.0/24 | 10.24.238.1 | /24 |
| R-PELABUHAN | Se0/0/0 | R-UTARA | 10.24.241.52/30 | 10.24.241.54 | /30 |
| R-PELABUHAN | Gi0/0 | Sensor dan SCADA Pelabuhan | 10.24.160.0/19 | 10.24.160.1 | /19 |
| R-PELABUHAN | Gi0/1 | Kantor Pelabuhan | 10.24.240.0/25 | 10.24.240.1 | /25 |
| R-PELABUHAN | Tunnel0 | R-PUSAT | 192.168.3.0/30 | 192.168.3.2 | /30 |
 

