---

# 🚀 EC2 Instance Setup for WordPress & Node.js App

This guide will help you set up a WordPress site and a Node.js app on an Amazon Linux EC2 instance.

## 🖥️ EC2 Instance Details

1. **Instance Type**: T2.micro
2. **Key Pair**: `ec2-wordpress.pem`
3. **Storage**: 20 GB
4. **Allowed Inbound Rules**:
   - Port 80: HTTP (WordPress)
   - Port 3000: Node.js
   - Port 22: SSH

---

## 🔐 Log into EC2

```bash
ssh -i ec2-wordpress.pem ec2-user@<EC2_PUBLIC_IP>
```

---

## ⚙️ Install Required Services

1. **Node.js Installation**:

   ```bash
   sudo yum install nodejs
   node -v
   npm -v
   ```

2. **MySQL Installation**:

   Enable the MySQL 5.7 repository and install MySQL:

   ```bash
   sudo yum-config-manager --enable mysql57-community
   sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
   sudo yum install -y mysql-community-server
   sudo systemctl start mysqld
   ```

   Secure the MySQL installation:

   ```bash
   sudo grep 'temporary password' /var/log/mysqld.log
   sudo mysql_secure_installation
   ```

3. **PHP & Apache Installation**:

   Install PHP and necessary extensions for WordPress:

   ```bash
   sudo yum install -y php php-mysqlnd php-fpm php-json php-xml php-gd php-curl
   sudo systemctl restart httpd
   ```

4. **Ensure Apache is enabled on boot**:

   ```bash
   sudo systemctl is-enabled httpd
   ```

---

## 🌐 WordPress Setup

1. **Download and Extract WordPress**:

   ```bash
   cd /var/www/html
   sudo wget https://wordpress.org/latest.tar.gz
   sudo tar -xzf latest.tar.gz
   sudo chown -R apache:apache /var/www/html/wordpress
   ```

2. **Configure WordPress**:

   ```bash
   cd wordpress
   sudo cp wp-config-sample.php wp-config.php
   sudo nano wp-config.php
   ```

3. **Create a MySQL Database for WordPress**:

   ```sql
   CREATE DATABASE wordpress_db; 
   CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'xxxxxxxx';
   GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```

4. **Update `wp-config.php`**:

   ```php
   define('DB_NAME', 'wordpress_db');          // The name of the database you created
   define('DB_USER', 'wp_user');               // The MySQL user you created
   define('DB_PASSWORD', 'xxxxxxxx');          // The password for the MySQL user
   define('DB_HOST', 'localhost');             // Keep this as 'localhost'
   ```

5. **Restart Apache**:

   ```bash
   sudo systemctl restart httpd
   ```

---

## 🛠️ Node.js Application Setup

1. **Extract your Node.js app** into the `/home/ec2-user` directory.

2. **Node.js app (server.js) example**:

   ```javascript
   const express = require('express');
   const app = express();
   const staticFile = express.static('./browser');
   const PORT = 3000;

   app.use(staticFile);

   app.all('*', (req, res, next) => {
       res.redirect('/');
   });

   app.listen(PORT, () => {
       console.log(`Server is Running at ${PORT}`);
   });
   ```

3. **Run Node.js as a Service**:

   ```bash
   sudo npm install pm2 -g
   pm2 start server.js
   pm2 save
   pm2 startup
   ```

---

## 🔄 Services/URLs that Automatically Run After System Restart

### 🌐 Main Service (Apache - WordPress)

- **Homepage**: 👉 [http://<EC2_PUBLIC_IP>/](http://<EC2_PUBLIC_IP>/)
- **WordPress Admin Panel**: 👉 [http://<EC2_PUBLIC_IP>/wordpress/wp-admin](http://<EC2_PUBLIC_IP>/wordpress/wp-admin)

### 🛠️ Node.js Application

- **URL**: 👉 [http://<EC2_PUBLIC_IP>:3000](http://<EC2_PUBLIC_IP>:3000)

---

### ✅ Health Checks

After setup, verify that the following URLs are working:
- **WordPress Homepage**: [http://<EC2_PUBLIC_IP>/](http://<EC2_PUBLIC_IP>/)
- **Node.js App**: [http://<EC2_PUBLIC_IP>:3000](http://<EC2_PUBLIC_IP>:3000)

---
