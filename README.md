Laravel + PostgreSQL on Fedora Linux
Panduan ini menjelaskan setup Laravel dengan PostgreSQL di Fedora Linux, dari instalasi PHP dan Composer hingga konfigurasi database, .env, migration, dan development server.
Daftar Isi
    1. Prasyarat
    2. Update Sistem
    3. Instalasi PHP
    4. Instalasi Composer
    5. Instalasi PostgreSQL
    6. Konfigurasi Database
    7. Konfigurasi pg_hba.conf
    8. PHP PostgreSQL Extension
    9. Membuat Project Laravel
    10. Konfigurasi .env
    11. Migration
    12. Menjalankan Laravel
    13. Verifikasi
    14. Troubleshooting
    15. Catatan Keamanan
Prasyarat
    • Fedora Linux
    • Akun dengan akses sudo
    • Koneksi internet
    • Terminal
Panduan ini ditujukan untuk development environment. Jangan gunakan password contoh seperti 123 untuk production.
1. Update Sistem
sudo dnf update -y
Perintah ini memperbarui paket Fedora yang sudah terpasang.
2. Instalasi PHP
Aktifkan PHP 8.4 dari repository/module Remi:
sudo dnf module reset php
sudo dnf module enable php:remi-8.4 -y
Install PHP dan extension yang umum dibutuhkan Laravel:
sudo dnf install php php-cli php-fpm php-zip php-devel php-gd php-mbstring php-curl php-xml php-pear php-bcmath php-json php-opcache php-intl -y
Periksa versi:
php -v
Pastikan versi PHP sesuai dengan requirement versi Laravel yang digunakan.
3. Instalasi Composer
Composer adalah dependency manager untuk PHP dan digunakan Laravel untuk mengelola framework serta package.
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
sudo mv composer.phar /usr/local/bin/composer
Verifikasi:
composer --version
4. Instalasi PostgreSQL
Install PostgreSQL:
sudo dnf install postgresql postgresql-server postgresql-contrib -y
Inisialisasi database cluster:
sudo postgresql-setup --initdb
Jalankan dan aktifkan PostgreSQL:
sudo systemctl enable --now postgresql
Periksa status:
sudo systemctl status postgresql
Status active (running) berarti PostgreSQL berjalan.
5. Konfigurasi Database PostgreSQL
Masuk sebagai administrator PostgreSQL:
sudo -u postgres psql
Atur password administrator:
\password postgres
Buat database:
CREATE DATABASE laravel_db;
Buat user khusus aplikasi:
CREATE USER laravel_user WITH PASSWORD 'GANTI_DENGAN_PASSWORD_ANDA';
Berikan akses database:
GRANT ALL PRIVILEGES ON DATABASE laravel_db TO laravel_user;
Pada PostgreSQL modern, permission schema dapat diperlukan untuk migration. Masuk ke database:
\c laravel_db
GRANT USAGE, CREATE ON SCHEMA public TO laravel_user;
Keluar:
\q
Mengapa membuat user khusus? Agar aplikasi Laravel tidak menggunakan akun administrator PostgreSQL.
6. Konfigurasi pg_hba.conf
File ini menentukan metode autentikasi koneksi PostgreSQL.
Buka:
sudo nano /var/lib/pgsql/data/pg_hba.conf
Untuk instalasi PostgreSQL modern, konfigurasi lokal dapat menggunakan:
local   all   all                         scram-sha-256
host    all   all   127.0.0.1/32         scram-sha-256
host    all   all   ::1/128               scram-sha-256
SCRAM-SHA-256 direkomendasikan untuk instalasi PostgreSQL modern. Jika tutorial lama meminta md5, jangan mengubah konfigurasi secara membabi buta; sesuaikan aturan yang diperlukan.
Setelah perubahan:
sudo systemctl restart postgresql
7. PHP PostgreSQL Extension
Install driver PostgreSQL untuk PHP:
sudo dnf install php-pgsql php-pdo_pgsql -y
Restart PHP-FPM:
sudo systemctl restart php-fpm
Verifikasi:
php -m | grep -E 'pgsql|pdo_pgsql'
Output seharusnya memuat:
pdo_pgsql
pgsql
8. Membuat Project Laravel
Buat project:
composer create-project laravel/laravel belajar-laravel
Masuk ke project:
cd belajar-laravel
Jika .env belum tersedia:
cp .env.example .env
Generate application key:
php artisan key:generate
APP_KEY digunakan Laravel untuk kebutuhan keamanan seperti enkripsi. Jangan membagikannya atau memasukkannya ke repository.
9. Konfigurasi .env
Buka:
nano .env
Atur database menjadi:
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=GANTI_DENGAN_PASSWORD_ANDA
Keterangan:
Variable          Fungsi

