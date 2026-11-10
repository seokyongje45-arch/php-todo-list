# Laravel Todo List Web Application

A clean, modern, responsive, and secure Todo List Web Application built with **PHP 8.3+**, **Laravel 13**, **MySQL 8.0 / MariaDB**, and **Bootstrap 5**.

---

## 1. Technology Stack & Key Highlights

- **Backend:** PHP 8.3+ & Laravel 13 (MVC architecture, Eloquent ORM, Form Requests, Policies)
- **Database:** MySQL 8.0 or MariaDB (Relational schema, foreign keys, cascade deletes, optimized indexes)
- **Frontend:** Laravel Blade templating engine
- **Styling:** Bootstrap 5.3 + Bootstrap Icons (Flat UI design, clean card typography, badges)
- **JavaScript:** Vanilla JavaScript (Theme switcher with `localStorage`, delete confirmation modal, auto-dismissing toasts)
- **Authentication:** Native Laravel session authentication with password hashing (`bcrypt`) and tenant-isolated data
- **Deployment:** Docker Compose with PHP-FPM, Nginx, and MySQL, or native Ubuntu Linux

---

## 2. Upgraded to Laravel 13

This project was cleanly upgraded from **Laravel 11 to Laravel 13**:

### What Changed in the Upgrade:
1. **Framework & Dependencies:**
   - `"php": "^8.3"` (drops PHP 8.2 requirement, full support for PHP 8.3 – 8.5)
   - `"laravel/framework": "^13.0"` (currently resolved to `v13.39.0`)
   - `"laravel/tinker": "^3.0"`
   - `"phpunit/phpunit": "^12.0"` (PHPUnit 12 migration with modern `<source>` coverage schema)
   - `"nunomaduro/collision": "^8.6"`
2. **Platform Pinning:**
   - Added `"config.platform.php": "8.3.0"` in `composer.json` to guarantee seamless dependency resolution across both host Ubuntu LTS environments and containerized PHP 8.3 FPM runners.
3. **PHPUnit 12 Test Suite Compatibility:**
   - Updated `phpunit.xml` to PHPUnit 12 standard structure with `<source>` code block and disabled legacy DocBlock annotation dependencies.
   - Verified 100% passing test suite (`13 tests, 43 assertions`) across authentication, task CRUD, and multi-tenant security isolation.
4. **Form Requests & Validation:**
   - Verified Form Request validation rules (`StoreTaskRequest`, `UpdateTaskRequest`) comply with Laravel 13's strict validation rules.

---

## 2. Core Features

### 🔐 Authentication & Authorization
- **User Registration:** Simple registration with validation (name, unique email, confirmed password min. 8 chars).
- **User Login & Logout:** Secure session management with "Remember Me" support.
- **Strict Tenant Isolation:** Each user can only view, create, edit, toggle, or delete their own tasks. Cross-user access is blocked via Controller authorization checks and Task Policies (HTTP 403 Forbidden).

### 📋 Task Management (CRUD)
- **Create Tasks:** Title (required), optional description, priority selection, initial status, and optional due date.
- **View Tasks:** Detailed view with status badge, priority pill, relative dates, and owner info.
- **Edit Tasks:** Form pre-populated with current values.
- **Delete Tasks:** Confirmation modal dialog prevents accidental deletions.
- **Quick Status Toggle:** Instant toggle between *Completed* and *Pending* from both Dashboard and Task table with 1 click.

### 📊 Dashboard & Analytics
- **5 Live Metric Cards:**
  - **Total Tasks:** Count of all user tasks.
  - **Pending Tasks:** Count of pending items.
  - **In Progress Tasks:** Active work items.
  - **Completed Tasks:** Finished tasks.
  - **Overdue Tasks:** Dynamic calculation (`due_date < today` and `status != 'completed'`).
- **Interactive Filters & Search:**
  - Search by task title or description.
  - Filter by status (`pending`, `in_progress`, `completed`).
  - Filter by priority (`low`, `medium`, `high`).
  - Sort by Creation Date, Due Date, Priority, or Title (Ascending / Descending).
  - Clean pagination with query parameter retention (`withQueryString()`).

