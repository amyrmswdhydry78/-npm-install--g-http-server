# VPS Deployment Guide

This guide covers deploying the HTTP server on a Virtual Private Server (VPS).

## Prerequisites

- Ubuntu/Debian VPS (18.04 or later)
- SSH access to your VPS
- Sudo privileges
- Domain name (optional, for SSL/TLS)

## Quick Deploy (Automated)

### 1. SSH into your VPS
```bash
ssh root@your-vps-ip
```

### 2. Clone the repository
```bash
git clone https://github.com/amyrmswdhydry78/-npm-install--g-http-server.git
cd -npm-install--g-http-server
```

### 3. Run the deployment script
```bash
chmod +x deploy.sh
./deploy.sh
```

The script will:
- Update system packages
- Install Node.js and npm
- Install dependencies
- Create a systemd service for auto-start
- Configure firewall
- Setup Nginx reverse proxy

## Manual Deployment

### 1. Update System
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

### 2. Install Node.js
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 3. Clone and Setup
```bash
git clone https://github.com/amyrmswdhydry78/-npm-install--g-http-server.git
cd -npm-install--g-http-server
npm install
```

### 4. Create Systemd Service
```bash
sudo tee /etc/systemd/system/http-server.service > /dev/null <<EOF
[Unit]
Description=HTTP Server
After=network.target

[Service]
Type=simple
User=$USER
WorkingDirectory=$(pwd)
ExecStart=$(which node) server.js
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable http-server.service
sudo systemctl start http-server.service
```

### 5. Setup Firewall
```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 8080/tcp
sudo ufw allow 443/tcp
sudo ufw --force enable
```

### 6. Install and Configure Nginx
```bash
sudo apt-get install -y nginx
sudo cp nginx.conf /etc/nginx/nginx.conf
sudo nginx -t
sudo systemctl restart nginx
```

## Docker Deployment (Alternative)

### 1. Install Docker
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
```

### 2. Deploy with Docker Compose
```bash
docker-compose up -d
```

### 3. View Logs
```bash
docker-compose logs -f http-server
```

## Monitoring and Management

### Check Service Status
```bash
sudo systemctl status http-server.service
```

### View Live Logs
```bash
sudo journalctl -u http-server.service -f
```

### Restart Service
```bash
sudo systemctl restart http-server.service
```

### Stop Service
```bash
sudo systemctl stop http-server.service
```

### View Nginx Logs
```bash
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log
```

## SSL/TLS with Let's Encrypt

### 1. Install Certbot
```bash
sudo apt-get install -y certbot python3-certbot-nginx
```

### 2. Get Certificate
```bash
sudo certbot --nginx -d your-domain.com
```

### 3. Auto-renewal
Certbot automatically sets up auto-renewal. Verify:
```bash
sudo systemctl status certbot.timer
```

## Performance Optimization

### Enable Gzip Compression
Already configured in nginx.conf

### Static File Caching
Already configured with 1-hour cache

### Connection Pooling
Handled by Nginx upstream configuration

## Troubleshooting

### Port 8080 Already in Use
```bash
lsof -i :8080
kill -9 <PID>
```

### Service Won't Start
```bash
sudo journalctl -u http-server.service -n 50
```

### Nginx Configuration Error
```bash
sudo nginx -t
```

### Check if Server is Running
```bash
curl http://localhost:8080
```

## Environment Variables

Add to `/etc/systemd/system/http-server.service`:
```
Environment="PORT=8080"
Environment="NODE_ENV=production"
```

## Backup Strategy

### Backup Application
```bash
tar -czf http-server-backup-$(date +%Y%m%d).tar.gz /opt/http-server/
```

### Backup Nginx Config
```bash
sudo tar -czf nginx-backup-$(date +%Y%m%d).tar.gz /etc/nginx/
```

## Support

For issues or questions, open an issue on GitHub.
