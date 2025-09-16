# pritunl Installation Guide

pritunl is a free and open-source enterprise VPN server. Pritunl provides enterprise distributed OpenVPN and IPsec server with web management, serving as an alternative to commercial VPN platforms

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
  - CPU: 1 core minimum (2+ recommended)
  - RAM: 512MB minimum
  - Storage: 1GB for installation
  - Network: VPN protocols
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 443 (default pritunl port)
  - VPN ports 1194, 9700
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

# Install pritunl
sudo dnf install -y pritunl

# Enable and start service
sudo systemctl enable --now pritunl

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload

# Verify installation
pritunl version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install pritunl
sudo apt install -y pritunl

# Enable and start service
sudo systemctl enable --now pritunl

# Configure firewall
sudo ufw allow 443

# Verify installation
pritunl version
```

### Arch Linux

```bash
# Install pritunl
sudo pacman -S pritunl

# Enable and start service
sudo systemctl enable --now pritunl

# Verify installation
pritunl version
```

### Alpine Linux

```bash
# Install pritunl
apk add --no-cache pritunl

# Enable and start service
rc-update add pritunl default
rc-service pritunl start

# Verify installation
pritunl version
```

### openSUSE/SLES

```bash
# Install pritunl
sudo zypper install -y pritunl

# Enable and start service
sudo systemctl enable --now pritunl

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload

# Verify installation
pritunl version
```

### macOS

```bash
# Using Homebrew
brew install pritunl

# Start service
brew services start pritunl

# Verify installation
pritunl version
```

### FreeBSD

```bash
# Using pkg
pkg install pritunl

# Enable in rc.conf
echo 'pritunl_enable="YES"' >> /etc/rc.conf

# Start service
service pritunl start

# Verify installation
pritunl version
```

### Windows

```bash
# Using Chocolatey
choco install pritunl

# Or using Scoop
scoop install pritunl

# Verify installation
pritunl version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/pritunl

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
pritunl version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable pritunl

# Start service
sudo systemctl start pritunl

# Stop service
sudo systemctl stop pritunl

# Restart service
sudo systemctl restart pritunl

# Check status
sudo systemctl status pritunl

# View logs
sudo journalctl -u pritunl -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add pritunl default

# Start service
rc-service pritunl start

# Stop service
rc-service pritunl stop

# Restart service
rc-service pritunl restart

# Check status
rc-service pritunl status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'pritunl_enable="YES"' >> /etc/rc.conf

# Start service
service pritunl start

# Stop service
service pritunl stop

# Restart service
service pritunl restart

# Check status
service pritunl status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start pritunl
brew services stop pritunl
brew services restart pritunl

# Check status
brew services list | grep pritunl
```

### Windows Service Manager

```powershell
# Start service
net start pritunl

# Stop service
net stop pritunl

# Using PowerShell
Start-Service pritunl
Stop-Service pritunl
Restart-Service pritunl

# Check status
Get-Service pritunl
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream pritunl_backend {
    server 127.0.0.1:443;
}

server {
    listen 80;
    server_name pritunl.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name pritunl.example.com;

    ssl_certificate /etc/ssl/certs/pritunl.example.com.crt;
    ssl_certificate_key /etc/ssl/private/pritunl.example.com.key;

    location / {
        proxy_pass http://pritunl_backend;
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
    ServerName pritunl.example.com
    Redirect permanent / https://pritunl.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName pritunl.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/pritunl.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/pritunl.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:443/
    ProxyPassReverse / http://127.0.0.1:443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend pritunl_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/pritunl.pem
    redirect scheme https if !{ ssl_fc }
    default_backend pritunl_backend

backend pritunl_backend
    balance roundrobin
    server pritunl1 127.0.0.1:443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R pritunl:pritunl /etc/pritunl
sudo chmod 750 /etc/pritunl

# Configure firewall
sudo firewall-cmd --permanent --add-port=443/tcp
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
sudo systemctl status pritunl

# View logs
sudo journalctl -u pritunl -f

# Monitor resource usage
top -p $(pgrep pritunl)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/pritunl"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/pritunl-backup-$DATE.tar.gz" /etc/pritunl /var/lib/pritunl

echo "Backup completed: $BACKUP_DIR/pritunl-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop pritunl

# Restore from backup
tar -xzf /backup/pritunl/pritunl-backup-*.tar.gz -C /

# Start service
sudo systemctl start pritunl
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u pritunl -n 100
sudo tail -f /var/log/pritunl/pritunl.log

# Check configuration
pritunl version

# Check permissions
ls -la /etc/pritunl
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 443

# Test connectivity
telnet localhost 443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep pritunl)

# Check disk I/O
iotop -p $(pgrep pritunl)

# Check connections
ss -an | grep 443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  pritunl:
    image: pritunl:latest
    ports:
      - "443:443"
    volumes:
      - ./config:/etc/pritunl
      - ./data:/var/lib/pritunl
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update pritunl

# Debian/Ubuntu
sudo apt update && sudo apt upgrade pritunl

# Arch Linux
sudo pacman -Syu pritunl

# Alpine Linux
apk update && apk upgrade pritunl

# openSUSE
sudo zypper update pritunl

# FreeBSD
pkg update && pkg upgrade pritunl

# Always backup before updates
tar -czf /backup/pritunl-pre-update-$(date +%Y%m%d).tar.gz /etc/pritunl

# Restart after updates
sudo systemctl restart pritunl
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/pritunl

# Clean old logs
find /var/log/pritunl -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/pritunl
```

## Additional Resources

- Official Documentation: https://docs.pritunl.org/
- GitHub Repository: https://github.com/pritunl/pritunl
- Community Forum: https://forum.pritunl.org/
- Best Practices Guide: https://docs.pritunl.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
