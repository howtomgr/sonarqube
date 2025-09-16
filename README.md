# sonarqube Installation Guide

sonarqube is a free and open-source continuous code quality inspection platform. SonarQube performs automatic code review to detect bugs, vulnerabilities, and code smells, serving as an alternative to commercial tools like Coverity or Checkmarx

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
  - CPU: 2+ cores minimum
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 10GB for installation
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9000 (default sonarqube port)
  - Port 9001 for Elasticsearch
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

# Install sonarqube
sudo dnf install -y sonarqube

# Enable and start service
sudo systemctl enable --now sonarqube

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Verify installation
sonar.sh status
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install sonarqube
sudo apt install -y sonarqube

# Enable and start service
sudo systemctl enable --now sonarqube

# Configure firewall
sudo ufw allow 9000

# Verify installation
sonar.sh status
```

### Arch Linux

```bash
# Install sonarqube
sudo pacman -S sonarqube

# Enable and start service
sudo systemctl enable --now sonarqube

# Verify installation
sonar.sh status
```

### Alpine Linux

```bash
# Install sonarqube
apk add --no-cache sonarqube

# Enable and start service
rc-update add sonarqube default
rc-service sonarqube start

# Verify installation
sonar.sh status
```

### openSUSE/SLES

```bash
# Install sonarqube
sudo zypper install -y sonarqube

# Enable and start service
sudo systemctl enable --now sonarqube

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
sudo firewall-cmd --reload

# Verify installation
sonar.sh status
```

### macOS

```bash
# Using Homebrew
brew install sonarqube

# Start service
brew services start sonarqube

# Verify installation
sonar.sh status
```

### FreeBSD

```bash
# Using pkg
pkg install sonarqube

# Enable in rc.conf
echo 'sonarqube_enable="YES"' >> /etc/rc.conf

# Start service
service sonarqube start

# Verify installation
sonar.sh status
```

### Windows

```bash
# Using Chocolatey
choco install sonarqube

# Or using Scoop
scoop install sonarqube

# Verify installation
sonar.sh status
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/sonarqube

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
sonar.sh status
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable sonarqube

# Start service
sudo systemctl start sonarqube

# Stop service
sudo systemctl stop sonarqube

# Restart service
sudo systemctl restart sonarqube

# Check status
sudo systemctl status sonarqube

# View logs
sudo journalctl -u sonarqube -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add sonarqube default

# Start service
rc-service sonarqube start

# Stop service
rc-service sonarqube stop

# Restart service
rc-service sonarqube restart

# Check status
rc-service sonarqube status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'sonarqube_enable="YES"' >> /etc/rc.conf

# Start service
service sonarqube start

# Stop service
service sonarqube stop

# Restart service
service sonarqube restart

# Check status
service sonarqube status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start sonarqube
brew services stop sonarqube
brew services restart sonarqube

# Check status
brew services list | grep sonarqube
```

### Windows Service Manager

```powershell
# Start service
net start sonarqube

# Stop service
net stop sonarqube

# Using PowerShell
Start-Service sonarqube
Stop-Service sonarqube
Restart-Service sonarqube

# Check status
Get-Service sonarqube
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream sonarqube_backend {
    server 127.0.0.1:9000;
}

server {
    listen 80;
    server_name sonarqube.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name sonarqube.example.com;

    ssl_certificate /etc/ssl/certs/sonarqube.example.com.crt;
    ssl_certificate_key /etc/ssl/private/sonarqube.example.com.key;

    location / {
        proxy_pass http://sonarqube_backend;
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
    ServerName sonarqube.example.com
    Redirect permanent / https://sonarqube.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName sonarqube.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/sonarqube.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/sonarqube.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9000/
    ProxyPassReverse / http://127.0.0.1:9000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend sonarqube_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/sonarqube.pem
    redirect scheme https if !{ ssl_fc }
    default_backend sonarqube_backend

backend sonarqube_backend
    balance roundrobin
    server sonarqube1 127.0.0.1:9000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R sonarqube:sonarqube /etc/sonarqube
sudo chmod 750 /etc/sonarqube

# Configure firewall
sudo firewall-cmd --permanent --add-port=9000/tcp
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
sudo systemctl status sonarqube

# View logs
sudo journalctl -u sonarqube -f

# Monitor resource usage
top -p $(pgrep sonarqube)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/sonarqube"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/sonarqube-backup-$DATE.tar.gz" /etc/sonarqube /var/lib/sonarqube

echo "Backup completed: $BACKUP_DIR/sonarqube-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop sonarqube

# Restore from backup
tar -xzf /backup/sonarqube/sonarqube-backup-*.tar.gz -C /

# Start service
sudo systemctl start sonarqube
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u sonarqube -n 100
sudo tail -f /var/log/sonarqube/sonarqube.log

# Check configuration
sonar.sh status

# Check permissions
ls -la /etc/sonarqube
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9000

# Test connectivity
telnet localhost 9000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep sonarqube)

# Check disk I/O
iotop -p $(pgrep sonarqube)

# Check connections
ss -an | grep 9000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  sonarqube:
    image: sonarqube:latest
    ports:
      - "9000:9000"
    volumes:
      - ./config:/etc/sonarqube
      - ./data:/var/lib/sonarqube
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update sonarqube

# Debian/Ubuntu
sudo apt update && sudo apt upgrade sonarqube

# Arch Linux
sudo pacman -Syu sonarqube

# Alpine Linux
apk update && apk upgrade sonarqube

# openSUSE
sudo zypper update sonarqube

# FreeBSD
pkg update && pkg upgrade sonarqube

# Always backup before updates
tar -czf /backup/sonarqube-pre-update-$(date +%Y%m%d).tar.gz /etc/sonarqube

# Restart after updates
sudo systemctl restart sonarqube
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/sonarqube

# Clean old logs
find /var/log/sonarqube -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/sonarqube
```

## Additional Resources

- Official Documentation: https://docs.sonarqube.org/
- GitHub Repository: https://github.com/sonarqube/sonarqube
- Community Forum: https://forum.sonarqube.org/
- Best Practices Guide: https://docs.sonarqube.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
