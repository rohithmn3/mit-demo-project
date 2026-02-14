# 🚀 Demo Flask Application with Nginx and Systemd

This guide walks you through deploying a simple **Python Flask application** behind **Nginx** and optionally managing it with **systemd**.  
Ideal for EC2 instances or any Linux server.

---

## 📁 1. Create the Flask Application

Create the app directory and file:

```bash
sudo mkdir -p /opt/myapp
cd /opt/myapp
sudo nano app.py

