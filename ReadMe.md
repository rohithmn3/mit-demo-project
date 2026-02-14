# 🚀 Demo Flask Application with Nginx and Systemd

This guide walks through deploying a simple **Python Flask application** behind **Nginx**, managed via **systemd**, and optionally exposed through an **AWS Application Load Balancer**.

---

## 📁 1. Create the Flask Application

```bash
sudo mkdir -p /opt/myapp
cd /opt/myapp
sudo nano app.py
```

### `app.py`

```python
from flask import Flask, jsonify, render_template_string

app = Flask(__name__)

HTML_TEMPLATE = """
<!DOCTYPE html>
<html>
<head>
    <title>My Demo Application</title>
    <style>
        body { 
            font-family: Arial, sans-serif; 
            max-width: 800px; 
            margin: 50px auto; 
            padding: 20px;
            background-color: #f5f5f5;
        }
        .container {
            background-color: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 { color: #333; }
        .status { 
            background-color: #d4edda; 
            color: #155724; 
            padding: 10px; 
            border-radius: 5px; 
            margin: 20px 0;
        }
        .info {
            background-color: #e7f3ff;
            padding: 15px;
            border-radius: 5px;
            margin: 20px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🚀 My Demo Application</h1>
        <div class="status">
            ✅ Application is running successfully!
        </div>
        <div class="info">
            <h3>Available Endpoints:</h3>
            <ul>
                <li><strong>/</strong> - Main page</li>
                <li><strong>/health</strong> - Health check</li>
                <li><strong>/api/status</strong> - JSON status</li>
                <li><strong>/api/info</strong> - App info</li>
            </ul>
        </div>
        <p>This Flask application is running behind nginx and AWS ALB.</p>
        <p><em>Powered by Flask 🐍</em></p>
    </div>
</body>
</html>
"""

@app.route('/')
def home():
    return render_template_string(HTML_TEMPLATE)

@app.route('/health')
def health():
    return '<h1>Ship It: App is Healthy</h1>'

@app.route('/api/status')
def api_status():
    return jsonify({
        'status': 'healthy',
        'message': 'Application is running',
        'version': '1.0.0'
    })

@app.route('/api/info')
def api_info():
    return jsonify({
        'application': 'Demo Flask App',
        'framework': 'Flask',
        'language': 'Python',
        'endpoints': ['/', '/health', '/api/status', '/api/info']
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=False)
```

---

## 📦 2. Install Dependencies

```bash
sudo yum update -y
sudo yum install -y python3 python3-pip
sudo pip3 install flask
```

Run the app:

```bash
cd /opt/myapp
python3 app.py
```

Test locally:

```bash
curl http://localhost:5000
curl http://localhost:5000/health
```

---

## ⚙️ 3. Create Systemd Service

```bash
sudo nano /etc/systemd/system/myapp.service
```

### `myapp.service`

```ini
[Unit]
Description=My Demo Flask Application
After=network.target

[Service]
Type=simple
User=ec2-user
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp
sudo systemctl status myapp
```

---

## 🌐 4. Configure Nginx

```bash
sudo nano /etc/nginx/conf.d/default.conf
```

```nginx
server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect off;
    }

    location /static/ {
        alias /opt/myapp/static/;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
}
```

Apply config:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🧹 5. Optional Cleanup

```bash
sudo mv /usr/share/nginx/html/index.html /usr/share/nginx/html/index.html.backup
sudo mv /usr/share/nginx/html/health /usr/share/nginx/html/health.backup
```

---

## 🧪 6. Test Everything

Local:

```bash
curl http://localhost/
curl http://localhost/health
curl http://localhost/api/status
```

Through Load Balancer:

```
http://YOUR-ALB-DNS/
http://YOUR-ALB-DNS/health
http://YOUR-ALB-DNS/api/status
```

---

## 📌 Endpoints

| Endpoint | Description |
|---------|------------|
| `/` | Main page |
| `/health` | Health check |
| `/api/status` | JSON status |
| `/api/info` | App info |

---

## ✅ Result

- Flask app running on port **5000**
- Managed via **systemd**
- Served through **Nginx**
- Ready behind **AWS ALB**
