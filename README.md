# nginx-database-learning

# Nginx Reverse Proxy Setup for Docker Application on Amazon Linux EC2

## 1. Installation

Install Nginx:
```
sudo amazon-linux-extras install nginx1
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

Configuration for HTTPS (assuming you have SSL certificates):
```nginx
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate /path/to/certificate.pem;
    ssl_certificate_key /path/to/certificate_key.pem;

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

## 5. SSL Certificates

For Let's Encrypt certificates:
1. Install Certbot:
   ```
   sudo amazon-linux-extras install epel
   sudo yum install certbot python2-certbot-nginx
   ```

2. Obtain and install certificate:
   ```
   sudo certbot --nginx -d yourdomain.com
   ```

## 6. Security Considerations

1. Configure firewall to allow HTTP (80) and HTTPS (443) traffic.
2. Regularly update Nginx and your system.
3. Use strong SSL settings (you can use Mozilla's SSL Configuration Generator).

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

Remember to replace `yourdomain.com` with your actual domain name and adjust paths and port numbers as necessary for your specific setup