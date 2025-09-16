# NSD Installation Guide

NSD is a free and open-source DNS Server. An authoritative-only, high performance DNS server

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
  - Network: 53 ports
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 53 (default nsd port)
- **Dependencies**:
  - nsd-control, ldns-utils
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

# Install nsd
sudo dnf install -y nsd nsd-control, ldns-utils

# Enable and start service
sudo systemctl enable --now nsd

# Configure firewall
sudo firewall-cmd --permanent --add-service=nsd
sudo firewall-cmd --reload

# Verify installation
nsd --version || systemctl status nsd
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install nsd
sudo apt install -y nsd nsd-control, ldns-utils

# Enable and start service
sudo systemctl enable --now nsd

# Configure firewall
sudo ufw allow 53

# Verify installation
nsd --version || systemctl status nsd
```

### Arch Linux

```bash
# Install nsd
sudo pacman -S nsd

# Enable and start service
sudo systemctl enable --now nsd

# Verify installation
nsd --version || systemctl status nsd
```

### Alpine Linux

```bash
# Install nsd
apk add --no-cache nsd

# Enable and start service
rc-update add nsd default
rc-service nsd start

# Verify installation
nsd --version || rc-service nsd status
```

### openSUSE/SLES

```bash
# Install nsd
sudo zypper install -y nsd nsd-control, ldns-utils

# Enable and start service
sudo systemctl enable --now nsd

# Configure firewall
sudo firewall-cmd --permanent --add-service=nsd
sudo firewall-cmd --reload

# Verify installation
nsd --version || systemctl status nsd
```

### macOS

```bash
# Using Homebrew
brew install nsd

# Start service
brew services start nsd

# Verify installation
nsd --version
```

### FreeBSD

```bash
# Using pkg
pkg install nsd

# Enable in rc.conf
echo 'nsd_enable="YES"' >> /etc/rc.conf

# Start service
service nsd start

# Verify installation
nsd --version || service nsd status
```

### Windows

```powershell
# Using Chocolatey
choco install nsd

# Or using Scoop
scoop install nsd

# Verify installation
nsd --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory if needed
sudo mkdir -p /etc/nsd

# Set up basic configuration
sudo tee /etc/nsd/nsd.conf << 'EOF'
# NSD Configuration
server-count: 4, tcp-count: 100
EOF

# Test configuration
sudo nsd -t || sudo nsd configtest

# Reload service
sudo systemctl reload nsd
```

### Security Hardening

```bash
# Set appropriate permissions
sudo chown -R nsd:nsd /etc/nsd
sudo chmod 750 /etc/nsd

# Enable security features
# See security section for detailed hardening steps
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable nsd

# Start service
sudo systemctl start nsd

# Stop service
sudo systemctl stop nsd

# Restart service
sudo systemctl restart nsd

# Reload configuration
sudo systemctl reload nsd

# Check status
sudo systemctl status nsd

# View logs
sudo journalctl -u nsd -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add nsd default

# Start service
rc-service nsd start

# Stop service
rc-service nsd stop

# Restart service
rc-service nsd restart

# Check status
rc-service nsd status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'nsd_enable="YES"' >> /etc/rc.conf

# Start service
service nsd start

# Stop service
service nsd stop

# Restart service
service nsd restart

# Check status
service nsd status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start nsd
brew services stop nsd
brew services restart nsd

# Check status
brew services list | grep nsd
```

### Windows Service Manager

```powershell
# Start service
net start nsd

# Stop service
net stop nsd

# Using PowerShell
Start-Service nsd
Stop-Service nsd
Restart-Service nsd

# Check status
Get-Service nsd
```

## Advanced Configuration

### Performance Optimization

```bash
# Configure performance settings
cat >> /etc/nsd/nsd.conf << 'EOF'
server-count: 4, tcp-count: 100
EOF

# Apply system tuning
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# Restart service
sudo systemctl restart nsd
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
upstream nsd_backend {
    server 127.0.0.1:53;
    server 127.0.0.1:{default_port}1 backup;
}