### 🎨 Modern Flat UI & UX
- **Flat Design:** Clean borders, soft color accents, and subtle elevation on hover.
- **Dark Mode Switcher:** One-click toggle between light and dark themes using Bootstrap 5.3's `data-bs-theme`, persisted in `localStorage`.
- **Responsive Layout:** Desktop sidebar with active route highlights, collapsible mobile drawer with backdrop.
- **Toast Notifications:** Automatic feedback for actions (task created, updated, status toggled, deleted, login, logout).

---

## 3. Database Schema

### `tasks` Table Definition
| Field | Type | Attributes / Constraints |
|---|---|---|
| `id` | `BIGINT UNSIGNED` | Primary Key, Auto Increment |
| `user_id` | `BIGINT UNSIGNED` | Foreign Key `users(id)` `ON DELETE CASCADE` |
| `title` | `VARCHAR(255)` | Not Null |
| `description` | `TEXT` | Nullable |
| `priority` | `ENUM('low','medium','high')` | Default `'medium'` |
| `status` | `ENUM('pending','in_progress','completed')` | Default `'pending'` |
| `due_date` | `DATE` | Nullable |
| `created_at` | `TIMESTAMP` | Nullable |
| `updated_at` | `TIMESTAMP` | Nullable |

### Indexes
- `tasks_status_index`
- `tasks_priority_index`
- `tasks_due_date_index`
- `tasks_user_id_status_index` (Composite)
- `tasks_user_id_due_date_index` (Composite)

---

## 4. Docker Compose Production Deployment

The Docker setup uses three containers:

| Service | Image / Target | Public port |
|---|---|---|
| Laravel PHP-FPM | `php-todo-list:app` | Internal `9000` only |
| Nginx | `php-todo-list:nginx` | Host `8081` to container `80` |
| MySQL | `mysql:8.0` | Internal `3306` only |

Port `8081` is used because the target EC2 server already uses port `80` for
WordPress and port `8080` for phpMyAdmin.

### 4.1 Prepare the environment

```bash
git clone https://github.com/chanoudomchim/php-todo-list.git
cd php-todo-list

cp .env.docker.example .env.docker

# Create secure random application and database secrets.
APP_KEY_VALUE="$(openssl rand -base64 32 | tr -d '\n')"
DB_PASSWORD_VALUE="$(openssl rand -hex 24)"
ROOT_PASSWORD_VALUE="$(openssl rand -hex 24)"

sed -i "s|^APP_KEY=.*|APP_KEY=base64:${APP_KEY_VALUE}|" .env.docker
sed -i "s|CHANGE_TODO_USER_PASSWORD|${DB_PASSWORD_VALUE}|g" .env.docker
sed -i "s|CHANGE_MYSQL_ROOT_PASSWORD|${ROOT_PASSWORD_VALUE}|g" .env.docker
chmod 600 .env.docker
```

Do not commit or upload `.env.docker`; it contains production secrets.

### 4.2 Build and run on EC2

```bash
sudo docker compose --env-file .env.docker config --quiet
sudo docker compose --env-file .env.docker build
sudo docker compose --env-file .env.docker up -d
sudo docker compose --env-file .env.docker ps
```

Laravel migrations run automatically when the app container starts.
Production seeders are disabled by default. To inspect the services:

```bash
sudo docker compose --env-file .env.docker logs --tail=100 app nginx db
curl -I http://127.0.0.1:8081/health
```

### 4.3 Publish both production images to Docker Hub

This project stores the PHP-FPM and Nginx images in one Docker Hub repository
using separate `app` and `nginx` tags:

```bash
export DOCKERHUB_USERNAME=YOUR_DOCKERHUB_USERNAME
sudo docker login

sudo docker build --target app \
  -t "${DOCKERHUB_USERNAME}/php-todo-list:app" .

sudo docker build --target nginx \
  -t "${DOCKERHUB_USERNAME}/php-todo-list:nginx" .

sudo docker push "${DOCKERHUB_USERNAME}/php-todo-list:app"
sudo docker push "${DOCKERHUB_USERNAME}/php-todo-list:nginx"
```

