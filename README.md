# Inception

A Docker-based infrastructure project that sets up a complete web hosting environment with multiple services orchestrated using Docker Compose.

## 📋 Overview

This project implements a multi-container Docker infrastructure featuring:
- **NGINX** - Web server with TLS/SSL encryption
- **WordPress** - Content Management System with PHP-FPM
- **MariaDB** - Database server
- **Redis** - Caching layer for WordPress
- **FTP Server** - File transfer protocol service
- **Adminer** - Web-based database management interface
- **Grafana** - Monitoring and visualization platform
- **Static Website** - Custom static website service

All services run in isolated Docker containers and communicate through a custom bridge network.

## 🏗️ Architecture

```
                                    ┌─────────────┐
                                    │   Client    │
                                    └──────┬──────┘
                                           │
                    ┌──────────────────────┼──────────────────────┐
                    │                      │                      │
                    │                      │                      │
              ┌─────▼─────┐          ┌────▼────┐          ┌─────▼─────┐
              │   NGINX   │          │ Adminer │          │  Grafana  │
              │  (Port    │          │  (Port  │          │  (Port    │
              │   443)    │          │  8080)  │          │   3000)   │
              └─────┬─────┘          └────┬────┘          └─────┬─────┘
                    │                     │                      │
                    │            ┌────────┴──────────────────────┘
                    │            │
              ┌─────▼─────┐      │
              │ WordPress │◄─────┘
              │  + Redis  │
              └─────┬─────┘
                    │
              ┌─────▼─────┐
              │  MariaDB  │
              └───────────┘
```

## 🔧 Prerequisites

- Docker (version 20.10 or higher)
- Docker Compose (version 1.29 or higher)
- Make
- Linux/Unix system with sudo privileges
- At least 2GB of free disk space

## 📦 Installation

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd inception
   ```

2. **Create environment file:**
   Create a `.env` file in the `srcs/` directory with the following variables:
   ```bash
   # Database Configuration
   MYSQL_DATABASE=wordpress
   MYSQL_USER=wpuser
   MYSQL_PASSWORD=your_secure_password
   MYSQL_ROOT_PASSWORD=your_root_password

   # WordPress Configuration
   WORDPRESS_DB_HOST=mariadb
   WORDPRESS_DB_NAME=wordpress
   WORDPRESS_DB_USER=wpuser
   WORDPRESS_DB_PASSWORD=your_secure_password
   
   # Domain Configuration
   DOMAIN_NAME=achakkaf.42.fr
   
   # FTP Configuration (if using FTP service)
   FTP_USER=ftpuser
   FTP_PASSWORD=your_ftp_password
   ```

3. **Update domain references:**
   Ensure your domain name matches in:
   - `/etc/hosts` file (add: `127.0.0.1 yourdomain.42.fr`)
   - Docker Compose volume paths (update username in `docker-compose.yml`)
   - Makefile (update USER variable if needed)

## 🚀 Usage

### Building and Starting Services

```bash
# Build and start all services
make

# Or explicitly
make build
```

This command will:
- Create necessary data directories
- Build all Docker images
- Start all containers in detached mode

### Managing Services

```bash
# View logs from all containers
make logs

# Stop all services
make down

# Clean up containers and images
make clean

# Complete cleanup (removes data volumes)
make fclean

# Rebuild everything
make re
```

### Accessing Services

Once running, access the services at:

- **WordPress**: https://yourdomain.42.fr
- **Adminer**: http://localhost:8080
- **Grafana**: http://localhost:3000
- **Static Website**: http://localhost:80

## 🐳 Services Description

### Core Services

#### NGINX
- **Image**: Custom Debian-based
- **Ports**: 443 (HTTPS)
- **Features**: 
  - TLS/SSL encryption with self-signed certificates
  - Reverse proxy for WordPress
  - Static file serving

#### WordPress
- **Image**: Custom Debian-based with PHP-FPM
- **Features**:
  - WP-CLI for management
  - PHP 7.4
  - Redis object caching support
  - Automatic installation and configuration

#### MariaDB
- **Image**: Custom Debian-based
- **Features**:
  - Database persistence
  - Automatic database and user creation
  - Secure configuration

### Bonus Services

#### Redis
- **Purpose**: WordPress object caching
- **Benefit**: Improved performance and reduced database load

#### FTP Server (vsftpd)
- **Purpose**: File management for WordPress
- **Access**: Upload/download files to WordPress directory

#### Adminer
- **Purpose**: Web-based database management
- **Access**: Port 8080
- **Features**: Browse, edit, and manage MariaDB databases

#### Grafana
- **Purpose**: Monitoring and metrics visualization
- **Access**: Port 3000

#### Static Website
- **Purpose**: Custom static website hosting
- **Access**: Port 80

## 📁 Project Structure

```
inception/
├── Makefile                     # Build automation
└── srcs/
    ├── docker-compose.yml       # Service orchestration
    ├── .env                     # Environment variables (create this)
    └── requirements/
        ├── nginx/
        │   ├── Dockerfile
        │   └── conf/            # NGINX configuration
        ├── wordpress/
        │   ├── Dockerfile
        │   ├── conf/            # PHP-FPM configuration
        │   └── tools/           # Setup scripts
        ├── mariadb/
        │   ├── Dockerfile
        │   └── tools/           # Database setup scripts
        └── bonus/
            ├── redis/
            ├── FTP/
            ├── adminer/
            ├── grafana/
            └── website/
```

## 🔒 Security Notes

- All services use custom-built images (no pre-built images from DockerHub for core services)
- NGINX uses TLS 1.3 encryption
- Self-signed SSL certificates (for production, use proper certificates)
- Database credentials should be stored securely in `.env` file
- The `.env` file is not committed to version control

## 💾 Data Persistence

Data is persisted in the following locations:
- **WordPress files**: `/home/$USER/data/wordpress`
- **Database data**: `/home/$USER/data/database`

These directories are automatically created by the Makefile and mounted as Docker volumes.

## 🛠️ Troubleshooting

### Common Issues

1. **Port conflicts**: Ensure ports 80, 443, 3000, and 8080 are not in use
2. **Permission errors**: Run `make fclean` and `make build` with appropriate permissions
3. **Domain resolution**: Verify `/etc/hosts` contains your domain mapping
4. **Volume mounting**: Check that data directories exist and have correct permissions

### Debugging

```bash
# Check container status
docker ps -a

# View specific container logs
docker logs <container_name>

# Enter a container shell
docker exec -it <container_name> bash
```

## 📝 Notes

- This project follows 42 school requirements for the Inception project
- All containers restart automatically on failure
- Services depend on each other in the correct order (managed by Docker Compose)
- The project uses Debian Bullseye as the base image for all custom containers

## 🤝 Contributing

This is an educational project. Feel free to fork and modify for your own learning purposes.

## 📄 License

This project is created for educational purposes as part of the 42 school curriculum.
