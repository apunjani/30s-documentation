# Nginx Reverse Proxy Setup Guide

This guide provides step-by-step instructions for setting up Nginx as a reverse proxy on Ubuntu Server to route traffic to multiple backend applications running on different ports.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Server Setup](#server-setup)
- [Nginx Installation and Configuration](#nginx-installation-and-configuration)
- [SSL Certificate Setup](#ssl-certificate-setup)
- [Backend Application Management](#backend-application-management)
- [Testing and Verification](#testing-and-verification)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)

## Prerequisites

- Ubuntu Server (18.04 or later)
- Domain name pointed to your server IP
- Root or sudo access
- Basic knowledge of command line operations

## Server Setup

### 1. Update System Packages

```bash
# Update package lists and upgrade system
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y python3 python3-pip python3-venv nginx certbot python3-certbot-nginx htop git curl ufw
```

### 2. Configure Firewall (UFW)

```bash
# Enable firewall
sudo ufw enable

# Allow essential ports
sudo ufw allow 22/tcp      # SSH
sudo ufw allow 80/tcp      # HTTP
sudo ufw allow 443/tcp     # HTTPS

# Check firewall status
sudo ufw status
```

## Nginx Installation and Configuration

### 1. Remove Default Configuration

```bash
# Remove default nginx site
sudo rm -f /etc/nginx/sites-enabled/default
```

### 2. Create Your Site Configuration

Create a new configuration file:

```bash
sudo nano /etc/nginx/sites-available/your-domain.com
```

Add the following configuration (replace `your-domain.com` and port numbers as needed):

```nginx
server {
    listen 80;
    server_name your-domain.com;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;

    # Root endpoint - API status page
    location = / {
        return 200 '{"status": "API Gateway", "available_endpoints": ["/api-v1", "/service-a", "/service-b", "/service-c"]}';
        add_header Content-Type application/json;
    }

    # Service 1 -> Port 4044
    location /api-v1 {
        proxy_pass http://127.0.0.1:4044;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Service 2 -> Port 9696
    location /service-a {
        proxy_pass http://127.0.0.1:9696;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Service 3 -> Port 8070
    location /service-b {
        proxy_pass http://127.0.0.1:8070;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Service 4 -> Port 8000
    location /service-c {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Handle undefined endpoints
    location / {
        return 404 '{"error": "Endpoint not found", "available_endpoints": ["/api-v1", "/service-a", "/service-b", "/service-c"]}';
        add_header Content-Type application/json;
    }
}
```

### 3. Enable the Site

```bash
# Create symbolic link to enable the site
sudo ln -s /etc/nginx/sites-available/your-domain.com /etc/nginx/sites-enabled/

# Test nginx configuration
sudo nginx -t

# Restart nginx
sudo systemctl restart nginx
sudo systemctl enable nginx
```

## SSL Certificate Setup

### 1. Install SSL Certificate with Let's Encrypt

```bash
# Get SSL certificate and auto-configure nginx
sudo certbot --nginx -d your-domain.com
```

Follow the prompts:
- Enter your email address
- Agree to terms of service (A)
- Choose whether to share email with EFF (Y/N)
- Choose to redirect HTTP to HTTPS (option 2 - recommended)

### 2. Verify SSL Configuration

```bash
# Test HTTPS access
curl https://your-domain.com/

# Check certificate auto-renewal
sudo systemctl status certbot.timer
```


## Testing and Verification

### 1. Test Nginx Configuration

```bash
# Test configuration syntax
sudo nginx -t

# Check nginx status
sudo systemctl status nginx

# View nginx logs
sudo tail -f /var/log/nginx/error.log
```

### 2. Test Endpoints

```bash
# Test root endpoint
curl https://your-domain.com/

# Test individual services
curl https://your-domain.com/api-v1
curl https://your-domain.com/service-a
curl https://your-domain.com/service-b
curl https://your-domain.com/service-c
```

### 3. Check Service Status

```bash
# Check all application services
sudo systemctl status app-4044 app-9696 app-8070 app-8000

# View application logs
sudo journalctl -u app-4044 -f
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Connection Refused Error
- **Check AWS/Cloud Security Groups**: Ensure ports 80 and 443 are open
- **Verify nginx is running**: `sudo systemctl status nginx`
- **Check firewall**: `sudo ufw status`

#### 2. 502 Bad Gateway
- **Backend service not running**: Check application services
- **Port conflicts**: Verify applications are listening on correct ports
- **Check logs**: `sudo tail -f /var/log/nginx/error.log`

#### 3. SSL Certificate Issues
- **Renew certificate**: `sudo certbot renew`
- **Check certificate status**: `sudo certbot certificates`

#### 4. Configuration Errors
```bash
# Test nginx configuration
sudo nginx -t

# Reload configuration
sudo systemctl reload nginx
```


## Security Considerations

### 1. Firewall Configuration
- Only open necessary ports (22, 80, 443)
- Consider restricting SSH access to specific IPs
- Use fail2ban for additional protection

### 2. Nginx Security Headers
The configuration includes basic security headers. Consider adding:
```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

### 3. Application Security
- Run applications with non-root users
- Use environment variables for sensitive configuration
- Implement proper authentication and authorization
- Regular security updates

### 4. SSL Best Practices
- Use strong SSL configurations
- Enable OCSP stapling
- Regular certificate renewals (automated with certbot)

## Customization Notes

### Port Mapping
Update the following in your configuration:
- Replace `your-domain.com` with your actual domain
- Change port numbers (4044, 9696, 8070, 8000) to match your applications
- Modify endpoint paths (`/api-v1`, `/service-a`, etc.) as needed

### Load Balancing
For high-traffic applications, consider adding upstream blocks:
```nginx
upstream backend_service {
    server 127.0.0.1:4044;
    server 127.0.0.1:4045;  # Additional instances
}

location /api-v1 {
    proxy_pass http://backend_service;
    # ... other proxy settings
}
```
