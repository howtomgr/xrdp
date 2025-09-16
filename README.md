# xrdp Installation Guide

xrdp is a free and open-source RDP server. xrdp provides open source RDP server for Linux

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
  - RAM: 1GB minimum
  - Storage: 1GB for sessions
  - Network: RDP protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3389 (default xrdp port)
  - None
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

# Install xrdp
sudo dnf install -y xrdp

# Enable and start service
sudo systemctl enable --now xrdp

# Configure firewall
sudo firewall-cmd --permanent --add-port=3389/tcp
sudo firewall-cmd --reload

# Verify installation
xrdp --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install xrdp
sudo apt install -y xrdp

# Enable and start service
sudo systemctl enable --now xrdp

# Configure firewall
sudo ufw allow 3389

# Verify installation
xrdp --version
```

### Arch Linux

```bash
# Install xrdp
sudo pacman -S xrdp

# Enable and start service
sudo systemctl enable --now xrdp

# Verify installation
xrdp --version
```

### Alpine Linux

```bash
# Install xrdp
apk add --no-cache xrdp

# Enable and start service
rc-update add xrdp default
rc-service xrdp start

# Verify installation
xrdp --version
```

### openSUSE/SLES

```bash
# Install xrdp
sudo zypper install -y xrdp

# Enable and start service
sudo systemctl enable --now xrdp

# Configure firewall
sudo firewall-cmd --permanent --add-port=3389/tcp
sudo firewall-cmd --reload

# Verify installation
xrdp --version
```

### macOS

```bash
# Using Homebrew
brew install xrdp

# Start service
brew services start xrdp

# Verify installation
xrdp --version
```

### FreeBSD

```bash
# Using pkg
pkg install xrdp

# Enable in rc.conf
echo 'xrdp_enable="YES"' >> /etc/rc.conf

# Start service
service xrdp start

# Verify installation
xrdp --version
```

### Windows

```bash
# Using Chocolatey
choco install xrdp

# Or using Scoop
scoop install xrdp

# Verify installation
xrdp --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/xrdp

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
xrdp --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable xrdp

# Start service
sudo systemctl start xrdp

# Stop service
sudo systemctl stop xrdp

# Restart service
sudo systemctl restart xrdp

# Check status
sudo systemctl status xrdp

# View logs
sudo journalctl -u xrdp -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add xrdp default

# Start service
rc-service xrdp start

# Stop service
rc-service xrdp stop

# Restart service
rc-service xrdp restart

# Check status
rc-service xrdp status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'xrdp_enable="YES"' >> /etc/rc.conf

# Start service
service xrdp start

# Stop service
service xrdp stop

# Restart service
service xrdp restart

# Check status
service xrdp status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start xrdp
brew services stop xrdp
brew services restart xrdp

# Check status
brew services list | grep xrdp
```

### Windows Service Manager

```powershell
# Start service
net start xrdp

# Stop service
net stop xrdp

# Using PowerShell
Start-Service xrdp
Stop-Service xrdp
Restart-Service xrdp

# Check status
Get-Service xrdp
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream xrdp_backend {
    server 127.0.0.1:3389;
}

server {
    listen 80;
    server_name xrdp.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name xrdp.example.com;

    ssl_certificate /etc/ssl/certs/xrdp.example.com.crt;
    ssl_certificate_key /etc/ssl/private/xrdp.example.com.key;

    location / {
        proxy_pass http://xrdp_backend;
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
    ServerName xrdp.example.com
    Redirect permanent / https://xrdp.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName xrdp.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/xrdp.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/xrdp.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3389/
    ProxyPassReverse / http://127.0.0.1:3389/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend xrdp_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/xrdp.pem
    redirect scheme https if !{ ssl_fc }
    default_backend xrdp_backend

backend xrdp_backend
    balance roundrobin
    server xrdp1 127.0.0.1:3389 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R xrdp:xrdp /etc/xrdp
sudo chmod 750 /etc/xrdp

# Configure firewall
sudo firewall-cmd --permanent --add-port=3389/tcp
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
sudo systemctl status xrdp

# View logs
sudo journalctl -u xrdp -f

# Monitor resource usage
top -p $(pgrep xrdp)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/xrdp"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/xrdp-backup-$DATE.tar.gz" /etc/xrdp /var/lib/xrdp

echo "Backup completed: $BACKUP_DIR/xrdp-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop xrdp

# Restore from backup
tar -xzf /backup/xrdp/xrdp-backup-*.tar.gz -C /

# Start service
sudo systemctl start xrdp
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u xrdp -n 100
sudo tail -f /var/log/xrdp/xrdp.log

# Check configuration
xrdp --version

# Check permissions
ls -la /etc/xrdp
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3389

# Test connectivity
telnet localhost 3389

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep xrdp)

# Check disk I/O
iotop -p $(pgrep xrdp)

# Check connections
ss -an | grep 3389
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  xrdp:
    image: xrdp:latest
    ports:
      - "3389:3389"
    volumes:
      - ./config:/etc/xrdp
      - ./data:/var/lib/xrdp
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update xrdp

# Debian/Ubuntu
sudo apt update && sudo apt upgrade xrdp

# Arch Linux
sudo pacman -Syu xrdp

# Alpine Linux
apk update && apk upgrade xrdp

# openSUSE
sudo zypper update xrdp

# FreeBSD
pkg update && pkg upgrade xrdp

# Always backup before updates
tar -czf /backup/xrdp-pre-update-$(date +%Y%m%d).tar.gz /etc/xrdp

# Restart after updates
sudo systemctl restart xrdp
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/xrdp

# Clean old logs
find /var/log/xrdp -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/xrdp
```

## Additional Resources

- Official Documentation: https://docs.xrdp.org/
- GitHub Repository: https://github.com/xrdp/xrdp
- Community Forum: https://forum.xrdp.org/
- Best Practices Guide: https://docs.xrdp.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
