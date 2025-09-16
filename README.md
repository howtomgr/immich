# Immich Installation Guide

Immich is a free and open-source self-hosted photo and video management solution. It serves as a FOSS alternative to cloud-based photo services like Google Photos, Apple iCloud Photos, Amazon Photos, Dropbox Photos, or OneDrive Photos. Immich provides automatic backup from mobile devices, facial recognition, object detection, and advanced search capabilities while keeping your photos under your complete control and ensuring privacy.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

### Hardware Requirements
- **CPU**: 4+ cores (8+ recommended for ML features)
- **RAM**: 4GB minimum (8GB+ recommended)
- **Storage**: 100GB+ for photos/videos (plan for growth)
- **GPU**: Optional but recommended for machine learning acceleration

### Software Requirements
- **Docker**: 24.0+ and Docker Compose
- **PostgreSQL**: 14+ (included in Docker setup)
- **Redis**: 6.2+ (included in Docker setup)
- **Node.js**: 18+ (for native installation)

### Network Requirements
- **Ports**: 
  - 2283: Web interface (HTTP)
  - 2284: Machine learning service
- **Mobile Access**: Internet connectivity for mobile app sync
- **Storage**: Network-attached storage support (NFS, SMB)

## 2. Supported Operating Systems

Immich officially supports:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04 LTS / 22.04 LTS / 24.04 LTS
- Arch Linux
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- macOS 12+ (Intel and Apple Silicon)
- Windows 10/11 (via WSL2 or Docker Desktop)

## 3. Installation

### Method 1: Docker Compose (Recommended)

#### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install Docker and Docker Compose
sudo dnf install -y docker docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker $USER

# Create Immich directory
mkdir -p ~/immich
cd ~/immich

# Download docker-compose.yml
curl -L https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml -o docker-compose.yml
curl -L https://github.com/immich-app/immich/releases/latest/download/example.env -o .env

# Edit environment variables
cp .env immich.env
nano immich.env

# Set required variables in immich.env:
# DB_PASSWORD=your_secure_database_password
# JWT_SECRET=$(openssl rand -base64 32)
# UPLOAD_LOCATION=/path/to/photos

# Create upload directory
sudo mkdir -p /var/lib/immich/upload
sudo chown -R $USER:$USER /var/lib/immich

# Start Immich
docker-compose up -d

# Check status
docker-compose ps
```

#### Debian/Ubuntu

```bash
# Update system
sudo apt update

# Install Docker
sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker $USER

# Log out and back in, or run:
newgrp docker

# Create Immich directory
mkdir -p ~/immich
cd ~/immich

# Download configuration files
wget https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget https://github.com/immich-app/immich/releases/latest/download/example.env -O .env

# Configure environment
cp .env immich.env
editor immich.env

# Generate secure passwords
DB_PASSWORD=$(openssl rand -base64 32)
JWT_SECRET=$(openssl rand -base64 32)

# Update immich.env with generated values
sed -i "s/DB_PASSWORD=postgres/DB_PASSWORD=$DB_PASSWORD/" immich.env
sed -i "s/# JWT_SECRET=CHANGE_ME_TO_A_RANDOM_PASSPHRASE/JWT_SECRET=$JWT_SECRET/" immich.env

# Set upload location
echo "UPLOAD_LOCATION=/var/lib/immich/upload" >> immich.env

# Create directories
sudo mkdir -p /var/lib/immich/upload
sudo chown -R $USER:$USER /var/lib/immich

# Start services
docker-compose --env-file immich.env up -d

# Verify installation
docker-compose logs immich_server
```

#### Arch Linux

```bash
# Install Docker
sudo pacman -S docker docker-compose
sudo systemctl enable --now docker
sudo usermod -aG docker $USER

# Create Immich setup
mkdir -p ~/immich && cd ~/immich

