# DVWA Deployment on Ubuntu Server

This section documents the complete process used to deploy DVWA (Damn Vulnerable Web Application) on an Ubuntu Server within the lab environment. DVWA is used to simulate common web application attacks and generate realistic web server logs for later analysis and detection using Splunk.

---

## Step 1: Create the Ubuntu Server Virtual Machine
I created a new virtual machine in VirtualBox to host DVWA. Ubuntu Server was selected to reflect a realistic production-style web server environment.

The VM was connected to the VirtualBox Internal Network (`LAB-LAN`) so that all DVWA traffic would pass through pfSense and remain isolated within the lab.

![Ubuntu Server Specs](../Images/ubuntu-server-specs.png)

---

## Step 2: Install Ubuntu Server
Using the official Ubuntu Server ISO, I installed the operating system with default options. During installation, **OpenSSH Server** was enabled to allow remote management of the system.

After installation completed, I logged in using the user account created during setup.

---

## Step 3: Verify Network Configuration
After logging in, I verified that the server received an IP address from pfSense via DHCP and could communicate with other lab systems.

```bash
ip a
ping 172.16.50.1

```
## Step 4: Update the System

Before installing any services, I updated the system packages to ensure a clean and stable base.

```bash
sudo apt update && sudo apt -y upgrade
```

## Step 5: Install Apache, PHP, and MariaDB

DVWA requires a web server, a database server, and PHP with specific extensions. I installed all required components using the package manager.

```bash
sudo apt -y install apache2 mariadb-server php libapache2-mod-php \
php-mysqli php-gd php-xml php-mbstring php-curl php-zip unzip git
 ```
After installation, I ensured the services were running and enabled at boot.

```bash
sudo systemctl enable --now apache2
sudo systemctl enable --now mariadb
```

## Step 6: Verify Apache Installation

To confirm Apache was working correctly, I accessed the default Apache page from a browser on another lab system.

```cpp
http://<DVWA-IP>/
```

![Apache Default Page](../Images/apache_default_page.png)

The default Apache page loaded successfully, confirming that the web server was reachable.

## Step 7: Download and Deploy DVWA

I downloaded DVWA from its official GitHub repository and placed it in the Apache web root.

```bash
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git dvwa
sudo chown -R www-data:www-data /var/www/html/dvwa
```
This made the DVWA application accessible through Apache.
![DVWA CLone](../Images/dvwa_on_apache_cloning.png)

## Step 8: Create Database and User for DVWA

Next, I created a dedicated database and database user for DVWA using MariaDB.

```bash
sudo mysql
```

```sql
CREATE DATABASE dvwa;
CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'dvwa_password';
GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
This ensures DVWA uses its own isolated database credentials.

## Step 9: Configure DVWA Application Settings

I copied the default DVWA configuration file and updated it with the database connection details.

```bash
cd /var/www/html/dvwa/config
sudo cp config.inc.php.dist config.inc.php
sudo nano config.inc.php
```
Updated values:
```php
$_DVWA['db_server']   = '127.0.0.1';
$_DVWA['db_database'] = 'dvwa';
$_DVWA['db_user']     = 'dvwa';
$_DVWA['db_password'] = 'dvwa_password';
```
## Step 10: Update PHP Configuration

DVWA requires specific PHP settings to be enabled. I first identified the PHP configuration file used by Apache and then updated the required options.
```bash
php --ini
sudo nano /etc/php/8.1/apache2/php.ini
```

The following settings were enabled or adjusted:
```bash
allow_url_fopen = On
allow_url_include = On
display_errors = On
file_uploads = On
upload_max_filesize = 10M
post_max_size = 10M
```

After making these changes, Apache was restarted to apply them.
```bash
sudo systemctl restart apache2
```

## Complete DVWA Setup via Browser

With all backend components configured, I accessed the DVWA setup page and initialized the database.
```bash
http://<DVWA-IP>/dvwa/setup.php
```

After the database was created successfully, I accessed the main DVWA page and logged in using the default credentials.

- Username: admin
- Password: password

## Step 12: Validate DVWA Functionality

Once logged in, I confirmed that DVWA was fully functional and accessible. This verified that the web server, database, and application configuration were working correctly.
![DVWA Main Page](../Images/dvwa_main_page.png)
At this point, the DVWA environment was ready for controlled attack simulations.