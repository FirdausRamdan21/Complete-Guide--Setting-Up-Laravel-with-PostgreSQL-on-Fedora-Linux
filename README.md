# Laravel + PostgreSQL Setup on Fedora Linux

This guide provides a complete, step-by-step procedure for setting up Laravel with PostgreSQL on Fedora Linux. It covers system preparation, PHP and Composer installation, PostgreSQL configuration, database creation, Laravel project setup, environment configuration, migrations, and verification.

The instructions are intended for **development environments only**. Do not use example passwords or insecure configurations in production.

---

## Table of Contents

1. Prerequisites
2. System Update
3. PHP Installation
4. Composer Installation
5. PostgreSQL Installation
6. Database Configuration
7. pg_hba.conf Configuration
8. PHP PostgreSQL Extension
9. Creating a Laravel Project
10. .env Configuration
11. Running Migrations
12. Running Laravel
13. Verification
14. Troubleshooting
15. Security Notes
16. Conclusion

---

## 1. Prerequisites

Before starting, ensure you have:

- Fedora Linux installed
- A user account with `sudo` privileges
- An active internet connection
- A terminal emulator

---

## 2. System Update

Update all installed packages to their latest versions:

sudo dnf update -y

This ensures system stability and compatibility with the packages you will install next.
3. PHP Installation

Laravel requires PHP 8.1 or higher. This guide uses PHP 8.4 from the Remi repository.

Enable the Remi module:
bash

sudo dnf module reset php
sudo dnf module enable php:remi-8.4 -y

Install PHP and commonly required extensions for Laravel:
bash

sudo dnf install php php-cli php-fpm php-zip php-devel php-gd php-mbstring php-curl php-xml php-pear php-bcmath php-json php-opcache php-intl -y

Verify the PHP version:
bash

php -v

Ensure the version meets the requirements of the Laravel version you intend to use.
4. Composer Installation

Composer is the dependency manager for PHP and is required by Laravel.

Download and install Composer:
bash

php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
sudo mv composer.phar /usr/local/bin/composer

Verify the installation:
bash

composer --version

5. PostgreSQL Installation

Install PostgreSQL and its contributed utilities:
bash

sudo dnf install postgresql postgresql-server postgresql-contrib -y

Initialize the database cluster:
bash

sudo postgresql-setup --initdb

Enable and start the PostgreSQL service:
bash

sudo systemctl enable --now postgresql

Check the service status:
bash

sudo systemctl status postgresql

A status of active (running) indicates PostgreSQL is operational.
6. Database Configuration

Log in as the PostgreSQL administrator:
bash

sudo -u postgres psql

Set a password for the postgres user:
sql

\password postgres

Create a new database for the Laravel application:
sql

CREATE DATABASE laravel_db;

Create a dedicated application user:
sql

CREATE USER laravel_user WITH PASSWORD 'REPLACE_WITH_YOUR_PASSWORD';

Grant all privileges on the database to the new user:
sql

GRANT ALL PRIVILEGES ON DATABASE laravel_db TO laravel_user;

In modern PostgreSQL versions, schema permissions are required for migrations. Connect to the database and grant the necessary rights:
sql

\c laravel_db
GRANT USAGE, CREATE ON SCHEMA public TO laravel_user;

Exit the PostgreSQL prompt:
sql

\q

Why create a dedicated user?
Using a separate user prevents the Laravel application from running with administrator privileges, which is a critical security practice.
7. pg_hba.conf Configuration

The pg_hba.conf file controls client authentication for PostgreSQL. Edit the file:
bash

sudo nano /var/lib/pgsql/data/pg_hba.conf

For modern PostgreSQL installations, use scram-sha-256 for local and host connections:
conf

local   all   all                         scram-sha-256
host    all   all   127.0.0.1/32         scram-sha-256
host    all   all   ::1/128               scram-sha-256

scram-sha-256 is the recommended authentication method. Do not blindly change settings to md5 based on outdated tutorials.

After making changes, restart PostgreSQL:
bash

sudo systemctl restart postgresql

