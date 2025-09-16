# fabio Installation Guide

fabio is a free and open-source load balancer. Fabio provides fast, zero-conf load balancing HTTP/S router

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
  - CPU: 1 core minimum
  - RAM: 256MB minimum
  - Storage: 100MB for config
  - Network: HTTP/HTTPS
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9999 (default fabio port)
  - UI on 9998
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

# Install fabio
sudo dnf install -y fabio

# Enable and start service
sudo systemctl enable --now fabio

# Configure firewall
sudo firewall-cmd --permanent --add-port=9999/tcp
sudo firewall-cmd --reload

# Verify installation
fabio --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install fabio
sudo apt install -y fabio

# Enable and start service
sudo systemctl enable --now fabio

# Configure firewall
sudo ufw allow 9999

# Verify installation
fabio --version
```

### Arch Linux

```bash
# Install fabio
sudo pacman -S fabio

# Enable and start service
sudo systemctl enable --now fabio

# Verify installation
fabio --version
```

### Alpine Linux

```bash
# Install fabio
apk add --no-cache fabio

# Enable and start service
rc-update add fabio default
rc-service fabio start

# Verify installation
fabio --version
```

### openSUSE/SLES

```bash
# Install fabio
sudo zypper install -y fabio

# Enable and start service
sudo systemctl enable --now fabio

# Configure firewall
sudo firewall-cmd --permanent --add-port=9999/tcp
sudo firewall-cmd --reload

# Verify installation
fabio --version
```

### macOS

```bash
# Using Homebrew
brew install fabio

# Start service
brew services start fabio

# Verify installation
fabio --version
```

### FreeBSD

```bash
# Using pkg
pkg install fabio

# Enable in rc.conf
echo 'fabio_enable="YES"' >> /etc/rc.conf

# Start service
service fabio start

# Verify installation
fabio --version
```

### Windows

```bash
# Using Chocolatey
choco install fabio

# Or using Scoop
scoop install fabio

# Verify installation
fabio --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/fabio

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
fabio --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable fabio

# Start service
sudo systemctl start fabio

# Stop service
sudo systemctl stop fabio

# Restart service
sudo systemctl restart fabio

# Check status
sudo systemctl status fabio

# View logs
sudo journalctl -u fabio -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add fabio default

# Start service
rc-service fabio start

# Stop service
rc-service fabio stop

# Restart service
rc-service fabio restart

# Check status
rc-service fabio status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'fabio_enable="YES"' >> /etc/rc.conf

# Start service
service fabio start

# Stop service
service fabio stop

# Restart service
service fabio restart

# Check status
service fabio status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start fabio
brew services stop fabio
brew services restart fabio

# Check status
brew services list | grep fabio
```

### Windows Service Manager

```powershell
# Start service
net start fabio

# Stop service
net stop fabio

# Using PowerShell
Start-Service fabio
Stop-Service fabio
Restart-Service fabio

# Check status
Get-Service fabio
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream fabio_backend {
    server 127.0.0.1:9999;
}

server {
    listen 80;
    server_name fabio.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name fabio.example.com;

    ssl_certificate /etc/ssl/certs/fabio.example.com.crt;
    ssl_certificate_key /etc/ssl/private/fabio.example.com.key;

    location / {
        proxy_pass http://fabio_backend;
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
    ServerName fabio.example.com
    Redirect permanent / https://fabio.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName fabio.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/fabio.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/fabio.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9999/
    ProxyPassReverse / http://127.0.0.1:9999/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend fabio_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/fabio.pem
    redirect scheme https if !{ ssl_fc }
    default_backend fabio_backend

backend fabio_backend
    balance roundrobin
    server fabio1 127.0.0.1:9999 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R fabio:fabio /etc/fabio
sudo chmod 750 /etc/fabio

# Configure firewall
sudo firewall-cmd --permanent --add-port=9999/tcp
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
sudo systemctl status fabio

# View logs
sudo journalctl -u fabio -f

# Monitor resource usage
top -p $(pgrep fabio)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/fabio"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/fabio-backup-$DATE.tar.gz" /etc/fabio /var/lib/fabio

echo "Backup completed: $BACKUP_DIR/fabio-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop fabio

# Restore from backup
tar -xzf /backup/fabio/fabio-backup-*.tar.gz -C /

# Start service
sudo systemctl start fabio
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u fabio -n 100
sudo tail -f /var/log/fabio/fabio.log

# Check configuration
fabio --version

# Check permissions
ls -la /etc/fabio
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9999

# Test connectivity
telnet localhost 9999

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep fabio)

# Check disk I/O
iotop -p $(pgrep fabio)

# Check connections
ss -an | grep 9999
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  fabio:
    image: fabio:latest
    ports:
      - "9999:9999"
    volumes:
      - ./config:/etc/fabio
      - ./data:/var/lib/fabio
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update fabio

# Debian/Ubuntu
sudo apt update && sudo apt upgrade fabio

# Arch Linux
sudo pacman -Syu fabio

# Alpine Linux
apk update && apk upgrade fabio

# openSUSE
sudo zypper update fabio

# FreeBSD
pkg update && pkg upgrade fabio

# Always backup before updates
tar -czf /backup/fabio-pre-update-$(date +%Y%m%d).tar.gz /etc/fabio

# Restart after updates
sudo systemctl restart fabio
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/fabio

# Clean old logs
find /var/log/fabio -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/fabio
```

## Additional Resources

- Official Documentation: https://docs.fabio.org/
- GitHub Repository: https://github.com/fabio/fabio
- Community Forum: https://forum.fabio.org/
- Best Practices Guide: https://docs.fabio.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
