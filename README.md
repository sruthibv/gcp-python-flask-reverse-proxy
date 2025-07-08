# GCP Flask + MySQL REST API with HTML Frontend

This project demonstrates how to build a REST API using **Flask + MySQL**, host it on a **Google Cloud Platform CentOS VM**, and serve a frontend using **NGINX** from a separate GCP instance that communicates with the backend API.

---

## Features

- RESTful API (GET, POST) for managing `users`
- MySQL database integration
- GCP CentOS backend server
- NGINX-hosted HTML frontend server
- CORS enabled to allow frontend-backend communication
- Add & view users from the browser interface


CREATE A CLOUDSQL (DATABASE GCP)

Create a database with public-ip and add user ready.

------

Frontend (NGINX + HTML) ──> Backend API (Flask + MySQL) ──> MySQL DB
[GCP VM: IP A] [GCP VM: IP B]


##  Backend Setup (CentOS VM with Flask + MySQL)

### 1. SSH into your backend VM:

gcloud compute ssh backend-instance-name --zone <your-zone>

### 2. Install dependencies:

sudo yum update -y
sudo yum install -y python3 python3-pip git
pip3 install flask flask-cors mysql-connector-python

3. Create your Flask app (app.py):
   Go to /home/user
   sudo vi app.py
   copy paste the backend app.py file here. 

4. Run Flask App

   python3 app.py

5.Open port 5000 on the firewall:

sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --reload


###You should see it running at: http://<backend-ip>:5000/users

----------------

MySQL Configuration
Make sure your MySQL is running on the backend and includes the following table:


CREATE DATABASE dev;
USE dev;

CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255),
  email VARCHAR(255)
);


----

Frontend Setup (NGINX on CentOS)

1. SSH into frontend VM:

gcloud compute ssh frontend-instance-name --zone <your-zone>

2. Install NGINX:

sudo yum install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx

3. Edit NGINX HTML:
Place the final index.html inside the default web root:

sudo vi /usr/share/nginx/html/index.html
Paste the full HTML code (which fetches users and allows adding them).

4. Open port 80 on the frontend VM:

sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

Connecting Frontend with Backend
In index.html, replace:

const backendIP = "http://<YOUR_BACKEND_IP>:5000";
With your actual backend IP.

##Test
Visit your frontend IP in browser:
http://<frontend-ip>/

You should see a styled page that lists users and a form to add new ones.

Test adding users via the form — they should immediately appear below.

🛠️ Troubleshooting
CORS Error: Make sure CORS is enabled using flask_cors

Port Not Reachable: Check GCP firewall rules for both 5000 (backend) and 80 (frontend)

Flask App Crashing: Check Python syntax and module names like __name__ not name

📁 File Structure
pgsql
Copy
Edit
backend/
├── app.py
├── requirements.txt
└── users.sql

frontend/
└── index.html
📌 Notes
Backend Flask app should be run with debug=False in production

NGINX should only serve static frontend — not act as reverse proxy unless configured

Consider setting up supervisor or systemd to keep Flask app running in background

----------------

✅ Step-by-Step: Run Flask App with systemd
🔧 1. Create a systemd service file

sudo vi /etc/systemd/system/flaskapp.service
Paste the following (update paths and username accordingly):


[Unit]
Description=Flask App Service
After=network.target

[Service]
User=bsruthi1994
WorkingDirectory=/home/bsruthi1994
ExecStart=/usr/bin/python3 /home/bsruthi1994/app.py
Restart=always
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target

🔐 2. Set permissions (if needed)
Ensure the script is executable:


chmod +x /home/bsruthi1994/app.py

▶️ 3. Reload systemd and start your service

sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl start flaskapp

✅ 4. Enable service on boot

sudo systemctl enable flaskapp





