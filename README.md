## n8n Self-Hosted with Docker & Nginx

This guide helps you set up **n8n** on your own Ubuntu server with **Docker**, **Nginx**, and free SSL from **Let's Encrypt**.

---

## Requirements
- Ubuntu server with public IP
- Domain name pointing to your server (A record)
- Docker & Docker Compose installed

---

## Project Structure

Create a project folder for n8n:

```
~/n8n/
 ├── docker-compose.yml
 ├── .env
 └── data/
```

You can find example files in this repo.

---

## Run n8n with Docker Compose

From inside your project folder:

```bash
docker compose up -d
```

---

## Setup Nginx as Reverse Proxy

Install Nginx & Certbot:

```bash
sudo apt install -y nginx certbot python3-certbot-nginx
sudo chown -R 1000:1000 ~/n8n/data
```

Create a new Nginx config:

```bash
sudo nano /etc/nginx/sites-available/n8n
```

Paste this config (replace `n8n.example.com` with your domain):

```nginx
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
```

Enable the site and reload Nginx:

```bash
sudo ln -s /etc/nginx/sites-available/n8n /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Enable Free SSL with Let's Encrypt

Run:

```bash
sudo certbot --nginx -d n8n.example.com
```

Follow the prompts to get a free SSL certificate.

---

## ✅ Test Your Setup

Open your browser:

👉 [https://n8n.example.com](https://n8n.example.com)

You should see your n8n instance running with HTTPS enabled 🎉

---

## 🛠 Useful Commands

Restart services:

```bash
docker compose restart
```

Check logs:

```bash
docker compose logs -f
```

Stop services:

```bash
docker compose down
```

Upgrade:

1- Change image version in docker-compose.yaml

2- 

```bash
docker compose pull
docker compose up -d
```

---

## 📖 References
- [n8n Documentation](https://docs.n8n.io)
- [Docker Documentation](https://docs.docker.com)
- [Certbot Documentation](https://certbot.eff.org)
