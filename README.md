# digisensi
Absensi Digital Berbasis Web Dengan Face Recognition

## About

Merupakan software sederhana yang mengimplementasikan absensi dengan pengenalan wajah.

Software ini dapat digunakan untuk absensi di perkantoran atau sekolah-sekolah.

## Installation

### Requirements

#### Mesin Server

Untuk tahap ujicoba, komputer dengan Intel Core I3, 4GB RAM dan HDD 7.2Krpm seharusnya sudah mencukupi.

Untuk tahap produksi, perlu disiapkan server yang sesungguhnya, yang sebaiknya setara Intel Xeon 4210 (atau di atasnya), RAM 8GB (atau di atasnya) dan Storage SSD yang mencukupi. Tambahan GPU akan sangat membantu meningkatkan performa. VPS dengan spesifikasi memadai pun dapat digunakan sebagai mesin server.



#### Browser

Browser modern yang mendukung akses kamera dapat digunakan. Sisi frontend telah diuji-coba pada browser:
* Safari Desktop
** Mac (Catalina), 
** Windows 7, Windows 10, Windows 11
* Safari Mobile
** iOS
* Firefox Desktop
** Mac (Catalina), 
** Windows 7, Windows 10, Windows 11
* Firefox Mobile
** iOS
** Android
* Chrome Desktop
** Mac (Catalina), 
** Windows 7, Windows 10, Windows 11
* Chrome Mobile
** iOS
** Android

Kami belum memiliki kesempatan untuk menguji frontend di Linux.

#### Backend Software



### Permissions

### SSL

Untuk mendukung akses kamera, frontend harus diinstall di domain publik yang diproteksi SSL. Dengan demikian, sertifikasi SSL dari Otoritas Publik harus disiapkan. Kami telah menjalankan software ini pada level produksi menggunakan sertifikat SSL gratis dari [Let's Encrypt](https://letsencrypt.org/), dan sejauh ini baik-baik saja.

Anda tentu dapat menggunakan sertifikat SSL dari Otoritas lain atau yang berbayar. Tapi jika Anda membutuhkan yang gratis (misalnya untuk tahap uji-coba), [Let's Encrypt](https://letsencrypt.org/) cukup handal dan telah menerbitkan sertfikat SSL untuk lebih dari 260 juta website.

## How It Works

Proses absensi dilakukan dengan pengambilan foto wajah pegawai/siswa menggunakan kamera handphone/laptop. Software akan mengirim foto wajah ke backend server untuk dianalisis dan dikomparasi dengan identifikasi wajah yang terdaftar dalam database. Apabila ditemukan kecocokan, ID pengenal untuk wajah tersebut dicatat ke database kehadiran beserta dengan waktu saat itu, yang nantinya menjadi dasar perhitungan waktu kehadiran pegawai/siswa.

### Frontend

Proses pengambilan foto memerlukan login, untuk mencegah aktivitas user yang tidak terdaftar dalam database pengguna. Proses login, pengambilan foto dan utilitas lainnya pada sisi frontend dibuat dengan framework [Onsen UI](https://onsen.io), [JQuery](https://jquery.com) dan Javascript. Onsen UI digunakan sebagai UI framework karena mendukung *mobile-like navigation*, di mana navigasi antar fragment UI (page) tidak perlu memuat halaman baru, dan karena Onsen UI memang didesain lebih ramping dan hemat resource.

### Backend

Backend dibangun dengan [Node.js](https://nodejs.org) dengan [Express](https://expressjs.com) middleware beserta beberapa framework pendukung lainnya. Ada pun Facial Recognition Engine adalah [Face-Api.js](https://github.com/justadudewhohacks/face-api.js) yang dimodifikasi untuk berjalan di platform Node.js. 

Backend telah diuji-coba dan berjalan dengan baik pada tahap produksi pada Windows Server 2012 R2 yang diinstall pada mesin HPE Proliant ML10 Gen-9 (Intel Xeon, 16GB RAM (upgrade dari 8GB), dan 480GB SATA SSD (upgrade dari 1TB HDD)). Namun pada tahap ujicoba, mesin dengan Intel Core I3, 4GB RAM dan HDD sudah mencukupi.

Framework Face-Api.js sebenarnya dapat dijalankan di sisi browser, sehingga beban CPU saat analisa dan identifikasi wajah dapat didistribusikan ke CPU device tiap user, namun browser-based scripting untuk proses ini ternyata sangat lambat dan ekstensif, sehingga pengalihan implementasi proses ini ke sisi backend merupakan pilihan terbaik. Dengan pengalihan proses analisa dan identifikasi wajah ke sisi backend, framework Face-Api.js dapat ditunjang dengan TensorFlow Node.js Backend, yang pada akhirnya meningkatkan performa sampai rata-rata 750%, dan menghemat waktu hingga 6 ~ 7 kali lebih cepat.

TensorFlow Node.js Backend terbagi atas dua paket, yaitu CPU-based dan GPU-based. Di sini saya menganggap bahwa mesin server mungkin tidak memiliki GPU, sehingga lebih aman jika menggunakan paket CPU-based. Namun jika Anda ingin mencoba pada mesin yang memiliki GPU dan cocok dengan spesifikasi TensorFlow (CUDA), tentu paket GPU-based akan memberikan performa lebih baik.

### SSL Certificate

Browser-browser saat ini tidak lagi mendukung akses kamera oleh halaman web pada URL non-HTTPS. Untuk mengatasi masalah ini, backend perlu dipublikasi dengan nama domain dan IP address yang dapat diakses secara publik, dan harus mendukung akses HTTPS. Untuk ini, perlu nama domain dengan A Record yang menunjuk ke IP address mesin server di mana backend server dijalankan.

Lebih lanjut, pada backend server perlu dipasangi sertifikat SSL, sehingga akses browser dapat berjalan tanpa kendala.

### Database

Database yang digunakan adalah PostgreSQL server 10.x, yang dipilih karena fleksibilitas Server Side Scripting (Trigger, Stored Function dsb.) selain dari kecepatan akses data.

### Keamanan Proses Absensi

