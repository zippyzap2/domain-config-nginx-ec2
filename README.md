# EC2 Domain Configuration with SSL

This document outlines the steps to configure a custom domain with an EC2 instance and enable SSL for secure HTTPS access.

## Prerequisites

1. **AWS EC2 Instance**:
   - A running EC2 instance with a public IP.
   - Nginx installed and serving your application.

2. **Custom Domain**:
   - A registered domain name (e.g., `weathertoday.us.to`).
   - Access to the domain's DNS settings.

3. **Access to the EC2 Instance**:
   - SSH access to the EC2 instance using a key pair.

4. **Certbot Installed**:
   - Installed `certbot` for SSL certificate setup.

## Steps to Configure Domain

### 1. Point Domain to EC2 Instance

1. **Login to your domain registrar** (e.g., Freedns).
2. **Update DNS Records**:
   - Add an A Record:
     - **Type**: `A`
     - **Name**: `@`
     - **Value**: `<Your_EC2_Public_IP>`
     - **TTL**: Default or 3600 seconds.
   - Add a CNAME Record (Optional):
     - **Type**: `CNAME`
     - **Name**: `www`
     - **Value**: `<yourdomain>` (e.g., `weathertoday.us.to`).
3. Save changes and wait for DNS propagation (can take a few minutes to a few hours).

   

### 2. Configure Nginx for Your Domain

1. SSH into the EC2 instance:
   ```bash
   ssh -i your-key.pem ubuntu@<Your_EC2_Public_IP>
   ```

2. Edit the Nginx configuration:
   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```

3. Update the configuration to include your domain:
   ```nginx
   server {
       listen 80;
       server_name weathertoday.us.to www.weathertoday.us.to;

       root /home/ubuntu/weather-app/frontend/build;
       index index.html;

       location / {
           try_files $uri /index.html;
       }
   }
   ```

4. Test and restart Nginx:
   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

5. Verify the configuration:
   - Open a browser and navigate to `http://weathertoday.us.to`.

### 3. Enable SSL with Let's Encrypt

1. Install Certbot and the Nginx plugin:
   ```bash
   sudo apt update
   sudo apt install certbot python3-certbot-nginx -y
   ```

2. Obtain and configure an SSL certificate:
   ```bash
   sudo certbot --nginx -d weathertoday.us.to -d www.weathertoday.us.to
   ```

3. Certbot will:
   - Verify your domain.
   - Automatically update Nginx to use HTTPS.

4. Test HTTPS:
   - Open a browser and navigate to `https://weathertoday.us.to`.

### 4. Redirect HTTP to HTTPS

1. Edit the Nginx configuration:
   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```

2. Add the following block to redirect HTTP traffic to HTTPS:
   ```nginx
   server {
       listen 80;
       server_name weathertoday.us.to www.weathertoday.us.to;
       return 301 https://$host$request_uri;
   }

   server {
       listen 443 ssl;
       server_name weathertoday.us.to www.weathertoday.us.to;

       ssl_certificate /etc/letsencrypt/live/weathertoday.us.to/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/weathertoday.us.to/privkey.pem;

       root /home/ubuntu/weather-app/frontend/build;
       index index.html;

       location / {
           try_files $uri /index.html;
       }
   }
   ```

3. Restart Nginx:
   ```bash
   sudo systemctl restart nginx
   ```

### 5. Automate SSL Renewal

1. Test automatic renewal:
   ```bash
   sudo certbot renew --dry-run
   ```

2. Set up a cron job to renew certificates automatically:
   ```bash
   sudo crontab -e
   ```

3. Add the following line:
   ```plaintext
   0 0 * * * certbot renew --quiet && systemctl reload nginx
   ```

## Troubleshooting

1. **Check Nginx Logs**:
   ```bash
   sudo tail -f /var/log/nginx/error.log
   ```

2. **Verify DNS Propagation**:
   Use tools like [DNS Checker](https://dnschecker.org) to ensure your domain points to your EC2 instance.

3. **Check Certbot Logs**:
   ```bash
   sudo tail -f /var/log/letsencrypt/letsencrypt.log
   ```

4. **Ping Your Domain**:
   ```bash
   ping weathertoday.us.to
   ```

## Summary

You’ve successfully configured your custom domain `weathertoday.us.to` to point to your EC2 instance and enabled HTTPS with a free SSL certificate from Let's Encrypt. Your website is now secure and accessible over HTTPS.

---

For additional questions or issues, feel free to reach out!
