# Laravel + PostgreSQL Setup on Fedora Linux

A comprehensive guide to installing and configuring Laravel with PostgreSQL on Fedora Linux. This document covers the full setup process from system preparation to running your first migration and development server.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [System Update](#system-update)
3. [PHP Installation](#php-installation)
4. [Composer Installation](#composer-installation)
5. [PostgreSQL Installation](#postgresql-installation)
6. [Database Configuration](#database-configuration)
7. [pg_hba.conf Configuration](#pghbaconf-configuration)
8. [PHP PostgreSQL Extension](#php-postgresql-extension)
9. [Creating a Laravel Project](#creating-a-laravel-project)
10. [Environment Configuration](#environment-configuration)
11. [Running Migrations](#running-migrations)
12. [Starting the Development Server](#starting-the-development-server)
13. [Verification](#verification)
14. [Troubleshooting](#troubleshooting)
15. [Security Notes](#security-notes)
16. [Setup Flowchart](#setup-flowchart)
17. [Conclusion](#conclusion)

---

## Prerequisites

Before beginning, ensure you have the following:

- Fedora Linux (any recent version)
- A user account with `sudo` privileges
- Active internet connection
- Terminal access

**Important:** This guide is intended for **development environments only**. Do not use example passwords such as `123` in production.

---

## System Update

Update all system packages to their latest versions:

```bash
sudo dnf update -y

This ensures compatibility and security for the installations that follow.
PHP Installation

Laravel requires PHP with specific extensions. We will use the Remi repository to obtain PHP 8.4.

Enable the Remi module:
bash

sudo dnf module reset php
sudo dnf module enable php:remi-8.4 -y

Install PHP and commonly required extensions:
bash

sudo dnf install php php-cli php-fpm php-zip php-devel php-gd php-mbstring php-curl php-xml php-pear php-bcmath php-json php-opcache php-intl -y

Verify the PHP version:
bash

php -v

Ensure the version matches the requirements of your Laravel version.
Composer Installation

Composer is PHP's dependency manager and is essential for Laravel.

Download and install Composer:
bash

php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php
sudo mv composer.phar /usr/local/bin/composer

Verify the installation:
bash

composer --version

PostgreSQL Installation

Install PostgreSQL database server and additional utilities.

Install PostgreSQL packages:
bash

sudo dnf install postgresql postgresql-server postgresql-contrib -y

Initialize the database cluster:
bash

sudo postgresql-setup --initdb

Start and enable PostgreSQL to run on boot:
bash

sudo systemctl enable --now postgresql

Check the service status:
bash

sudo systemctl status postgresql

The status should show active (running).
Database Configuration

Now we will configure PostgreSQL and create a dedicated database and user for Laravel.

Access the PostgreSQL prompt as the administrator:
bash

sudo -u postgres psql

Set a password for the postgres administrator:
sql

\password postgres

Create a database for Laravel:
sql

CREATE DATABASE laravel_db;

Create a dedicated application user:
sql

CREATE USER laravel_user WITH PASSWORD 'YOUR_SECURE_PASSWORD';

Grant all privileges on the database:
sql

GRANT ALL PRIVILEGES ON DATABASE laravel_db TO laravel_user;

Grant schema permissions for migrations:
sql

\c laravel_db
GRANT USAGE, CREATE ON SCHEMA public TO laravel_user;

Exit PostgreSQL:
sql

\q

Why create a dedicated user? Using a non-administrator account reduces security risks and follows the principle of least privilege.
pg_hba.conf Configuration

This file defines client authentication methods for PostgreSQL.

Open the configuration file:
bash

sudo nano /var/lib/pgsql/data/pg_hba.conf

For modern PostgreSQL installations, ensure the following lines are present:
text

local   all   all                         scram-sha-256
host    all   all   127.0.0.1/32         scram-sha-256
host    all   all   ::1/128               scram-sha-256

Important: Use scram-sha-256 as it is the recommended authentication method for modern PostgreSQL. Avoid blindly copying older tutorials that use md5.

Restart PostgreSQL to apply changes:
bash

sudo systemctl restart postgresql

PHP PostgreSQL Extension

Install the PostgreSQL driver for PHP to enable database connectivity.

Install the required packages:
bash

sudo dnf install php-pgsql php-pdo_pgsql -y

Restart PHP-FPM:
bash

sudo systemctl restart php-fpm

Verify the extensions are loaded:
bash

php -m | grep -E 'pgsql|pdo_pgsql'

Expected output should include:
text

pdo_pgsql
pgsql

Creating a Laravel Project

Create a new Laravel project using Composer:
bash

composer create-project laravel/laravel belajar-laravel

Navigate into the project directory:
bash

cd belajar-laravel

Create the environment file if it does not exist:
bash

cp .env.example .env

Generate the application key:
bash

php artisan key:generate

Note: The APP_KEY is used for encryption and security. Do not share it or commit it to version control.
Environment Configuration

The .env file stores environment-specific settings, including database credentials.

Open the environment file:
bash

nano .env

Configure the database connection settings:
text

DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=YOUR_SECURE_PASSWORD

Variable	Description
DB_CONNECTION	Database driver (pgsql)
DB_HOST	PostgreSQL server address
DB_PORT	PostgreSQL port (default: 5432)
DB_DATABASE	Database name
DB_USERNAME	Database user
DB_PASSWORD	Database user password

Important: Never commit .env to version control as it contains sensitive credentials.

Clear cached configuration if Laravel uses old settings:
bash

php artisan config:clear

Running Migrations

Migrations create and modify database tables.

Run the Laravel migrations:
bash

php artisan migrate

This command will create all default Laravel tables in the laravel_db database.

If successful, Laravel has successfully connected to PostgreSQL.
Starting the Development Server

Start the built-in Laravel development server:
bash

php artisan serve

Access the application at:
text

http://127.0.0.1:8000

Stop the server: Press Ctrl + C.
Verification

Run these commands to confirm each component is correctly installed:
Component	Command	Purpose
PHP	php -v	Check PHP version
Composer	composer --version	Check Composer version
PostgreSQL	sudo systemctl status postgresql	Check PostgreSQL service status
PHP Driver	php -m | grep -E 'pgsql|pdo_pgsql'	Verify PostgreSQL extensions
Laravel	php artisan --version	Check Laravel version
Migration	php artisan migrate:status	Check migration status

If migrate:status runs without connection errors, Laravel is successfully communicating with PostgreSQL.
Troubleshooting

Error: could not find driver

Install the PostgreSQL PHP extensions:
bash

sudo dnf install php-pgsql php-pdo_pgsql -y

Verify with:
bash

php -m | grep -E 'pgsql|pdo_pgsql'

Error: password authentication failed

    Verify the DB_USERNAME and DB_PASSWORD in .env are correct.

    Clear the configuration cache:

bash

php artisan config:clear

Error: connection refused

Check if PostgreSQL is running:
bash

sudo systemctl status postgresql

If stopped, start it:
bash

sudo systemctl start postgresql

Ensure .env uses:
text

DB_HOST=127.0.0.1
DB_PORT=5432

Error: permission denied for schema public

Grant schema permissions to the user:
bash

sudo -u postgres psql

Then execute:
sql

\c laravel_db
GRANT USAGE, CREATE ON SCHEMA public TO laravel_user;
\q

Run migrations again:
bash

php artisan migrate

Security Notes

The configuration provided is suitable for development. For production environments, consider the following:

    Use strong, unique passwords for all database users.

    Never use the PostgreSQL administrator account for application connections.

    Never commit .env files to version control.

    Enable HTTPS for all production traffic.

    Restrict database access to specific IP addresses.

    Implement regular database backups.

    Configure web server and PHP-FPM with production-grade settings.

    Match PHP and Laravel versions according to official compatibility.

    Use a secure secret management solution for sensitive keys.

Setup Flowchart
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
.env Configuration
    ↓
php artisan migrate
    ↓
php artisan serve

Conclusion

Setting up Laravel with PostgreSQL on Fedora Linux involves multiple interdependent components:

    PHP runs the Laravel application.

    Composer manages dependencies.

    PostgreSQL stores application data.

    The .env file connects Laravel to the database.

Once php artisan migrate executes successfully and php artisan serve starts the development server, your Laravel + PostgreSQL environment is fully operational and ready for development.
Author

Firdaus Ramdan
Software Engineering Student at SMKN 8 Jakarta
Email: mrfirdausramdan@gmail.com
Learning Focus: Front-end development, problem-solving, computational thinking
