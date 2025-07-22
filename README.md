## Langkah install server EC2 (Amazon Linux)
a. Buat nama server

b. Pilih OS Amazon Linux

c. Buat key_pair (jika belum punya key-nya)

d. Pilih security group nya default
  - add inbound rules
  - HTTP, HTTPS, SSH, Custom TCP port 5000
  - Source anywhere IPv4

e. Klik launch instance

## Membuat RDS
a. Pilih "easy create" dan pilih "MySql"

b. Buat nama dan databasenya dan buat username dan password

c. Set up EC2 connection, pilih server EC2 kita yang dibuat tadi

## Connect SSH key
a. Buka cmd, masukkan ssh key yang didapat pada server EC2 kita. Example: ssh -i "sigap-key.pem" ec2-user@ec2-3-25-114-113.ap-southeast-2.compute.amazonaws.com

## Install PHP, Python3, PIP, Flask, beserta modul PHP
```bash
sudo yum update -y
sudo yum install httpd php php-mysqli php-common php-xml php-zip
sudo yum install python3 -y
sudo yum install python3-pip -y
pip3 install flask pymysql
```

### Mengubah kepemilikan directory /var/www/html
```bash
sudo chown -R ec2-user:ec2-user /var/www/html
sudo chmod -R 775 /var/www/html
```

```bash
sudo systemctl enable httpd
sudo systemctl start httpd
sudo systemctl status httpd
```

## Langkah install phpmyadmin
```bash
sudo wget https://www.phpmyadmin.net/downloads/phpMyAdmin-latest-all-languages.tar.gz
```
```bash
sudo mkdir /var/www/html/phpmyadmin
```
```bash
sudo tar xvf phpMyAdmin-latest-all-languages.tar.gz --strip-components=1 -C /var/www/html/phpmyadmin
```
``` bash
sudo cp /var/www/html/phpmyadmin/config.sample.inc.php /var/www/html/phpmyadmin/config.inc.php
```
``` bash
sudo nano /var/www/html/phpmyadmin/config.inc.php
```
``` bash
$cfg['blowfish_secret'] = 'My_Secret_Passphras3!';
```
``` bash
sudo chmod 775 /var/www/html/phpmyadmin/config.inc.php
```
``` bash
sudo chown -R ec2-user:ec2-user /var/www/html/phpmyadmin
```

## Buat directory untuk file
```bash
mkdir ~/flask-register
sudo chown -R ec2-user:ec2-user flask-register
chmod -R 755 flask-register
cd ~/flask-register
ls -l
```

## Menambahkan file
cd ~/falsk-register
nano .py

register.py
```bash
from flask import request
import pymysql

DB_CONFIG = {
    "host": "database-2.cjqs0i4iie68.ap-southeast-2.rds.amazonaws.com",
    "user": "root",
    "password": "admin123#",
    "database": "sigap_db"
}

def register():
    nama = request.form.get('nama')
    email = request.form.get('email')
    no_hp = request.form.get('no_hp')
    password = request.form.get('password')

    if not nama or not email or not no_hp or not password:
        return "Semua field harus diisi", 400

    try:
        conn = pymysql.connect(**DB_CONFIG)
        cursor = conn.cursor()

        cursor.execute("SELECT id FROM users WHERE email = %s", (email,))
        if cursor.fetchone():
            return "Email sudah terdaftar", 409

        cursor.execute("""
            INSERT INTO users (nama, email, no_hp, password, role)
            VALUES (%s, %s, %s, %s, 'user')
        """, (nama, email, no_hp, password))
        conn.commit()
        return "Pendaftaran berhasil", 200

    except Exception as e:
        return f"Database error: {str(e)}", 500

    finally:
        cursor.close()
        conn.close()
```

login.py
```bash
from flask import request
import pymysql

DB_CONFIG = {
    "host": "database-2.cjqs0i4iie68.ap-southeast-2.rds.amazonaws.com",
    "user": "root",
    "password": "admin123#",
    "database": "sigap_db"
}

def login():
    nama = request.form.get('nama')
    password = request.form.get('password')

    if not nama or not password:
        return "Nama dan password harus diisi", 400

    try:
        conn = pymysql.connect(**DB_CONFIG)
        cursor = conn.cursor()

        cursor.execute("SELECT role FROM users WHERE nama = %s AND password = %s", (nama, password))
        result = cursor.fetchone()

        if result:
            return result[0]  # misalnya 'admin' atau 'user'
        else:
            return "invalid", 401

    except Exception as e:
        return f"Database error: {str(e)}", 500

    finally:
        cursor.close()
        conn.close()
```

app.py
```bash
from flask import Flask
from register import register
from login import login

app = Flask(__name__)

@app.route('/register', methods=['POST'])
def handle_register():
    return register()

@app.route('/login', methods=['POST'])
def handle_login():
    return login()

if __name__ == '__main__':
    app.run(debug=True, host="0.0.0.0", port=5000)
```

## Buat flask service
```bash
sudo nano /etc/systemd/system/flask-register.service
```

### Masukkan ini sesuaikan
```bash
[Unit]
Description=Flask Register Service
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/home/ec2-user/flask-register
ExecStart=/usr/bin/python3 register.py
Restart=always

[Install]
WantedBy=multi-user.target
```
cd ~/flask-register
pwd

hasil: /home/ec2-user/flask-register

WorkingDirectory=/home/ec2-user/flask-register (directory file kita)

which python3

hasil: /usr/bin/python3

ExecStart=/usr/bin/python3 app.py

## Jalankan flask dan enable
```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload


sudo systemctl start flask-register
sudo systemctl enable flask-register
```
```bash
sudo systemctl status flask-register
```
hasil: Active: active (running)

### Jalankan di directory file
```bash
python3 app.py
```
