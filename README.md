# celery Installation Guide

celery is a free and open-source distributed task queue. Celery provides distributed task queue for Python applications

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

- **Hardware Requirements**:
  - CPU: 2+ cores
  - RAM: 2GB minimum
  - Storage: 5GB for results
  - Network: Message broker
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 5672 (default celery port)
  - Flower UI on 5555
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install celery
sudo dnf install -y celery

# Enable and start service
sudo systemctl enable --now celery

# Configure firewall
sudo firewall-cmd --permanent --add-port=5672/tcp
sudo firewall-cmd --reload

# Verify installation
celery --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install celery
sudo apt install -y celery

# Enable and start service
sudo systemctl enable --now celery

# Configure firewall
sudo ufw allow 5672

# Verify installation
celery --version
```

### Arch Linux

```bash
# Install celery
sudo pacman -S celery

# Enable and start service
sudo systemctl enable --now celery

# Verify installation
celery --version
```

### Alpine Linux

```bash
# Install celery
apk add --no-cache celery

# Enable and start service
rc-update add celery default
rc-service celery start

# Verify installation
celery --version
```

### openSUSE/SLES

```bash
# Install celery
sudo zypper install -y celery

# Enable and start service
sudo systemctl enable --now celery

# Configure firewall
sudo firewall-cmd --permanent --add-port=5672/tcp
sudo firewall-cmd --reload

# Verify installation
celery --version
```

### macOS

```bash
# Using Homebrew
brew install celery

# Start service
brew services start celery

# Verify installation
celery --version
```

### FreeBSD

```bash
# Using pkg
pkg install celery

# Enable in rc.conf
echo 'celery_enable="YES"' >> /etc/rc.conf

# Start service
service celery start

# Verify installation
celery --version
```

### Windows

```bash
# Using Chocolatey
choco install celery

# Or using Scoop
scoop install celery

# Verify installation
celery --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/celery

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
celery --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable celery

# Start service
sudo systemctl start celery

# Stop service
sudo systemctl stop celery

# Restart service
sudo systemctl restart celery

# Check status
sudo systemctl status celery

# View logs
sudo journalctl -u celery -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add celery default

# Start service
rc-service celery start

# Stop service
rc-service celery stop

# Restart service
rc-service celery restart

# Check status
rc-service celery status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'celery_enable="YES"' >> /etc/rc.conf

# Start service
service celery start

# Stop service
service celery stop

# Restart service
service celery restart

# Check status
service celery status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start celery
brew services stop celery
brew services restart celery

# Check status
brew services list | grep celery
```

### Windows Service Manager

```powershell
# Start service
net start celery

# Stop service
net stop celery

# Using PowerShell
Start-Service celery
Stop-Service celery
Restart-Service celery

# Check status
Get-Service celery
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream celery_backend {
    server 127.0.0.1:5672;
}

server {
    listen 80;
    server_name celery.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name celery.example.com;

    ssl_certificate /etc/ssl/certs/celery.example.com.crt;
    ssl_certificate_key /etc/ssl/private/celery.example.com.key;

    location / {
        proxy_pass http://celery_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName celery.example.com
    Redirect permanent / https://celery.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName celery.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/celery.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/celery.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:5672/
    ProxyPassReverse / http://127.0.0.1:5672/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend celery_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/celery.pem
    redirect scheme https if !{ ssl_fc }
    default_backend celery_backend

backend celery_backend
    balance roundrobin
    server celery1 127.0.0.1:5672 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R celery:celery /etc/celery
sudo chmod 750 /etc/celery

# Configure firewall
sudo firewall-cmd --permanent --add-port=5672/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status celery

# View logs
sudo journalctl -u celery -f

# Monitor resource usage
top -p $(pgrep celery)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/celery"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/celery-backup-$DATE.tar.gz" /etc/celery /var/lib/celery

echo "Backup completed: $BACKUP_DIR/celery-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop celery

# Restore from backup
tar -xzf /backup/celery/celery-backup-*.tar.gz -C /

# Start service
sudo systemctl start celery
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u celery -n 100
sudo tail -f /var/log/celery/celery.log

# Check configuration
celery --version

# Check permissions
ls -la /etc/celery
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 5672

# Test connectivity
telnet localhost 5672

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep celery)

# Check disk I/O
iotop -p $(pgrep celery)

# Check connections
ss -an | grep 5672
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  celery:
    image: celery:latest
    ports:
      - "5672:5672"
    volumes:
      - ./config:/etc/celery
      - ./data:/var/lib/celery
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update celery

# Debian/Ubuntu
sudo apt update && sudo apt upgrade celery

# Arch Linux
sudo pacman -Syu celery

# Alpine Linux
apk update && apk upgrade celery

# openSUSE
sudo zypper update celery

# FreeBSD
pkg update && pkg upgrade celery

# Always backup before updates
tar -czf /backup/celery-pre-update-$(date +%Y%m%d).tar.gz /etc/celery

# Restart after updates
sudo systemctl restart celery
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/celery

# Clean old logs
find /var/log/celery -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/celery
```

## Additional Resources

- Official Documentation: https://docs.celery.org/
- GitHub Repository: https://github.com/celery/celery
- Community Forum: https://forum.celery.org/
- Best Practices Guide: https://docs.celery.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
