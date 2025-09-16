# Mailu Installation Guide

Mailu is a free and open-source Mail Server Stack. Simple yet full-featured mail server

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
  - CPU: 2 cores minimum (4+ cores recommended)
  - RAM: 2GB minimum (4GB+ recommended)
  - Storage: 1GB for installation
  - Network: 443 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 443 (default mailu port)
- **Dependencies**:
  - docker, docker-compose
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

# Install mailu
sudo dnf install -y mailu docker, docker-compose

# Enable and start service
sudo systemctl enable --now mailu

# Configure firewall
sudo firewall-cmd --permanent --add-service=mailu
sudo firewall-cmd --reload

# Verify installation
mailu --version || systemctl status mailu
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install mailu
sudo apt install -y mailu docker, docker-compose

# Enable and start service
sudo systemctl enable --now mailu

# Configure firewall
sudo ufw allow 443

# Verify installation
mailu --version || systemctl status mailu
```

### Arch Linux

```bash
# Install mailu
sudo pacman -S mailu

# Enable and start service
sudo systemctl enable --now mailu

# Verify installation
mailu --version || systemctl status mailu
```

### Alpine Linux

```bash
# Install mailu
apk add --no-cache mailu

# Enable and start service
rc-update add mailu default
rc-service mailu start

# Verify installation
mailu --version || rc-service mailu status
```

### openSUSE/SLES

```bash
# Install mailu
sudo zypper install -y mailu docker, docker-compose

# Enable and start service
sudo systemctl enable --now mailu

# Configure firewall
sudo firewall-cmd --permanent --add-service=mailu
sudo firewall-cmd --reload

# Verify installation
mailu --version || systemctl status mailu
```

### macOS

```bash
# Using Homebrew
brew install mailu

# Start service
brew services start mailu

# Verify installation
mailu --version
```

### FreeBSD

```bash
# Using pkg
pkg install mailu

# Enable in rc.conf
echo 'mailu_enable="YES"' >> /etc/rc.conf

# Start service
service mailu start

# Verify installation
mailu --version || service mailu status
```

### Windows

```powershell
# Using Chocolatey
choco install mailu

# Or using Scoop
scoop install mailu

# Verify installation
mailu --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /mailu

# Set up basic configuration
sudo tee /mailu/mailu.conf << 'EOF'
# Mailu Configuration
MESSAGE_RATELIMIT=200/hour
EOF

# Test configuration
sudo mailu -t || sudo mailu configtest

# Reload service
sudo systemctl reload mailu
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R mailu:mailu /mailu
sudo chmod 750 /mailu

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable mailu

# Start service
sudo systemctl start mailu

# Stop service
sudo systemctl stop mailu

# Restart service
sudo systemctl restart mailu

# Reload configuration
sudo systemctl reload mailu

# Check status
sudo systemctl status mailu

# View logs
sudo journalctl -u mailu -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add mailu default

# Start service
rc-service mailu start

# Stop service
rc-service mailu stop

# Restart service
rc-service mailu restart

# Check status
rc-service mailu status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'mailu_enable="YES"' >> /etc/rc.conf

# Start service
service mailu start

# Stop service
service mailu stop

# Restart service
service mailu restart

# Check status
service mailu status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start mailu
brew services stop mailu
brew services restart mailu

# Check status
brew services list | grep mailu
```

### Windows Service Manager

```powershell
# Start service
net start mailu

# Stop service
net stop mailu

# Using PowerShell
Start-Service mailu
Stop-Service mailu
Restart-Service mailu

# Check status
Get-Service mailu
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /mailu/mailu.conf << 'EOF'
MESSAGE_RATELIMIT=200/hour
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart mailu
```

### Clustering and High Availability

```bash
# Configure clustering (if supported)
# See official documentation for cluster setup

# Basic load balancing setup example
# Configure multiple instances on different ports
```

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream mailu_backend {
    server 127.0.0.1:443;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name mailu.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name mailu.example.com;

    ssl_certificate /etc/ssl/certs/mailu.example.com.crt;
    ssl_certificate_key /etc/ssl/private/mailu.example.com.key;

    location / {
        proxy_pass http://mailu_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # WebSocket support (if needed)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName mailu.example.com
    Redirect permanent / https://mailu.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName mailu.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/mailu.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/mailu.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:443/
    ProxyPassReverse / http://127.0.0.1:443/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:443/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend mailu_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/mailu.pem
    redirect scheme https if !{ ssl_fc }
    default_backend mailu_backend

backend mailu_backend
    balance roundrobin
    option httpchk GET /health
    server mailu1 127.0.0.1:443 check
    server mailu2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R mailu:mailu /mailu
sudo chmod 750 /mailu

# Configure firewall
sudo firewall-cmd --permanent --add-service=mailu
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/mailu.conf << 'EOF'
[mailu]
enabled = true
port = 443
filter = mailu
logpath = /var/log/mailu/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/mailu.key \
    -out /etc/ssl/certs/mailu.crt

# Configure SSL in mailu
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE mailu_db;
CREATE USER mailu_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE mailu_db TO mailu_user;
EOF

# Configure mailu to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE mailu_db;
CREATE USER 'mailu_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON mailu_db.* TO 'mailu_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

## Performance Optimization

### System Tuning

```bash
# Kernel parameters
sudo tee -a /etc/sysctl.conf << EOF
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.core.netdev_max_backlog = 5000
vm.swappiness = 10
EOF

