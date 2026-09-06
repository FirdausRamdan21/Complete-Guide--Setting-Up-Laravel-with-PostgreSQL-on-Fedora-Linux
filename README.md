
## 🐘 What is PostgreSQL?

**PostgreSQL** (Postgres) is a powerful, open-source relational database management system (RDBMS).

### Why Choose PostgreSQL?

| Feature | PostgreSQL | MySQL |
| --- | --- | --- |
| **SQL Compliance** | ✅ Full standard | ⚠️ Partial |
| **JSON Support** | ✅ JSONB (faster) | ✅ JSON |
| **Full-Text Search** | ✅ Advanced | ⚠️ Basic |
| **ACID Compliance** | ✅ Strict | ⚠️ Variable |
| **Concurrency** | ✅ MVCC | ✅ InnoDB |
| **Geospatial Data** | ✅ PostGIS | ✅ MySQL Spatial |
| **Enterprise Features** | ✅ Rich | ⚠️ Limited |

### PostgreSQL Advantages:

* 📊 **Better data integrity**
* 🔍 **Advanced query capabilities**
* 🏢 **Used by major companies** (Apple, Uber, Instagram)
* 🔒 **Strong security features**

---

## 🚀 Step-by-Step Installation

### Step 1: Setup PHP

First, let's install PHP 8.4 with essential extensions.

```bash
# Update system packages
sudo dnf update -y

# Reset and enable PHP 8.4 from Remi repository
sudo dnf module reset php
sudo dnf module enable php:remi-8.4 -y

# Install PHP and common extensions
sudo dnf install php php-cli php-fpm php-zip php-devel \
    php-gd php-mbstring php-curl php-xml php-pear \
    php-bcmath php-json php-opcache php-intl -y

# Verify PHP installation
php -v

```

**Expected output:**

```text
PHP 8.4.x (cli) (built: ... )
Copyright (c) The PHP Group
Zend Engine v4.x.x

```

---

### Step 2: Setup Composer

Composer is the dependency manager for PHP (like npm for Node.js or pip for Python).

```bash
# Download Composer installer
php -r "copy('[https://getcomposer.org/installer](https://getcomposer.org/installer)', 'composer-setup.php');"

# Verify installer signature (security check)
php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

# Install Composer
php composer-setup.php

# Remove installer
php -r "unlink('composer-setup.php');"

# Move Composer to global path
sudo mv composer.phar /usr/local/bin/composer

# Verify installation
composer --version

```

**Expected output:**

```text
Composer version 2.7.x

```

---

### Step 3: Install PostgreSQL

```bash
# Install PostgreSQL and required packages
sudo dnf install postgresql postgresql-server postgresql-contrib -y

# Initialize the database cluster (first time only)
sudo postgresql-setup --initdb

# Start PostgreSQL service
sudo systemctl start postgresql

# Enable PostgreSQL to start automatically on boot
sudo systemctl enable postgresql

# Check PostgreSQL status
sudo systemctl status postgresql

```

**Expected output:**

```text
● postgresql.service - PostgreSQL database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled)
   Active: active (running) since ...

```

---

### Step 4: Configure PostgreSQL

#### 4.1 Set Password for PostgreSQL Admin

```bash
# Switch to postgres user
sudo -u postgres psql

# Inside PostgreSQL shell, change password
\password postgres

# Enter new password: 123 (type it twice)

# Exit PostgreSQL shell
\q

```

#### 4.2 Create Database and User

```bash
# Login as postgres user
sudo -u postgres psql

# Create a new database
CREATE DATABASE laravel_db;

# Create a new user
CREATE USER laravel_user WITH PASSWORD '123';

# Grant all privileges on the database to the user
GRANT ALL PRIVILEGES ON DATABASE laravel_db TO laravel_user;

# Exit PostgreSQL
\q

```

#### 4.3 Configure Authentication Method

```bash
# Edit PostgreSQL authentication configuration
sudo nano /var/lib/pgsql/data/pg_hba.conf

```

Find and modify the following lines (change `ident` or `peer` to `md5`):

```conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     md5

# IPv4 local connections:
host    all             all             127.0.0.1/32            md5

# IPv6 local connections:
host    all             all             ::1/128                 md5

```

**Save and exit:** Press `Ctrl+O`, then `Enter`, then `Ctrl+X`.

#### 4.4 Restart PostgreSQL

```bash
# Restart PostgreSQL to apply changes
sudo systemctl restart postgresql

# Verify PostgreSQL is running
sudo systemctl status postgresql

```

---

### Step 5: Install PHP PostgreSQL Extensions

Laravel needs PostgreSQL extensions to communicate with the database.

```bash
# Install PostgreSQL extensions for PHP
sudo dnf install php-pgsql php-pdo_pgsql -y

# Restart PHP-FPM service
sudo systemctl restart php-fpm

# Verify pgsql extension is installed
php -m | grep pgsql

# Verify pdo_pgsql extension is installed
php -m | grep pdo_pgsql

```

**Expected output:**

```text
pgsql
pdo_pgsql

```

---

### Step 6: Create Laravel Project

```bash
# Create a new Laravel project
composer create-project laravel/laravel belajar-laravel

# Navigate to the project directory
cd belajar-laravel

# Generate application key
php artisan key:generate

# Start Laravel development server
php artisan serve

```

**Open your browser and visit:** `http://localhost:8000`

You should see the Laravel welcome page!

---

### Step 7: Configure Environment

#### 7.1 Edit `.env` File

```bash
# Open .env file
nano .env

```