# Download files
curl -L https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml -o docker-compose.yml
curl -L https://github.com/immich-app/immich/releases/latest/download/example.env -o immich.env

# Configure environment
vim immich.env

# Generate secrets
openssl rand -base64 32  # Use for DB_PASSWORD
openssl rand -base64 32  # Use for JWT_SECRET

# Set upload location
mkdir -p /home/$USER/immich-photos
echo "UPLOAD_LOCATION=/home/$USER/immich-photos" >> immich.env

# Start Immich
docker-compose --env-file immich.env up -d
```

#### Alpine Linux

```bash
# Install Docker
apk add --no-cache docker docker-compose
rc-service docker start
rc-update add docker default
addgroup $USER docker

# Create Immich directory
mkdir -p /opt/immich && cd /opt/immich

# Download configuration
wget https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget https://github.com/immich-app/immich/releases/latest/download/example.env -O immich.env

# Configure environment
vi immich.env

# Create upload directory
mkdir -p /var/lib/immich/upload
chown -R 1000:1000 /var/lib/immich

# Start services
docker-compose --env-file immich.env up -d
```

### Method 2: Native Installation (Advanced)

#### Ubuntu/Debian Native Setup

```bash
# Install dependencies
sudo apt update
sudo apt install -y nodejs npm postgresql redis-server nginx certbot python3-certbot-nginx

# Install Node.js 18+
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Create immich user
sudo useradd -r -s /bin/false -d /opt/immich immich

# Setup PostgreSQL
sudo -u postgres createuser -P immich
sudo -u postgres createdb -O immich immich

# Setup Redis
sudo systemctl enable --now redis-server

# Clone Immich
sudo git clone https://github.com/immich-app/immich.git /opt/immich
sudo chown -R immich:immich /opt/immich

# Build Immich
cd /opt/immich
sudo -u immich npm ci --workspaces
sudo -u immich npm run build

# Configure environment
sudo -u immich cp .env.example .env
sudo -u immich nano .env

# Install and configure systemd services
# (Complex native setup - Docker recommended)
```

## 4. Configuration

### Environment Configuration

Edit `immich.env` file:
```env
# Database
DB_HOSTNAME=immich_postgres
DB_USERNAME=postgres
DB_PASSWORD=your_secure_password
DB_DATABASE_NAME=immich

# Redis
REDIS_HOSTNAME=immich_redis

# Upload Location
UPLOAD_LOCATION=/var/lib/immich/upload

# Security
JWT_SECRET=your_jwt_secret_here

# Machine Learning
IMMICH_MACHINE_LEARNING_ENABLED=true
IMMICH_MACHINE_LEARNING_URL=http://immich_machine_learning:3003

# Server
IMMICH_SERVER_URL=http://localhost:2283
IMMICH_WEB_URL=http://localhost:2283
```

### Docker Compose Customization

Create custom `docker-compose.override.yml`:
```yaml
version: "3.8"

services:
  immich-server:
    ports:
      - "2283:3001"
    volumes:
      - /mnt/photos:/usr/src/app/upload/photos
      - /mnt/videos:/usr/src/app/upload/videos
    environment:
      - TZ=America/New_York

  immich-postgres:
    volumes:
      - /var/lib/postgresql/data:/var/lib/postgresql/data

  immich-redis:
    volumes:
      - /var/lib/redis:/data