Set `DOCKERHUB_USERNAME` in `.env.docker`, then deploy from Docker Hub:

```bash
sudo docker compose --env-file .env.docker -f compose.prod.yaml pull
sudo docker compose --env-file .env.docker -f compose.prod.yaml up -d
sudo docker compose --env-file .env.docker -f compose.prod.yaml ps
```

### 4.4 Make Laravel public in AWS

Add an EC2 Security Group inbound rule:

| Type | Port | Source | Purpose |
|---|---:|---|---|
| Custom TCP | `8081` | `0.0.0.0/0` | Public Laravel website |

Do **not** expose MySQL port `3306`. After the rule is saved, open:

`http://47.131.229.5:8081`

For a stable production address, attach an Elastic IP or use a domain name and
HTTPS reverse proxy instead of relying on the instance's temporary public IP.

---

## 5. Ubuntu Linux Native Installation Guide

This application is designed to run directly on an **Ubuntu Linux (22.04 LTS / 24.04 LTS / 26.04 LTS)** server without Docker.

### Step 1: Update System & Install PHP 8.3

```bash
sudo apt update && sudo apt upgrade -y

# Add Ondřej Surý PHP repository (if on Ubuntu LTS)
sudo apt install -y software-properties-common curl git unzip
sudo add-apt-repository -y ppa:ondrej/php
sudo apt update

# Install PHP 8.3 and required extensions
sudo apt install -y php8.3 php8.3-cli php8.3-fpm php8.3-mysql \
                    php8.3-xml php8.3-mbstring php8.3-curl \
                    php8.3-zip php8.3-bcmath php8.3-intl
```

Verify PHP installation:
```bash
php -v
```

---

### Step 2: Install Composer

```bash
# Download and install Composer globally
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"

# Verify Composer version
composer -V
```

---

### Step 3: Install & Configure MySQL 8.0 / MariaDB

```bash
sudo apt install -y mysql-server

# Secure installation
sudo mysql_secure_installation
```

Log in to MySQL as root and create database and user:
```bash
sudo mysql
```

Run the following SQL statements:
```sql
CREATE DATABASE IF NOT EXISTS todo_app CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER IF NOT EXISTS 'todo_user'@'localhost' IDENTIFIED BY 'YourStrongPassword123!';

GRANT ALL PRIVILEGES ON todo_app.* TO 'todo_user'@'localhost';

FLUSH PRIVILEGES;
EXIT;
```

---

### Step 4: Clone / Set Up Project & Install Dependencies

```bash
# Navigate to web root directory
cd /var/www

# Clone repository or copy project
sudo git clone https://github.com/your-username/php-todo-list.git todo-app
cd /var/www/todo-app

# Install Composer dependencies
composer install --no-dev --optimize-autoloader
```

---

### Step 5: Configure Environment (`.env`)

```bash
# Copy example configuration
cp .env.example .env

# Generate application key
php artisan key:generate
```

Edit `.env` using your preferred editor:
```bash
nano .env
```

Ensure the database credentials match Step 3:
```dotenv
APP_NAME="TodoApp"
APP_ENV=production
APP_DEBUG=false
APP_URL=http://your-server-domain-or-ip

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=todo_app
DB_USERNAME=todo_user
DB_PASSWORD=YourStrongPassword123!

SESSION_DRIVER=database
CACHE_STORE=database
```

---

### Step 6: Run Database Migrations & Seeders

```bash
# Run migrations to create tables
php artisan migrate --force

# Seed database with sample users and realistic tasks
php artisan db:seed --force
```

#### Pre-seeded Demo Accounts
| Email | Password | Role / Details |
|---|---|---|
| `user@example.com` | `password` | Standard user (preloaded with active and overdue tasks) |
| `admin@example.com` | `password` | Administrator account (isolated tasks) |

---

### Step 7: Configure Storage Permissions

The web server user (`www-data`) must have write permissions to `storage` and `bootstrap/cache`:

```bash
sudo chown -R www-data:www-data /var/www/todo-app/storage /var/www/todo-app/bootstrap/cache
sudo chmod -R 775 /var/www/todo-app/storage /var/www/todo-app/bootstrap/cache
```

