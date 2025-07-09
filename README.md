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
----------------

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


-----------
output
#### run app.py
 * Serving Flask app 'app'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.31.15.35:5000

Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 124-338-485

note-if we want to stop press "ctrl+c" to exit

------------------------------

Open a new console window from vm then switch to root. 
then run the commands below


5.Open port 5000 on the firewall:

sudo firewall-cmd --permanent --add-port=5000/tcp
sudo firewall-cmd --reload


###You should see it running at: http://<backend-ip>:5000/users

----------------
### API Methods -->> we need to conduct all activities after backend-to rds -connection establish ###



Updated Flask App with Full API Methods
This includes:

GET /users → Fetch all users
POST /users/add → Add a new user
GET /users/<id> → Fetch a single user by ID
PUT /users/update/<id> → Update a user's info
DELETE /users/delete/<id> → Delete a user

#### Post method ###   --->To add new user post method 

curl -X POST http://localhost:5000/users/add      -H "Content-Type: application/json"      -d '{"name":"Naveen", "email":"naveen@example.com"}'

[root@ip-172-31-15-35 ~]# curl -X POST http://localhost:5000/users/add      -H "Content-Type: application/json"      -d '{"name":"John DDoe", "email":"john@example.com"}'
{
  "message": "User added successfully"
}


#### Get method ###   --->To Fetch all users
curl -X GET http://localhost:5000/users 

[root@ip-172-31-15-35 ~]# curl -X GET http://localhost:5000/users
[
  {
    "email": "john@example.com",
    "id": 1,
    "name": "John DDoe"
  }
]


#### Get method ###  --->To Fetch single users
curl -X GET http://localhost:5000/users/1 
[root@ip-172-31-15-35 ~]# curl -X GET http://localhost:5000/users/1 
{
  "email": "john@example.com",
  "id": 1,
  "name": "John DDoe"
}



#### Put method ###  --->To update user post method 

curl -X PUT http://localhost:5000/users/update/1 \
     -H "Content-Type: application/json" \
     -d '{"name": "John Updated", "email": "john.updated@example.com"}'

[root@ip-172-31-15-35 ~]# curl -X PUT http://localhost:5000/users/update/1 \
     -H "Content-Type: application/json" \
     -d '{"name": "John Updated", "email": "john.updated@example.com"}'
{
  "message": "User updated successfully"
}


### how to check API request from external #########

step-1
https://web.postman.co/            ### signup with your credentials " git-hub or any other account

step-2
once to logged-in
workspace----->>>>new---->>>select http

step-3
enter url to check api activities
 
http://43.204.228.245:5000/users    ### to check all users

http://43.204.228.245:5000/users/1  ### to check user with id-1

http://43.204.228.245:5000/users/2  ### to check user with id-2   etcccc



### Delete method ### ----> To Delete user 

curl -X DELETE http://localhost:5000/users/delete/1


ps aux | grep app.py   # to check running or not python background


pkill -f app.py    # to kill process 


## To run continuously  background without stop

nohup python3 app.py > flask.log 2>&1 &


### How It Works

nohup prevents the process from stopping when you log out.
& runs it in the background.
> flask.log 2>&1 redirects output (stdout & stderr) to flask.log.

----------------


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





