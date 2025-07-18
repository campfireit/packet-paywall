# AWS EC2 Deployment Guide - Packet Paywall Application

This comprehensive guide will walk you through deploying the Packet Paywall application on AWS EC2 using a t2.micro instance for testing purposes.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [AWS EC2 Setup](#aws-ec2-setup)
3. [Security Groups Configuration](#security-groups-configuration)
4. [Server Setup](#server-setup)
5. [Application Deployment](#application-deployment)
6. [Database Configuration](#database-configuration)
7. [Environment Configuration](#environment-configuration)
8. [Process Management with PM2](#process-management-with-pm2)
9. [SSL/Domain Setup](#ssldomain-setup)
10. [Monitoring and Maintenance](#monitoring-and-maintenance)
11. [Cost Optimization](#cost-optimization)
12. [Troubleshooting](#troubleshooting)

## Prerequisites

Before starting, ensure you have:
- An active AWS account with billing configured
- Basic knowledge of Linux command line
- SSH client installed on your local machine
- Domain name (optional, for SSL setup)
- Stripe account for payment processing
- UniFi Controller access credentials

## AWS EC2 Setup

### Step 1: Launch EC2 Instance

1. **Access AWS Management Console**
   - Log in to [AWS Management Console](https://aws.amazon.com/console/)
   - Navigate to **Services > EC2**

2. **Launch Instance**
   - Click **Launch Instance**
   - Name your instance: `packet-paywall-server`

3. **Choose AMI**
   - Select **Ubuntu Server 22.04 LTS (HVM), SSD Volume Type**
   - Architecture: 64-bit (x86)

4. **Select Instance Type**
   - Choose **t2.micro** (Free Tier eligible)
   - 1 vCPU, 1 GB RAM, Low to Moderate network performance

5. **Key Pair Configuration**
   - Create a new key pair or use existing one
   - Key pair type: **ED25519** (recommended for better security)
   - Download the `.pem` file and store it securely
   - Set proper permissions:
     ```bash
     chmod 400 your-key.pem
     ```

6. **Storage Configuration**
   - Root volume: **30 GB gp3** (Free Tier eligible up to 30GB)
   - Delete on termination: **Yes**

### Step 2: Configure Instance Details

- **Number of instances**: 1
- **Network**: Default VPC
- **Subnet**: Default subnet (or choose specific AZ)
- **Auto-assign Public IP**: Enable
- **IAM role**: None (for basic setup)

## Security Groups Configuration

### Step 3: Create Security Group

Create a new security group with the following rules:

**Inbound Rules:**
```
Type            Protocol    Port Range    Source          Description
SSH             TCP         22            Your IP/32      SSH access
HTTP            TCP         80            0.0.0.0/0       Web traffic
HTTPS           TCP         443           0.0.0.0/0       Secure web traffic
Custom TCP      TCP         3000          0.0.0.0/0       Next.js dev (temporary)
PostgreSQL      TCP         5432          10.0.0.0/8      Database (internal only)
```

**Outbound Rules:**
```
Type            Protocol    Port Range    Destination     Description
All traffic     All         All           0.0.0.0/0       All outbound traffic
```

**Security Best Practices:**
- Restrict SSH access to your IP only
- Remove port 3000 rule after setting up reverse proxy
- Use VPC security groups for database access
- Regularly review and update security group rules

## Server Setup

### Step 4: Connect to Your Instance

```bash
ssh -i your-key.pem ubuntu@<public-ip-address>
```

### Step 5: Initial Server Configuration

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y
sudo apt dist-upgrade -y && sudo apt autoremove -y

# Install essential packages
sudo apt install -y curl wget git unzip software-properties-common \
    ca-certificates gnupg lsb-release build-essential

# Set up firewall
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw allow 80,443/tcp
sudo ufw --force enable

# Set timezone (adjust as needed)
sudo timedatectl set-timezone UTC
```

### Step 6: Install Node.js 20 LTS

```bash
# Add NodeSource repository
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | \
    sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main" | \
    sudo tee /etc/apt/sources.list.d/nodesource.list

# Update and install Node.js
sudo apt update
sudo apt install -y nodejs

# Verify installation
node -v  # Should show v20.x.x
npm -v   # Should show 10.x.x
```

### Step 7: Install PostgreSQL 15

```bash
# Add PostgreSQL official repository
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" | \
    sudo tee /etc/apt/sources.list.d/postgresql.list

# Update and install PostgreSQL
sudo apt update
sudo apt install -y postgresql-15 postgresql-client-15

# Start and enable PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Verify installation
sudo systemctl status postgresql
```

## Application Deployment

### Step 8: Clone and Setup Application

```bash
# Create application directory
sudo mkdir -p /var/www
sudo chown ubuntu:ubuntu /var/www

# Clone the repository
cd /var/www
git clone https://github.com/campfireit/packet-paywall.git
cd packet-paywall

# Install dependencies
cd app
npm install

# Build the application
npm run build
```

## Database Configuration

### Step 9: Configure PostgreSQL

```bash
# Switch to postgres user and create database
sudo -u postgres psql << EOF
CREATE DATABASE packet_paywall;
CREATE USER packet_paywall_user WITH ENCRYPTED PASSWORD 'your_secure_password';
GRANT ALL PRIVILEGES ON DATABASE packet_paywall TO packet_paywall_user;
ALTER USER packet_paywall_user CREATEDB;
\q
EOF

# Configure PostgreSQL for local connections
sudo nano /etc/postgresql/15/main/pg_hba.conf
```

Add this line before other rules:
```
local   packet_paywall    packet_paywall_user                     md5
```

```bash
# Restart PostgreSQL
sudo systemctl restart postgresql

# Test database connection
psql -U packet_paywall_user -d packet_paywall -h localhost
```

### Step 10: Run Database Migrations

```bash
cd /var/www/packet-paywall/app

# Generate Prisma client
npx prisma generate

# Run database migrations
npx prisma db push

# Seed database (if seed script exists)
npm run seed
```

## Environment Configuration

### Step 11: Create Production Environment File

```bash
cd /var/www/packet-paywall
cp .env.example .env
nano .env
```

Configure the following variables:

```bash
# Database Configuration
DATABASE_URL="postgresql://packet_paywall_user:your_secure_password@localhost:5432/packet_paywall"

# NextAuth Configuration
NEXTAUTH_URL="https://yourdomain.com"  # or http://your-ec2-ip for testing
NEXTAUTH_SECRET="generate-a-secure-random-string-here"

# Stripe Payment Configuration
STRIPE_PUBLISHABLE_KEY="pk_live_your_stripe_publishable_key"
STRIPE_SECRET_KEY="sk_live_your_stripe_secret_key"
STRIPE_WEBHOOK_SECRET="whsec_your_stripe_webhook_secret"

# UniFi Controller Configuration
UNIFI_CONTROLLER_URL="https://your-unifi-controller.local:8443"
UNIFI_USERNAME="admin"
UNIFI_PASSWORD="your-unifi-password"
UNIFI_SITE_ID="default"

# Production Settings
LOG_LEVEL="info"
NODE_ENV="production"
```

## Process Management with PM2

### Step 12: Setup PM2

```bash
# Install PM2 globally
sudo npm install -g pm2

# Create PM2 configuration
cd /var/www/packet-paywall
nano ecosystem.config.js
```

```javascript
module.exports = {
  apps: [
    {
      name: 'packet-paywall',
      script: 'app/node_modules/next/dist/bin/next',
      args: 'start',
      cwd: '/var/www/packet-paywall/app',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3000
      },
      error_file: '/var/log/packet-paywall/error.log',
      out_file: '/var/log/packet-paywall/out.log',
      log_file: '/var/log/packet-paywall/combined.log',
      time: true,
      max_memory_restart: '500M'
    }
  ]
};
```

```bash
# Start the application
pm2 start ecosystem.config.js

# Save PM2 configuration
pm2 save

# Setup PM2 to start on boot
pm2 startup
```

## SSL/Domain Setup

### Step 13: Configure Nginx and SSL

```bash
# Install Nginx
sudo apt install -y nginx

# Install Certbot for SSL
sudo apt install -y certbot python3-certbot-nginx

# Create Nginx configuration
sudo nano /etc/nginx/sites-available/packet-paywall
```

```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```bash
# Enable the site
sudo ln -s /etc/nginx/sites-available/packet-paywall /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# Get SSL certificate
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
```

## Monitoring and Maintenance

### Step 14: Setup Monitoring

```bash
# Monitor PM2 processes
pm2 status
pm2 logs packet-paywall
pm2 monit

# Monitor system resources
htop
df -h
free -h

# Check application logs
tail -f /var/log/packet-paywall/combined.log
```

## Troubleshooting

### Common Issues

1. **Application won't start**
   - Check PM2 logs: `pm2 logs packet-paywall`
   - Verify environment variables
   - Check database connection

2. **Database connection issues**
   - Verify PostgreSQL is running: `sudo systemctl status postgresql`
   - Check database credentials
   - Test connection manually

3. **SSL certificate issues**
   - Verify domain DNS settings
   - Check Nginx configuration
   - Renew certificates: `sudo certbot renew`

4. **Performance issues**
   - Monitor system resources
   - Check PM2 cluster mode
   - Optimize database queries

### Useful Commands

```bash
# Restart application
pm2 restart packet-paywall

# Update application
cd /var/www/packet-paywall
git pull origin main
cd app
npm install
npm run build
pm2 restart packet-paywall

# Backup database
pg_dump -U packet_paywall_user -h localhost packet_paywall > backup.sql

# Restore database
psql -U packet_paywall_user -h localhost packet_paywall < backup.sql
```

## Cost Optimization

- Use t2.micro for development/testing (Free Tier eligible)
- Monitor usage with AWS CloudWatch
- Set up billing alerts
- Consider Reserved Instances for production
- Use AWS RDS for managed database in production

## Security Best Practices

- Keep system packages updated
- Use strong passwords and SSH keys
- Configure firewall properly
- Enable fail2ban for intrusion prevention
- Regular security audits
- Monitor logs for suspicious activity

---

For additional support, please refer to the main README.md or create an issue in the GitHub repository.