8. PHP PostgreSQL Extension

Install the PostgreSQL driver for PHP:
bash

sudo dnf install php-pgsql php-pdo_pgsql -y

Restart PHP-FPM to load the new extensions:
bash

sudo systemctl restart php-fpm

Verify the extensions are loaded:
bash

php -m | grep -E 'pgsql|pdo_pgsql'

Expected output:
text

pdo_pgsql
pgsql

9. Creating a Laravel Project

Create a new Laravel project using Composer:
bash

composer create-project laravel/laravel laravel-app

Navigate into the project directory:
bash

cd laravel-app

If the .env file is missing, copy the example file:
bash

cp .env.example .env

Generate the application key:
bash

php artisan key:generate

The APP_KEY is used by Laravel for encryption and other security features. Never share it or commit it to a repository.
10. .env Configuration

Open the .env file for editing:
bash

nano .env

Update the database connection settings:
env

DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=REPLACE_WITH_YOUR_PASSWORD

Variable reference:
Variable	Purpose
DB_CONNECTION	Database driver
DB_HOST	PostgreSQL host address
DB_PORT	PostgreSQL port (default 5432)
DB_DATABASE	Database name
DB_USERNAME	Database user
DB_PASSWORD	Database user password

Important: Do not commit the .env file to version control, as it contains sensitive credentials.

If Laravel caches old configuration, clear it:
bash

php artisan config:clear

11. Running Migrations

Execute the database migrations:
bash

php artisan migrate

This creates the necessary tables for Laravel in the laravel_db database. A successful migration confirms that Laravel can connect to PostgreSQL.
12. Running Laravel

Start the development server:
bash

php artisan serve

The application will be available at:
text

http://127.0.0.1:8000

Stop the server with Ctrl + C.
13. Verification

Verify each component of the setup:

PHP version:
bash

php -v

Composer version:
bash

composer --version

PostgreSQL status:
bash

sudo systemctl status postgresql

PHP PostgreSQL driver:
bash

php -m | grep -E 'pgsql|pdo_pgsql'

Laravel version:
bash

php artisan --version

Migration status:
bash

php artisan migrate:status

If migrate:status runs without connection errors, Laravel is successfully communicating with PostgreSQL.
14. Troubleshooting
could not find driver

Install the PostgreSQL driver for PHP:
bash

sudo dnf install php-pgsql php-pdo_pgsql -y

Verify:
bash

php -m | grep -E 'pgsql|pdo_pgsql'

password authentication failed

Check the credentials in .env:
env

DB_USERNAME=laravel_user
DB_PASSWORD=your_password

Then clear the configuration cache:
bash

php artisan config:clear

connection refused

Check if PostgreSQL is running:
bash

sudo systemctl status postgresql

If stopped, start it:
bash

sudo systemctl start postgresql

Ensure .env uses:
env

DB_HOST=127.0.0.1
DB_PORT=5432

permission denied for schema public

Log in to PostgreSQL as administrator:
bash

sudo -u postgres psql

Connect to the database and grant schema permissions:
sql

\c laravel_db
GRANT USAGE, CREATE ON SCHEMA public TO laravel_user;
\q

Then retry migrations:
bash

php artisan migrate

15. Setup Flow
text

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

16. Security Notes

The configuration in this guide is suitable for development. For production, consider the following:
    Use strong, unique passwords.
    Never use the PostgreSQL administrator account for the application.
    Never commit the .env file to version control.
    Enable HTTPS.
    Restrict database access to trusted hosts.
    Implement regular database backups.
    Properly configure the web server and PHP-FPM.
    Match the PHP version to Laravel's requirements.
    Use a dedicated secret management solution.

17. Conclusion

Setting up Laravel with PostgreSQL on Fedora involves multiple layers:
    PHP runs Laravel.
    Composer manages dependencies.
    PostgreSQL stores data.
    The .env file connects Laravel to the database.

Once php artisan migrate succeeds and php artisan serve launches the application, your basic Laravel + PostgreSQL development environment is ready for further development.
