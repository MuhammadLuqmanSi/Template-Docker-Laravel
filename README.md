# Docker Laravel Template – Cara Pakai

Template ini menyiapkan **Laravel + Apache + MySQL + phpMyAdmin** dengan Docker Compose. Cocok untuk dipakai sebagai template GitHub agar teman/teammate bisa langsung jalan tanpa setup ribet.

> **Catatan:** Template sudah menyertakan `docker/` (Dockerfile, config Apache, init script) dan `docker-compose.yml`. Folder project Laravel akan dipasang (mount) dari host ke container (default: `./src`).

---

## Prasyarat

* **Docker** & **Docker Compose** (Compose v2 bawaan Docker Desktop juga bisa).
* **Git** (jika kloning dari GitHub).
* (Opsional) **Node.js + npm/pnpm/yarn** untuk frontend asset Laravel (Vite).

---

## Struktur Projek

```
Template-docker-laravel/
├─ .env                      # Konfigurasi Compose & service
├─ docker-compose.yml        # Definisi container
├─ docker/
│  ├─ Dockerfile             # PHP 8.2 + Apache + ext Laravel
│  ├─ apache.conf            # DocumentRoot ke /public
│  └─ init.sh                # Auto-setup Laravel & permission
└─ src/                      # (Default) tempat source Laravel kamu
```

**DocumentRoot** sudah diarahkan ke `public/` (lihat `docker/apache.conf`).

---

## Konfigurasi `.env`

File `.env` di root template **bukan** `.env` Laravel, melainkan env untuk Docker Compose.

Nilai default penting:

```
COMPOSE_PROJECT_NAME=my-laravel
APP_PATH=./src
APP_PORT=80
PMA_PORT=8001
MYSQL_PORT_HOST=3310
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel
DB_PASSWORD=secret
MYSQL_ROOT_PASSWORD=root
```

* **APP\_PORT**: port aplikasi Laravel di host → akses `http://localhost:80`.
* **PMA\_PORT**: port phpMyAdmin di host → akses `http://localhost:8001`.
* **MYSQL\_PORT\_HOST**: port MySQL di host (untuk koneksi dari tool di host, mis. TablePlus/HeidiSQL) → `localhost:3310`.
* **APP\_PATH**: path folder Laravel di host yang akan di-mount ke container. Default `./src`.

> Ubah nilai di `.env` sesuai kebutuhan sebelum menjalankan.

---

## Menjalankan

> Jalankan semua perintah dari folder root template.

### 1) Start services

```bash
# Linux/Mac/WSL/Git Bash
cp .env .env.local 2>/dev/null || true   # opsional, kalau mau copy dulu

# jalankan docker
docker compose --env-file .env up -d --build
```

Perintah di atas akan membuat 3 service utama:

* `app` (PHP 8.2 + Apache)
* `mysql` (MySQL)
* `phpmyadmin`

### 2) Akses

* App: `http://localhost:<APP_PORT>` (default `http://localhost:80`)
* phpMyAdmin: `http://localhost:<PMA_PORT>` (default `http://localhost:8001`)

  * Server: `mysql`
  * User/Password: pakai `DB_USERNAME` / `DB_PASSWORD` dari `.env`

---

## Mode Pemakaian

Script `docker/init.sh` akan membantu otomatis.

### A. Mulai **Proyek Baru** (src masih kosong / belum ada `artisan`)

1. Pastikan `APP_PATH` mengarah ke folder kosong (default `./src`).
2. Jalankan:

   ```bash
   docker compose up -d --build
   ```
3. Container `app` akan mendeteksi belum ada `artisan` lalu **membuat proyek Laravel baru** dan menjalankan setelan awal (install composer, generate key, set permission).
4. Buka `http://localhost:80` → harusnya muncul halaman Laravel.

> Jika halaman belum tampil, lihat log: `docker compose logs -f app`.

### B. Pakai **Proyek Laravel yang Sudah Ada**

1. Salin kode Laravel kamu ke folder sesuai `APP_PATH` (mis. `./src`). Pastikan ada `artisan`.
2. Jalankan services:

   ```bash
   docker compose up -d --build
   ```
3. Masuk ke container untuk instal dependensi & migrasi:

   ```bash
   docker compose exec app bash
   composer install
   cp .env.example .env || true
   php artisan key:generate
   php artisan migrate
   exit
   ```
4. Buka `http://localhost:80`.

### Frontend Asset (Vite)

Jika proyek menggunakan Vite:

```bash
docker compose exec app bash
# pilih salah satu package manager\	npm install
	npm run dev       # atau: npm run build
```

> Atau jalankan Node di host langsung (di dalam `./src`). Sesuaikan preferensi tim.

---

## Perintah Umum

```bash
# Melihat log
docker compose logs -f app

# Masuk ke container (shell)
docker compose exec app bash

# Jalankan artisan
docker compose exec app php artisan migrate

# Stop & hapus container (data DB tetap ada)
docker compose down

# Bersihkan volume DB (hapus data MySQL)
docker compose down -v
```

---

## Tips Git & Template

* Tambahkan `.gitignore` agar repo tetap ringan:

  ```gitignore
  /src/vendor/
  /src/node_modules/
  /src/public/build/
  .env
  .env.local
  .DS_Store
  ```
* **Jangan** commit `vendor` atau `node_modules`. Temanmu cukup menjalankan `composer install` dan `npm install` di lingkungan mereka (di container atau host).
* Kalau repo ini mau dipakai sebagai **template GitHub**, aktifkan *Use this template* saat membuat repo baru.

---

## Koneksi Database dari Host (opsional)

Gunakan tool seperti TablePlus/HeidiSQL:

* Host: `127.0.0.1`
* Port: `MYSQL_PORT_HOST` (default `3310`)
* User/Pass: `DB_USERNAME` / `DB_PASSWORD`
* Database: `DB_DATABASE`

---

## Trouble Shooting

**Port sudah terpakai (80/8001/3310)**
Ubah `APP_PORT`, `PMA_PORT`, `MYSQL_PORT_HOST` di `.env`, lalu `docker compose up -d`.

**Halaman 403/404**
Pastikan `public/` ada dan `apache.conf` sudah mengarah ke `DocumentRoot /var/www/html/public` (default template sudah benar). Cek permission `storage` dan `bootstrap/cache`.

**Tidak bisa migrasi**
Cek koneksi DB (`DB_HOST=mysql`, `DB_PORT=3306`, user/pass benar). Jalankan `docker compose logs -f mysql` untuk memastikan MySQL sudah siap.

**Perubahan file tidak terdeteksi**
Di Windows, pakai WSL/Git Bash untuk performa bind mount yang lebih baik. Atau restart container `app`.

---

## FAQ

**Q: Boleh ganti versi PHP/MySQL?**
A: Boleh. Ubah `docker/Dockerfile` (base image PHP) atau versi image MySQL di `docker-compose.yml`.

**Q: Bisa deploy ke server pakai ini?**
A: Template ini fokus untuk **development**. Untuk produksi, pertimbangkan setup terpisah (nginx/php-fpm, persistent storage, dan keamanan tambahan).

---

## Lisensi

Bebas dipakai untuk keperluan pribadi/komersial. Beri kredit jika dirasa bermanfaat. 🙌