---

### Step 8: Web Server Configuration

Choose either **Nginx** or **Apache2**:

#### Option A: Nginx (Recommended)

1. Install Nginx:
   ```bash
   sudo apt install -y nginx
   ```

2. Create virtual host file `/etc/nginx/sites-available/todo-app`:
   ```nginx
   server {
       listen 80;
       server_name your-server-domain-or-ip;
       root /var/www/todo-app/public;

       add_header X-Frame-Options "SAMEORIGIN";
       add_header X-Content-Type-Options "nosniff";

       index index.php;

       charset utf-8;

       location / {
           try_files $uri $uri/ /index.php?$query_string;
       }

       location = /favicon.ico { access_log off; log_not_found off; }
       location = /robots.txt  { access_log off; log_not_found off; }

       error_page 404 /index.php;

       location ~ \.php$ {
           fastcgi_pass unix:/run/php/php8.3-fpm.sock;
           fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
           include fastcgi_params;
           fastcgi_hide_header X-Powered-By;
       }

       location ~ /\.(?!well-known).* {
           deny all;
       }
   }
   ```

3. Enable the site and restart Nginx:
   ```bash
   sudo ln -s /etc/nginx/sites-available/todo-app /etc/nginx/sites-enabled/
   sudo rm -f /etc/nginx/sites-enabled/default
   sudo nginx -t
   sudo systemctl restart php8.3-fpm
   sudo systemctl restart nginx
   ```

---

#### Option B: Apache2

1. Install Apache2 and PHP module:
   ```bash
   sudo apt install -y apache2 libapache2-mod-php8.3
   sudo a2enmod rewrite
   ```

2. Create virtual host file `/etc/apache2/sites-available/todo-app.conf`:
   ```apache
   <VirtualHost *:80>
       ServerName your-server-domain-or-ip
       DocumentRoot /var/www/todo-app/public

       <Directory /var/www/todo-app/public>
           Options -Indexes +FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>

       ErrorLog ${APACHE_LOG_DIR}/todo_error.log
       CustomLog ${APACHE_LOG_DIR}/todo_access.log combined
   </VirtualHost>
   ```

3. Enable site and restart Apache:
   ```bash
   sudo a2ensite todo-app.conf
   sudo a2dissite 000-default.conf
   sudo apache2ctl configtest
   sudo systemctl restart apache2
   ```

---

### Step 9: Production Optimization & Caching

Run Laravel's built-in caching commands to maximize performance:

```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

---

## 6. Local Development Quickstart

To run the application locally for quick testing using PHP's built-in server:

```bash
# 1. Install dependencies
composer install

# 2. Setup .env
cp .env.example .env
php artisan key:generate

# 3. If testing with SQLite:
touch database/database.sqlite
# Set DB_CONNECTION=sqlite in .env

# 4. Migrate and Seed
php artisan migrate --seed

# 5. Start dev server
php artisan serve
```

Visit `http://localhost:8000` in your browser.

---

## 7. Running Automated Tests

Run the test suite with PHPUnit:

```bash
vendor/bin/phpunit
```

Test coverage includes:
- Authentication workflows (Render login, Register user, Login, Validation failure, Logout)
- Task CRUD operations (Create, View, Update, Toggle status, Delete)
- Security & Tenant isolation (Verifying User A cannot read, edit, or delete User B's tasks)

---

## 8. Security Best Practices Implemented

1. **CSRF Protection:** All non-GET forms include `@csrf` token validation.
2. **SQL Injection Defense:** All database queries utilize Eloquent ORM and prepared PDO statements.
3. **Cross-Site Scripting (XSS) Prevention:** Blade escaping syntax (`{{ }}`) is used for all dynamic output.
4. **Tenant Isolation:** Controller authorization guards check `user_id === auth()->id()`.
5. **Secure Password Hashing:** User passwords hashed using Bcrypt via `Hash::make()`.
6. **Input Validation:** Form Requests validate required fields, max string lengths, and enum values.

---

## 9. License

Open-sourced software licensed under the [MIT License](LICENSE).
