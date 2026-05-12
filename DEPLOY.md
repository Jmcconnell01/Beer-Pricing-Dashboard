# Beer Pricing Dashboard — Deployment Guide

## Files needed on your server
```
BeerPricing_Dashboard.py
survey_results.csv          ← keep in the same folder
requirements.txt
.streamlit/secrets.toml     ← set your password here
```

---

## Option A — Streamlit Community Cloud (Free, easiest)

1. Create a **private** GitHub repository
2. Upload `BeerPricing_Dashboard.py`, `survey_results.csv`, and `requirements.txt`
3. Go to [share.streamlit.io](https://share.streamlit.io) → **New app**
4. Connect your repo, set the main file to `BeerPricing_Dashboard.py`
5. Under **Advanced settings → Secrets**, add:
   ```toml
   DASHBOARD_PASSWORD = "your-chosen-password"
   ```
6. Deploy — share the URL only with your team

---

## Option B — VPS / Cloud Server (DigitalOcean, AWS, etc.)

### 1. Install Python & dependencies
```bash
sudo apt update && sudo apt install python3-pip -y
pip install -r requirements.txt
```

### 2. Set your password
Create `.streamlit/secrets.toml` in the same folder:
```toml
DASHBOARD_PASSWORD = "your-chosen-password"
```
Or set an environment variable:
```bash
export DASHBOARD_PASSWORD="your-chosen-password"
```

### 3. Run the app
```bash
streamlit run BeerPricing_Dashboard.py --server.port 8501
```

### 4. Keep it running (systemd service)
Create `/etc/systemd/system/beerdash.service`:
```ini
[Unit]
Description=Beer Pricing Dashboard
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/beerdash
ExecStart=streamlit run BeerPricing_Dashboard.py --server.port 8501
Restart=always
Environment=DASHBOARD_PASSWORD=your-chosen-password

[Install]
WantedBy=multi-user.target
```
Then:
```bash
sudo systemctl enable beerdash
sudo systemctl start beerdash
```

### 5. Add HTTPS with Nginx (recommended)
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://localhost:8501;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```
Use [Certbot](https://certbot.eff.org/) for a free SSL certificate.

---

## Changing the password

| Method | How |
|---|---|
| Streamlit Cloud | Dashboard → App settings → Secrets → update `DASHBOARD_PASSWORD` |
| VPS secrets.toml | Edit `.streamlit/secrets.toml`, restart the app |
| VPS environment | Update `DASHBOARD_PASSWORD=` in the systemd service, `sudo systemctl restart beerdash` |

**Default password if none is set:** `SharpenTheEdge2025`
Change this before going live.