server {
    listen 80;
    server_name nsd.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name nsd.example.com;

    ssl_certificate /etc/ssl/certs/nsd.example.com.crt;
    ssl_certificate_key /etc/ssl/private/nsd.example.com.key;

    location / {
        proxy_pass http://nsd_backend;
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
    ServerName nsd.example.com
    Redirect permanent / https://nsd.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName nsd.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/nsd.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/nsd.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:53/
    ProxyPassReverse / http://127.0.0.1:53/
    
    # WebSocket support (if needed)
    RewriteEngine on
    RewriteCond %{HTTP:Upgrade} websocket [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/?(.*) "ws://127.0.0.1:53/$1" [P,L]
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend nsd_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/nsd.pem
    redirect scheme https if !{ ssl_fc }
    default_backend nsd_backend

backend nsd_backend
    balance roundrobin
    option httpchk GET /health
    server nsd1 127.0.0.1:53 check
    server nsd2 127.0.0.1:{default_port}1 check backup
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R nsd:nsd /etc/nsd
sudo chmod 750 /etc/nsd

# Configure firewall
sudo firewall-cmd --permanent --add-service=nsd
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on

# Configure fail2ban
sudo tee /etc/fail2ban/jail.d/nsd.conf << 'EOF'
[nsd]
enabled = true
port = 53
filter = nsd
logpath = /var/log/nsd/*.log
maxretry = 5
bantime = 3600
EOF
```

### SSL/TLS Configuration

```bash
# Generate SSL certificates
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/nsd.key \
    -out /etc/ssl/certs/nsd.crt

# Configure SSL in nsd
# See official documentation for SSL configuration
```

## Database Setup

### PostgreSQL Backend (if applicable)

```bash
# Create database and user
sudo -u postgres psql << EOF
CREATE DATABASE nsd_db;
CREATE USER nsd_user WITH ENCRYPTED PASSWORD 'secure_password';
GRANT ALL PRIVILEGES ON DATABASE nsd_db TO nsd_user;
EOF

# Configure nsd to use PostgreSQL
# See official documentation for database configuration
```

### MySQL/MariaDB Backend (if applicable)

```bash
# Create database and user
sudo mysql << EOF
CREATE DATABASE nsd_db;
CREATE USER 'nsd_user'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON nsd_db.* TO 'nsd_user'@'localhost';
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

# NSD specific tuning
server-count: 4, tcp-count: 100
```

### Resource Limits

```bash
# Configure system limits
sudo tee -a /etc/security/limits.conf << EOF
nsd soft nofile 65535
nsd hard nofile 65535
nsd soft nproc 32768
nsd hard nproc 32768
EOF
```

## Monitoring

### Prometheus Integration

```yaml
# prometheus.yml configuration
scrape_configs:
  - job_name: 'nsd'
    static_configs:
      - targets: ['localhost:53']
    metrics_path: '/metrics'
```

### Health Checks

```bash
# Basic health check script
#!/bin/bash
if systemctl is-active --quiet nsd; then
    echo "NSD is running"
    exit 0
else
    echo "NSD is not running"
    exit 1
fi
```

### Log Monitoring

```bash
# Configure log rotation
sudo tee /etc/logrotate.d/nsd << 'EOF'
/var/log/nsd/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    create 0640 nsd nsd
    postrotate
        systemctl reload nsd > /dev/null 2>&1 || true
    endscript
}
EOF
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# NSD backup script
BACKUP_DIR="/backup/nsd"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"

# Stop service (if required)
systemctl stop nsd

# Backup configuration
tar -czf "$BACKUP_DIR/nsd-config-$DATE.tar.gz" /etc/nsd

# Backup data (adjust paths as needed)
tar -czf "$BACKUP_DIR/nsd-data-$DATE.tar.gz" /var/lib/nsd

# Start service
systemctl start nsd

# Clean old backups (keep 30 days)
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $BACKUP_DIR"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop nsd

# Restore configuration
sudo tar -xzf /backup/nsd/nsd-config-*.tar.gz -C /

# Restore data
sudo tar -xzf /backup/nsd/nsd-data-*.tar.gz -C /

# Set permissions
sudo chown -R nsd:nsd /etc/nsd
sudo chown -R nsd:nsd /var/lib/nsd

# Start service
sudo systemctl start nsd
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u nsd -n 100
sudo tail -f /var/log/nsd/*.log

# Check configuration
sudo nsd -t || sudo nsd configtest

# Check permissions
ls -la /etc/nsd
ls -la /var/lib/nsd
```

2. **Connection refused**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 53
sudo netstat -tlnp | grep 53

# Check firewall
sudo firewall-cmd --list-all
sudo iptables -L -n

# Test connection
telnet localhost 53
nc -zv localhost 53
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep nsd)
htop -p $(pgrep nsd)

# Check connections
ss -ant | grep :53 | wc -l

# Monitor I/O
iotop -p $(pgrep nsd)
```

### Debug Mode

```bash
# Run in debug mode
sudo nsd -d
# or
sudo nsd debug

# Increase log verbosity
# Edit configuration to enable debug logging
```

## Integration Examples

### Docker Compose

```yaml
version: '3.8'
services:
  nsd:
    image: nsd:latest
    container_name: nsd
    ports:
      - "53:53"
    volumes:
      - ./config:/etc/nsd
      - ./data:/var/lib/nsd
    environment:
      - nsd_CONFIG=/etc/nsd/nsd.conf
    restart: unless-stopped
    networks:
      - nsd_net

networks:
  nsd_net:
    driver: bridge
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nsd
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nsd
  template:
    metadata:
      labels:
        app: nsd
    spec:
      containers:
      - name: nsd
        image: nsd:latest
        ports:
        - containerPort: 53
        volumeMounts:
        - name: config
          mountPath: /etc/nsd
      volumes:
      - name: config
        configMap:
          name: nsd-config
---
apiVersion: v1
kind: Service
metadata:
  name: nsd
spec:
  selector:
    app: nsd
  ports:
  - port: 53
    targetPort: 53
  type: LoadBalancer
```

### Ansible Playbook

```yaml
---
- name: Install and configure NSD
  hosts: all
  become: yes
  tasks:
    - name: Install nsd
      package:
        name: nsd
        state: present
    
    - name: Configure nsd
      template:
        src: nsd.conf.j2
        dest: /etc/nsd/nsd.conf
        owner: nsd
        group: nsd
        mode: '0640'
      notify: restart nsd
    
    - name: Start and enable nsd
      systemd:
        name: nsd
        state: started
        enabled: yes
  
  handlers:
    - name: restart nsd
      systemd:
        name: nsd
        state: restarted
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update nsd

# Debian/Ubuntu
sudo apt update && sudo apt upgrade nsd

# Arch Linux
sudo pacman -Syu nsd

# Alpine Linux
apk update && apk upgrade nsd

# openSUSE
sudo zypper update nsd

# FreeBSD
pkg update && pkg upgrade nsd

# Always backup before updates
tar -czf /backup/nsd-pre-update-$(date +%Y%m%d).tar.gz /etc/nsd

# Restart after updates
sudo systemctl restart nsd
```

### Regular Maintenance Tasks

```bash
# Clean logs
find /var/log/nsd -name "*.log" -mtime +30 -delete

# Verify integrity
sudo nsd --verify || sudo nsd check

# Update databases (if applicable)
sudo nsd-update-db

# Optimize performance
sudo nsd-optimize

# Check for security updates
sudo nsd --security-check
```

## Additional Resources

- Official Documentation: https://docs.nsd.org/
- GitHub Repository: https://github.com/nsd/nsd
- Community Forum: https://forum.nsd.org/
- Wiki: https://wiki.nsd.org/
- Comparison vs BIND, PowerDNS, Knot DNS, CoreDNS: https://docs.nsd.org/comparison

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
