# wordpress_template
This project sets up a **WordPress** web-instance using Docker Compose with three main services:

---

## 📦 Services Overview

### 1. **WordPress**
- **Image**: `wordpress:latest`
- **Port**: Exposed via Caddy
- **Volumes**: Persists WordPress content in the `wordpress` volume
- **Environment**:
  - Reads database credentials from `.env`
- **Networks**:
  - `frontend`: connects to Caddy
  - `backend`: connects to MySQL

### 2. **Caddy (Web Server & Reverse Proxy)**
- **Image**: `caddy:2.10`
- **Port Mapping**: `443 → 8081` on host (access via `https://wp.local:8081`)
- **TLS**: Uses `tls internal` or Let's Encrypt depending on configuration
- **Config**: 
  - Loaded from a generated `Caddyfile` in the `conf/` directory
- **Features**:
  - Automatic HTTPS (for real domains) or internal TLS for development
  - Gzip compression
- **Volumes**:
  - Persists TLS and configuration data via `caddy_data` and `caddy_config`
- **Security**:
  - Runs as read-only
  - Uses `tmpfs` for temporary and cert-related directories
  - Requires `NET_ADMIN` capability (needed for Caddy’s network config)

### 3. **MySQL (Database)**
- **Image**: `mysql:5.7`
- **Volumes**: Data persisted in `db_data`
- **Environment**:
  - Fully controlled via `.env` (e.g., DB name, user, root password)
- **Networks**:
  - `backend`: communicates only with WordPress

---

## ⚙️ Configuration

### `.env` File

Before running the stack, create a `.env` file in the root directory with:

```env
HOSTNAME=wp.local
EMAIL=hostmaster@fenker.eu

DB_NAME=wordpress
DB_USER=wordpress
DB_PASSWORD=your_db_password
DB_ROOT_PASSWORD=your_root_password
```

### Run the Stack

To start the services, run:

```bash
./run.sh
```

The caddyfile needs to be rendered with the .env variables, which is done in the `run.sh` script.

## A Quick Guide on WordPress

User Login URL: `https://<hostname>/wp-login.php`