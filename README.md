# nginx-database-learning

# Nginx Reverse Proxy Setup for Docker Application on Amazon Linux EC2

## 1. Installation

Install Nginx:
```
sudo yum update -y
sudo yum install nginx -y
```

Start Nginx and enable it to start on boot:
```
sudo systemctl start nginx
sudo systemctl enable nginx
```

## 2. Configuration

Create a new configuration file for your domain:
```
sudo nano /etc/nginx/conf.d/yourdomain.conf
```

Basic configuration for HTTP:
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:3080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

## 3. Testing and Reloading

Test Nginx configuration:
```
sudo nginx -t
```

Reload Nginx to apply changes:
```
sudo systemctl reload nginx
```

## 4. Troubleshooting

Check Nginx error logs:
```
sudo tail -f /var/log/nginx/error.log
```

Check if Nginx is listening on correct ports:
```
sudo ss -tulnp | grep nginx
```

Verify that conf.d is included in main config:
```
sudo cat /etc/nginx/nginx.conf | grep include
```

## 5. SSL Certificates (Let's Encrypt)

1. Install Certbot:
   ```
   sudo yum install certbot python3-certbot-nginx -y
   ```

2. Obtain and install certificate:
   ```
   sudo certbot --nginx -d yourdomain.com
   ```

3. If the above fails, try standalone mode:
   ```
   sudo systemctl stop nginx
   sudo certbot certonly --standalone -d yourdomain.com
   sudo systemctl start nginx
   ```

4. Configure Nginx for HTTPS:
   ```
   sudo nano /etc/nginx/conf.d/yourdomain.conf
   ```

   Use this configuration:
   ```nginx
   server {
       listen 80;
       server_name yourdomain.com;
       return 301 https://$server_name$request_uri;
   }

   server {
       listen 443 ssl http2;
       server_name yourdomain.com;

       ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

       ssl_protocols TLSv1.2 TLSv1.3;
       ssl_prefer_server_ciphers on;
       ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

       add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

       location / {
           proxy_pass http://localhost:3080;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

5. Test and reload Nginx:
   ```
   sudo nginx -t
   sudo systemctl reload nginx
   ```

## 6. Security Considerations

1. Configure EC2 security group to allow HTTP (80) and HTTPS (443) traffic.
2. Regularly update Nginx and your system:
   ```
   sudo yum update -y
   ```
3. Use strong SSL settings (as provided in the HTTPS configuration above).

## 7. Common Issues and Solutions

1. 502 Bad Gateway: Check if your Docker application is running and accessible on the specified port.
2. Unable to connect: Ensure EC2 security group allows inbound traffic on ports 80 and 443.
3. SSL certificate issues: Verify certificate paths and permissions.

## 8. Useful Commands

- Start Nginx: `sudo systemctl start nginx`
- Stop Nginx: `sudo systemctl stop nginx`
- Restart Nginx: `sudo systemctl restart nginx`
- Reload Nginx configuration: `sudo systemctl reload nginx`
- Check Nginx status: `sudo systemctl status nginx`
- View Nginx configuration: `sudo nano /etc/nginx/nginx.conf`
- List Nginx config files: `ls /etc/nginx/conf.d/`
- Check Certbot renewal: `sudo certbot renew --dry-run`
- Force certificate renewal: `sudo certbot renew --force-renewal`

Remember to replace `yourdomain.com` with your actual domain name and adjust paths and port numbers as necessary for your specific setup.

This updated README now includes the complete process for installing and configuring SSL certificates, as well as additional useful commands for managing Nginx and Certbot.