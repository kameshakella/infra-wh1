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
Here’s a README file with step-by-step instructions on how to back up and restore a WordPress site and MySQL database from one EC2 instance to another, including useful emojis to make it visually engaging:

---

# 📝 Backup and Restore WordPress on EC2

This guide walks you through backing up a WordPress website and its database from one EC2 instance and restoring it on another. 

## 🔧 Prerequisites

- Access to two EC2 instances (existing and new).
- SSH access to both instances.
- MySQL and Apache or Nginx installed on both servers.
- `scp` command to transfer files.

---

## 🛠️ Step 1: Backup Existing Server

### 📦 1.1 Backup the WordPress Database

Run the following command on the existing EC2 instance to dump the WordPress database:

```bash
mysqldump -u wp_user -p wordpress_db > wordpress-db-backup.sql
```

### 🗄 1.2 Backup the WordPress Files

Create a compressed archive of the WordPress directory (usually located in `/var/www/html`):

```bash
cd /var/www/
sudo tar -czf wordpress-backup.tar.gz html/
```

---

## 🚀 Step 2: Set Up New Server

### 🖥️ 2.1 Prepare New EC2 Instance

SSH into the new EC2 instance and update the system:

```bash
sudo yum update -y
```

### 📦 2.2 Install Required Packages

Install Apache, PHP, and MySQL on the new server:

```bash
sudo yum install -y httpd php php-mysqlnd php-fpm php-json php-xml php-gd php-curl
sudo yum install -y https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
sudo yum-config-manager --enable mysql57-community
sudo yum install -y mysql-community-server
```

### 🔑 2.3 Start and Secure MySQL

Start the MySQL service and set up security configurations:

```bash
sudo systemctl start mysqld
sudo systemctl enable mysqld
sudo grep 'temporary password' /var/log/mysqld.log
sudo mysql_secure_installation
```

---

## 📂 Step 3: Transfer Backup Files to New Server

### 🚚 3.1 Copy the WordPress Files

Transfer the `wordpress-backup.tar.gz` file from the existing server to the new server:

```bash
scp -i your-key.pem wordpress-backup.tar.gz ec2-user@NEW_SERVER_IP:/tmp
```

Then move and extract the files on the new server:

```bash
sudo cp /tmp/wordpress-backup.tar.gz /var/www/html/
cd /var/www/html/
sudo tar -xzf wordpress-backup.tar.gz
sudo rm -rf wordpress-backup.tar.gz
```

### 🗄 3.2 Copy the Database Backup

Transfer the database backup file to the new server:

```bash
scp -i your-key.pem wordpress-db-backup.sql ec2-user@NEW_SERVER_IP:/home/ec2-user/
```

---

## 🛠️ Step 4: Restore the Backup on the New Server

### 🗃️ 4.1 Restore the Database

On the new EC2 instance, log into MySQL and create a new database and user:

```bash
mysql -u root -p
```

In the MySQL shell, run:

```sql
CREATE DATABASE wordpress_db;
CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'xxxxxx';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wp_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Import the backup SQL file:

```bash
mysql -u wp_user -p wordpress_db < /home/ec2-user/wordpress-db-backup.sql
```

### ⚙️ 4.2 Update `wp-config.php`

Edit the `wp-config.php` file to match the new database settings:

```bash
sudo vi /var/www/html/wp-config.php
```

Update these lines to reflect the new database information:

```php
define('DB_NAME', 'wordpress_db');
define('DB_USER', 'wp_user');
define('DB_PASSWORD', 'xxxxxx');
define('DB_HOST', 'localhost');
```

---

## 🚦 Step 5: Set Correct File Permissions

Make sure the web server user has ownership of the WordPress files:

```bash
sudo chown -R apache:apache /var/www/html
```

Set correct directory and file permissions:

```bash
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
```

---

## 🔄 Step 6: Restart Services

Restart the Apache (or Nginx) service to apply changes:

```bash
sudo service httpd restart
```

---

## 🎉 You're Done!

Your WordPress site and database have now been successfully backed up from the old EC2 instance and restored on the new instance. Check your website to ensure everything is running smoothly.

