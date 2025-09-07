# Nginx SSL Setup Guide

A comprehensive guide for setting up SSL/TLS certificates with Nginx, including commands and important configuration steps.

## Prerequisites

- Nginx installed and running
- Domain name pointed to your server
- Root/sudo access to the server
- Port 80 and 443 open in firewall

## Quick Setup Commands

### 1. Install Certbot (Let's Encrypt)

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install certbot python3-certbot-nginx

# CentOS/RHEL
sudo yum install certbot python3-certbot-nginx

# Or using snap (universal)
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

### 2. Obtain SSL Certificate

```bash
# For single domain
sudo certbot --nginx -d yourdomain.com

# For multiple domains
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Manual certificate only (without auto-configuration)
sudo certbot certonly --nginx -d yourdomain.com
```

### 3. Test Nginx Configuration

```bash
# Test configuration syntax
sudo nginx -t

# Reload Nginx safely
sudo systemctl reload nginx

# Restart Nginx if needed
sudo systemctl restart nginx
```

## Important Configuration Points

### SSL Configuration Block

Add this to your Nginx server block:

```nginx
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL Certificate paths
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";

    # SSL Security settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Your application configuration
    location / {
        # Your app configuration here
    }
}

# HTTP to HTTPS redirect
server {
    listen 80;
    listen [::]:80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$server_name$request_uri;
}
```

## Certificate Management

### Auto-renewal Setup

```bash
# Test renewal
sudo certbot renew --dry-run

# Check renewal timer (systemd)
sudo systemctl status certbot.timer

# Manual renewal
sudo certbot renew

# Setup cron job (if timer not available)
echo "0 12 * * * /usr/bin/certbot renew --quiet" | sudo crontab -
```

### Certificate Information

```bash
# List all certificates
sudo certbot certificates

# Check certificate expiry
sudo certbot certificates | grep -A 2 "Certificate Name"

# Check SSL with openssl
echo | openssl s_client -servername yourdomain.com -connect yourdomain.com:443 2>/dev/null | openssl x509 -noout -dates
```

## Security Best Practices

### 1. **Strong SSL Configuration**
- Use TLS 1.2 and 1.3 only
- Disable weak ciphers
- Enable HSTS headers

### 2. **Regular Updates**
```bash
# Update system packages
sudo apt update && sudo apt upgrade

# Update Nginx
sudo apt update nginx
```

### 3. **Firewall Configuration**
```bash
# UFW (Ubuntu)
sudo ufw allow 'Nginx Full'
sudo ufw delete allow 'Nginx HTTP'

# iptables
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

### 4. **Monitoring and Logs**
```bash
# Check Nginx error logs
sudo tail -f /var/log/nginx/error.log

# Check access logs
sudo tail -f /var/log/nginx/access.log

# Check SSL certificate status
sudo certbot certificates
```

## Troubleshooting

### Common Issues

1. **Port 80/443 blocked**
   ```bash
   sudo netstat -tlnp | grep :80
   sudo netstat -tlnp | grep :443
   ```

2. **DNS not propagated**
   ```bash
   nslookup yourdomain.com
   dig yourdomain.com
   ```

3. **Certificate renewal fails**
   ```bash
   sudo certbot renew --force-renewal
   sudo nginx -t && sudo systemctl reload nginx
   ```

4. **SSL test failed**
   - Use [SSL Labs Test](https://www.ssllabs.com/ssltest/)
   - Check certificate chain
   - Verify domain ownership

## File Locations

- **Nginx config**: `/etc/nginx/sites-available/default`
- **SSL certificates**: `/etc/letsencrypt/live/yourdomain.com/`
- **Nginx logs**: `/var/log/nginx/`
- **Certbot config**: `/etc/letsencrypt/`

## Additional Resources

- [Nginx SSL Documentation](https://nginx.org/en/docs/http/configuring_https_servers.html)
- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [SSL Configuration Generator](https://ssl-config.mozilla.org/)
- [SSL Labs Testing Tool](https://www.ssllabs.com/ssltest/)

---

**Note**: Always backup your configuration files before making changes and test in a staging environment first.