sudo sysctl -p

# Mailu specific tuning
MESSAGE_RATELIMIT=200/hour
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
mailu soft nofile 65535
mailu hard nofile 65535
mailu soft nproc 32768
mailu hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'mailu'
    static_configs:
      - targets: ['localhost:443']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet mailu; then
    echo "Mailu is running"
    exit 0
else
    echo "Mailu is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/mailu << 'EOF'
/var/log/mailu/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 mailu mailu
    postrotate
        systemctl reload mailu > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Mailu backup script
BACKUP_DIR="/backup/mailu"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop mailu

# Backup configuration
tar -czf "$BACKUP_DIR/mailu-config-$DATE.tar.gz" /mailu

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/mailu-data-$DATE.tar.gz" /var/lib/mailu

# Start service
systemctl start mailu

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop mailu

# Restore configuration
sudo tar -xzf /backup/mailu/mailu-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/mailu/mailu-data-*.tar.gz -C /

# Set permissions
sudo chown -R mailu:mailu /mailu
sudo chown -R mailu:mailu /var/lib/mailu

# Start service
sudo systemctl start mailu
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u mailu -n 100
sudo tail -f /var/log/mailu/*.log

# Check configuration
sudo mailu -t || sudo mailu configtest

# Check permissions
ls -la /mailu
ls -la /var/lib/mailu
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 443
sudo netstat -tlnp | grep 443

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 443
nc -zv localhost 443
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep nginx)
htop -p $(pgrep nginx)

# Check connections
ss -ant | grep :443 | wc -l

# Monitor I/O
iotop -p $(pgrep nginx)
```

### Debug Mode

```bash
# Run in debug mode
sudo mailu -d
# or
sudo mailu debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  mailu:
    image: mailu:latest
    container_name: mailu
    ports:
      - "443:443"
    volumes:
      - ./config:/mailu
      - ./data:/var/lib/mailu
    environment:
      - mailu_CONFIG=/mailu/mailu.conf
    restart: unless-stopped
    networks:
      - mailu_net

networks:
  mailu_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mailu
spec:
  replicas: 3
  selector:
    matchLabels:
      app: mailu
  template:
    metadata:
      labels:
        app: mailu
    spec:
      containers:
      - name: mailu
        image: mailu:latest
        ports:
        - containerPort: 443
        volumeMounts:
        - name: config
          mountPath: /mailu
      volumes:
      - name: config
        configMap:
          name: mailu-config
---
apiVersion: v1
kind: Service
metadata:
  name: mailu
spec:
  selector:
    app: mailu
  ports:
  - port: 443
    targetPort: 443
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure Mailu
  hosts: all
  become: yes
  tasks:
    - name: Install mailu
      package:
        name: mailu
        state: present
    
    - name: Configure mailu
      template:
        src: mailu.conf.j2
        dest: /mailu/mailu.conf
        owner: mailu
        group: mailu
        mode: '0640'
      notify: restart mailu
    
    - name: Start and enable mailu
      systemd:
        name: mailu
        state: started
        enabled: yes
  
  handlers:
    - name: restart mailu
      systemd:
        name: mailu
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update mailu

# Debian/Ubuntu
sudo apt update && sudo apt upgrade mailu

# Arch Linux
sudo pacman -Syu mailu

# Alpine Linux
apk update && apk upgrade mailu

# openSUSE
sudo zypper update mailu

# FreeBSD
pkg update && pkg upgrade mailu

# Always backup before updates
tar -czf /backup/mailu-pre-update-$(date +%Y%m%d).tar.gz /mailu

# Restart after updates
sudo systemctl restart mailu
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/mailu -name "*.log" -mtime +30 -delete

# Verify integrity
sudo mailu --verify || sudo mailu check

# Update databases (if applicable)
sudo mailu-update-db

# Optimize performance
sudo mailu-optimize

# Check for security updates
sudo mailu --security-check
```

## Additional Resources

- Official Documentation: https://docs.mailu.org/
- GitHub Repository: https://github.com/mailu/mailu
- Community Forum: https://forum.mailu.org/
- Wiki: https://wiki.mailu.org/
- Comparison vs mailcow, Mail-in-a-Box, iRedMail, Postal: https://docs.mailu.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