DB_CONNECTION   Driver database DB_HOST         Alamat PostgreSQL DB_PORT         Port PostgreSQL, biasanya 5432 DB_DATABASE     Nama database DB_USERNAME     User database DB_PASSWORD     Password user database
Jangan commit .env karena dapat berisi credential.
Jika Laravel masih menggunakan konfigurasi lama:
php artisan config:clear
10. Menjalankan Migration
Jalankan:
php artisan migrate
Migration akan membuat tabel yang diperlukan Laravel pada laravel_db.
Jika berhasil, berarti Laravel sudah dapat terhubung ke PostgreSQL.
11. Menjalankan Laravel
Jalankan development server:
php artisan serve
Biasanya aplikasi tersedia di:
http://127.0.0.1:8000
Hentikan server dengan Ctrl + C.
12. Verifikasi Instalasi
PHP:
php -v
Composer:
composer --version
PostgreSQL:
sudo systemctl status postgresql
Driver PHP:
php -m | grep -E 'pgsql|pdo_pgsql'
Laravel:
php artisan --version
Status migration:
php artisan migrate:status
Jika migrate:status dapat berjalan tanpa error koneksi, Laravel berhasil berkomunikasi dengan PostgreSQL.
Troubleshooting
could not find driver
Install driver:
sudo dnf install php-pgsql php-pdo_pgsql -y
Periksa:
php -m | grep -E 'pgsql|pdo_pgsql'
password authentication failed
Periksa kembali:
DB_USERNAME=laravel_user
DB_PASSWORD=your_password
Kemudian:
php artisan config:clear
connection refused
Periksa PostgreSQL:
sudo systemctl status postgresql
Jika berhenti:
sudo systemctl start postgresql
Pastikan .env menggunakan:
DB_HOST=127.0.0.1
DB_PORT=5432
permission denied for schema public
Masuk ke PostgreSQL:
sudo -u postgres psql
Lalu:
\c laravel_db
GRANT USAGE, CREATE ON SCHEMA public TO laravel_user;
Keluar:
\q
Kemudian:
php artisan migrate
Alur Setup
Fedora Linux
    ↓
PHP + Extensions
    ↓
Composer
    ↓
PostgreSQL
    ↓
Database + User
    ↓
PHP PostgreSQL Driver
    ↓
Laravel Project
    ↓
.env
    ↓
php artisan migrate
    ↓
php artisan serve
Catatan Keamanan
Untuk development, konfigurasi di atas sudah cukup sebagai dasar. Untuk production:
    • gunakan password yang kuat;
    • jangan gunakan user PostgreSQL administrator untuk aplikasi;
    • jangan commit .env;
    • gunakan HTTPS;
    • batasi akses database;
    • siapkan backup database;
    • konfigurasi web server dan PHP-FPM dengan benar;
    • sesuaikan versi PHP dengan requirement Laravel;
    • gunakan secret management yang sesuai.
Kesimpulan
Setup Laravel + PostgreSQL di Fedora terdiri dari beberapa lapisan: PHP menjalankan Laravel, Composer mengelola dependency, PostgreSQL menyimpan data, dan konfigurasi .env menghubungkan Laravel dengan database.
Setelah php artisan migrate berhasil dan php artisan serve dapat membuka aplikasi, environment dasar Laravel + PostgreSQL sudah siap digunakan untuk pengembangan lebih lanjut.