```

### Reverse Proxy Setup (nginx)

```nginx
server {
    listen 80;
    server_name photos.example.com;
    
    location / {
        proxy_pass http://localhost:2283;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 5. Service Management

### Docker Compose Management

```bash
# Start services
docker-compose --env-file immich.env up -d

# Stop services
docker-compose down

# Restart services
docker-compose restart

# View logs
docker-compose logs -f immich_server
docker-compose logs -f immich_machine_learning

# Update Immich
docker-compose pull
docker-compose up -d

# Check service status
docker-compose ps
```

### Individual Container Management

```bash
# Restart specific service
docker-compose restart immich_server

# Scale machine learning workers
docker-compose up -d --scale immich_machine_learning=2

# Execute commands in container
docker-compose exec immich_server bash
```

### Health Monitoring

```bash
# Check container health
docker-compose exec immich_server wget -qO- http://localhost:3001/api/server-info/ping

# Monitor resource usage
docker stats

# Database health check
docker-compose exec immich_postgres pg_isready -U postgres
```

## 6. Troubleshooting

### Common Issues

1. **Database connection failed**:
```bash
# Check PostgreSQL logs
docker-compose logs immich_postgres

# Reset database
docker-compose down
docker volume rm immich_postgres_data
docker-compose up -d
```

2. **Upload issues**:
```bash
# Check permissions
ls -la /var/lib/immich/upload

# Fix permissions
sudo chown -R 1000:1000 /var/lib/immich/upload
sudo chmod -R 755 /var/lib/immich/upload
```

3. **Machine learning not working**:
```bash
# Check ML service logs
docker-compose logs immich_machine_learning

# Restart ML service
docker-compose restart immich_machine_learning

# Check GPU support
docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

4. **Mobile app won't connect**:
```bash
# Check server URL accessibility
curl http://your-server:2283/api/server-info/ping

# Check firewall
sudo firewall-cmd --list-ports
sudo ufw status
```

### Debug Mode

```bash
# Enable debug logging
echo "LOG_LEVEL=debug" >> immich.env
docker-compose up -d

# View detailed logs
docker-compose logs -f --tail=100
```

## 7. Security Considerations

### Network Security

```bash
# Configure firewall
sudo firewall-cmd --permanent --add-port=2283/tcp
sudo firewall-cmd --reload

# UFW (Ubuntu/Debian)
sudo ufw allow 2283/tcp
sudo ufw enable
```

### SSL/TLS Configuration

```bash
# Install SSL certificate with Certbot
sudo certbot --nginx -d photos.example.com

# Manual certificate installation
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/immich.key \
    -out /etc/ssl/certs/immich.crt
```

### Access Control

```nginx
# Restrict admin access by IP
location /admin {
    allow 192.168.1.0/24;
    deny all;
    proxy_pass http://localhost:2283;
}

# Basic authentication
location / {
    auth_basic "Immich Access";
    auth_basic_user_file /etc/nginx/.htpasswd;
    proxy_pass http://localhost:2283;
}
```

### Database Security

```bash
# Secure PostgreSQL
docker-compose exec immich_postgres psql -U postgres -c "ALTER USER postgres PASSWORD 'new_secure_password';"

# Enable SSL in PostgreSQL
# Add to docker-compose.yml:
# command: postgres -c ssl=on -c ssl_cert_file=/etc/ssl/certs/server.crt
```

## 8. Performance Tuning

### Database Optimization

```sql
-- Connect to PostgreSQL
docker-compose exec immich_postgres psql -U postgres immich

-- Optimize for photo metadata
ALTER SYSTEM SET shared_buffers = '256MB';
ALTER SYSTEM SET effective_cache_size = '1GB';
ALTER SYSTEM SET maintenance_work_mem = '64MB';
ALTER SYSTEM SET random_page_cost = 1.1;

-- Restart PostgreSQL
docker-compose restart immich_postgres
```

### Storage Optimization

```bash
# Use SSD for database
mkdir -p /mnt/ssd/postgresql
docker-compose down
sudo mv /var/lib/docker/volumes/immich_postgres_data /mnt/ssd/postgresql
ln -s /mnt/ssd/postgresql /var/lib/docker/volumes/immich_postgres_data

# Configure photo storage on separate drive
echo "UPLOAD_LOCATION=/mnt/storage/photos" >> immich.env
```

### Memory Configuration

```yaml
# Add to docker-compose.override.yml
services:
  immich-server:
    deploy:
      resources:
        limits:
          memory: 2G
        reservations:
          memory: 1G

  immich-machine-learning:
    deploy:
      resources:
        limits:
          memory: 4G
        reservations:
          memory: 2G
```

### GPU Acceleration

```yaml
# For NVIDIA GPU support
services:
  immich-machine-learning:
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

## 9. Backup and Restore

### Database Backup

```bash
#!/bin/bash
# backup-immich-db.sh

BACKUP_DIR="/var/backups/immich"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p $BACKUP_DIR

# Backup PostgreSQL database
docker-compose exec -T immich_postgres pg_dump -U postgres immich | \
    gzip > $BACKUP_DIR/immich_db_$DATE.sql.gz

# Backup environment configuration
cp immich.env $BACKUP_DIR/immich_env_$DATE.env

echo "Database backup completed: $BACKUP_DIR/immich_db_$DATE.sql.gz"
```

### Photo and Video Backup

```bash
#!/bin/bash
# backup-immich-photos.sh

BACKUP_DIR="/var/backups/immich"
UPLOAD_DIR="/var/lib/immich/upload"
DATE=$(date +%Y%m%d_%H%M%S)

# Incremental backup with rsync
rsync -av --progress --delete \
    $UPLOAD_DIR/ \
    $BACKUP_DIR/photos_backup/

# Create compressed archive for offsite backup
tar -czf $BACKUP_DIR/immich_photos_$DATE.tar.gz \
    -C /var/lib/immich upload

echo "Photo backup completed"
```

### Complete System Backup

```bash
#!/bin/bash
# backup-immich-complete.sh

BACKUP_DIR="/var/backups/immich"
DATE=$(date +%Y%m%d_%H%M%S)

# Stop services for consistent backup
docker-compose down

# Backup all volumes
docker run --rm -v immich_postgres_data:/data -v $BACKUP_DIR:/backup \
    alpine tar czf /backup/postgres_volume_$DATE.tar.gz -C /data .

docker run --rm -v immich_redis_data:/data -v $BACKUP_DIR:/backup \
    alpine tar czf /backup/redis_volume_$DATE.tar.gz -C /data .

# Backup upload directory
tar -czf $BACKUP_DIR/upload_$DATE.tar.gz -C /var/lib/immich upload

# Backup configuration
cp -r ~/immich $BACKUP_DIR/config_$DATE/

# Restart services
docker-compose --env-file immich.env up -d

echo "Complete backup finished"
```

### Restore Procedures

```bash
# Restore database
docker-compose down
docker volume rm immich_postgres_data
docker-compose up -d immich_postgres
sleep 30

gunzip -c immich_db_backup.sql.gz | \
    docker-compose exec -T immich_postgres psql -U postgres immich

# Restore photos
sudo rm -rf /var/lib/immich/upload/*
sudo tar -xzf immich_photos_backup.tar.gz -C /var/lib/immich/

# Restart all services
docker-compose up -d
```

## 10. System Requirements

### Minimum Requirements
- **CPU**: 2 cores, 2.0 GHz
- **RAM**: 4GB
- **Storage**: 50GB + photo storage
- **Network**: 100 Mbps

### Recommended Requirements
- **CPU**: 4+ cores, 3.0+ GHz
- **RAM**: 8GB+
- **Storage**: 1TB+ SSD + bulk storage
- **GPU**: NVIDIA GTX 1060+ or equivalent
- **Network**: Gigabit Ethernet

### Storage Planning

| Photo Count | Estimated Storage | RAM Recommended |
|-------------|------------------|-----------------|
| 10,000      | 50GB             | 4GB             |
| 50,000      | 250GB            | 6GB             |
| 100,000     | 500GB            | 8GB             |
| 500,000+    | 2TB+             | 16GB+           |

## 11. Support

### Official Resources
- **Website**: https://immich.app
- **GitHub**: https://github.com/immich-app/immich
- **Documentation**: https://immich.app/docs
- **Discord**: https://discord.immich.app

### Community Support
- **Reddit**: r/immich
- **GitHub Discussions**: https://github.com/immich-app/immich/discussions
- **Issues**: https://github.com/immich-app/immich/issues

## 12. Contributing

### How to Contribute
1. Fork the repository on GitHub
2. Create a feature branch
3. Submit pull request
4. Follow TypeScript/Angular coding standards
5. Include tests and documentation

### Development Setup
```bash
# Clone repository
git clone https://github.com/immich-app/immich.git
cd immich

# Install dependencies
npm ci --workspaces

# Start development environment
npm run dev
```

## 13. License

Immich is licensed under the GNU Affero General Public License v3.0 (AGPL-3.0).

Key points:
- Free to use, modify, and distribute
- Source code must remain open
- Network use triggers copyleft requirements
- Commercial use allowed with restrictions

## 14. Acknowledgments

### Credits
- **Immich Team**: Core development team
- **Community Contributors**: Feature development and testing
- **Machine Learning Libraries**: TensorFlow, OpenCV
- **Database Systems**: PostgreSQL, Redis

## 15. Version History

### Recent Releases
- **v1.90.x**: Latest stable with enhanced performance
- **v1.80.x**: Added advanced search features
- **v1.70.x**: Improved mobile sync and UI

### Major Features by Version
- **v1.90**: Performance optimizations, better mobile sync
- **v1.80**: Advanced search, facial recognition improvements
- **v1.70**: Enhanced UI, better album management

## 16. Appendices

### A. Mobile App Setup

#### iOS App Installation
1. Download Immich from the App Store
2. Open app and enter server URL: `https://photos.example.com`
3. Login with your credentials
4. Configure auto-backup settings
5. Select albums and folders to sync

#### Android App Setup
1. Install from Google Play Store or F-Droid
2. Configure server connection
3. Enable background sync
4. Set up backup preferences

### B. API Usage Examples

#### Get Server Info
```bash
curl -X GET "http://localhost:2283/api/server-info" \
  -H "x-api-key: your_api_key"
```

#### Upload Photo via API
```bash
curl -X POST "http://localhost:2283/api/asset/upload" \
  -H "x-api-key: your_api_key" \
  -F "assetData=@photo.jpg"
```

### C. Migration Scripts

#### From Google Photos
```python
#!/usr/bin/env python3
# migrate-google-photos.py

import os
import requests
from pathlib import Path

def upload_to_immich(file_path, api_key, server_url):
    url = f"{server_url}/api/asset/upload"
    headers = {"x-api-key": api_key}
    
    with open(file_path, 'rb') as f:
        files = {"assetData": f}
        response = requests.post(url, headers=headers, files=files)
        return response.status_code == 201

# Usage
api_key = "your_api_key"
server_url = "http://localhost:2283"
photos_dir = "/path/to/google/photos"

for photo in Path(photos_dir).rglob("*.jpg"):
    if upload_to_immich(photo, api_key, server_url):
        print(f"Uploaded: {photo}")
    else:
        print(f"Failed: {photo}")
```

### D. Monitoring Script

```bash
#!/bin/bash
# monitor-immich.sh

echo "=== Immich Service Status ==="
docker-compose ps

echo -e "\n=== Container Resource Usage ==="
docker stats --no-stream

echo -e "\n=== Database Status ==="
docker-compose exec immich_postgres pg_isready -U postgres

echo -e "\n=== Storage Usage ==="
df -h /var/lib/immich/upload

echo -e "\n=== Recent Uploads ==="
find /var/lib/immich/upload -type f -mtime -1 | wc -l
echo "files uploaded in last 24 hours"

echo -e "\n=== API Health Check ==="
curl -s http://localhost:2283/api/server-info/ping || echo "API not responding"
```

---

For more information and updates, visit https://github.com/howtomgr/immich