Remove the `#` (comment) from database configuration lines and update with PostgreSQL settings:

```env
# Database Configuration
DB_CONNECTION=pgsql          # Remove # and change to pgsql
DB_HOST=127.0.0.1            # Remove # 
DB_PORT=5432                 # Remove # (PostgreSQL default port)
DB_DATABASE=laravel_db       # Remove # - database name we created
DB_USERNAME=postgres         # Remove # - or use laravel_user
DB_PASSWORD=123              # Remove # - password set earlier

```

**Save and exit:** Press `Ctrl+O`, then `Enter`, then `Ctrl+X`.

#### 7.2 Test Database Connection

```bash
# Run migrations to test connection
php artisan migrate

```

**Expected output:**

```text
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table (0.05 seconds)
Migrating: 2014_10_12_100000_create_password_resets_table
Migrated:  2014_10_12_100000_create_password_resets_table (0.03 seconds)
...

```

---

## 🔧 Troubleshooting

### Common Issues and Solutions

#### Issue 1: "Connection refused" or "could not connect to server"

```bash
# Check if PostgreSQL is running
sudo systemctl status postgresql

# If not running, start it
sudo systemctl start postgresql

# Check port availability
sudo netstat -tulpn | grep 5432

```

#### Issue 2: "password authentication failed"

If you forget or need to change the PostgreSQL password:

```bash
# Change password for postgres user
sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD '123';"

# Change password for laravel_user (if used)
sudo -u postgres psql -c "ALTER USER laravel_user WITH PASSWORD '123';"

# Update .env file with the new password
nano .env

```

#### Issue 3: "SQLSTATE[08006] [7] FATAL: no pg_hba.conf entry"

Edit the `pg_hba.conf` file:

```bash
sudo nano /var/lib/pgsql/data/pg_hba.conf

```

Make sure these lines exist and are uncommented:

```conf
local   all             all                                     md5
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5

```

Then restart PostgreSQL:

```bash
sudo systemctl restart postgresql

```

#### Issue 4: Extension not found (pdo_pgsql)

```bash
# Reinstall extensions
sudo dnf reinstall php-pdo_pgsql php-pgsql

# Restart web server
sudo systemctl restart php-fpm

# Verify again
php -m | grep pgsql
php -m | grep pdo_pgsql

```

#### Issue 5: "Database laravel_db does not exist"

```bash
# Create the database
sudo -u postgres psql -c "CREATE DATABASE laravel_db;"

# Verify it was created
sudo -u postgres psql -c "\l" | grep laravel_db

```

#### Issue 6: Permission issues with vendor folder

```bash
# Fix permissions
sudo chown -R $USER:$USER .
chmod -R 755 storage bootstrap/cache

```

---

## 📝 Quick Reference Commands

| Task | Command |
| --- | --- |
| **Check PHP Version** | `php -v` |
| **Check Composer Version** | `composer --version` |
| **Check PostgreSQL Status** | `sudo systemctl status postgresql` |
| **Start PostgreSQL** | `sudo systemctl start postgresql` |
| **Stop PostgreSQL** | `sudo systemctl stop postgresql` |
| **Restart PostgreSQL** | `sudo systemctl restart postgresql` |
| **Login to PostgreSQL** | `sudo -u postgres psql` |
| **List Databases** | `sudo -u postgres psql -c "\l"` |
| **Create Database** | `sudo -u postgres psql -c "CREATE DATABASE dbname;"` |
| **Change Password** | `sudo -u postgres psql -c "ALTER USER username WITH PASSWORD 'newpass';"` |
| **Check PHP Extensions** | `php -m | grep pgsql` |
| **Clear Laravel Cache** | `php artisan optimize:clear` |
| **Run Migrations** | `php artisan migrate` |
| **Rollback Migrations** | `php artisan migrate:rollback` |
| **Start Dev Server** | `php artisan serve` |

---

## ✅ Verification Checklist

* [ ] PHP 8.4+ installed (`php -v`)
* [ ] Composer installed (`composer --version`)
* [ ] PostgreSQL installed and running (`sudo systemctl status postgresql`)
* [ ] Database `laravel_db` created
* [ ] PostgreSQL user configured with password
* [ ] `pg_hba.conf` configured with `md5` method
* [ ] `php-pgsql` and `php-pdo_pgsql` extensions installed
* [ ] `.env` file configured with correct PostgreSQL settings
* [ ] Laravel project created successfully
* [ ] `php artisan migrate` runs without errors
* [ ] Application can be accessed at `http://localhost:8000`

---

## 📚 Next Steps

Now that your environment is set up, you can:

1. **Build a CRUD application** - Create, Read, Update, Delete operations
2. **Learn Eloquent ORM** - Laravel's powerful database layer
3. **Explore PostgreSQL-specific features** - JSONB, Full-Text Search, etc.
4. **Implement authentication** - Using Laravel Breeze or Jetstream
5. **Add security measures** - CSRF protection, input validation, SQL injection prevention

---

## 🤝 Contributing

Found a bug or want to improve this guide? Feel free to:

1. Fork the repository
2. Create a new branch (`git checkout -b improve-guide`)
3. Make your changes
4. Submit a pull request

---

## 📄 License

This documentation is open-source and available under the [MIT License](https://www.google.com/search?q=LICENSE).

---

## 🙏 Acknowledgments

* Laravel Framework - PHP Web Framework
* PostgreSQL - Open-Source Database
* Fedora Project - Linux Distribution

---

**Happy Coding! 🚀**

```

```