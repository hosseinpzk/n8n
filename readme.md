1 - Add "A record to your host to point at your Ubuntu server's public IP"

~/n8n/
 ├── docker-compose.yml
 ├── .env
 └── data/

Create a directory like mentioned above. Files are available in this repo.

Then --> 

```bash
docker compose up -d
```

preparation NGINX

```bash
sudo apt install -y nginx certbot python3-certbot-nginx
sudo chown -R 1000:1000 ~/n8n/data
sudo nano /etc/nginx/sites-available/n8n
```

inside this --^ should be this:

server {
    server_name n8n.example.com;

    location / {
        proxy_pass http://127.0.0.1:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

ACTIVATION:

```bash
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```


Free SSL:

```bash
sudo certbot --nginx -d n8n.example.com
```


TEST ---->>>> 
```bash
https://n8n.example.com
```









