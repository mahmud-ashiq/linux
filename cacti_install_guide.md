# Cacti Installation Guide for Ubuntu 22.04

This guide provides step-by-step instructions for installing Cacti, a network monitoring and graphing tool, on Ubuntu 22.04 with a LAMP stack.

## Prerequisites

Before beginning the installation, ensure you have:

- Ubuntu 22.04 or Ubuntu Server
- User account with sudo privileges
- LAMP (Linux, Apache, MySQL, PHP) stack installed

## Step 1: Install Required Packages

Install all necessary packages for Cacti to function properly:

```bash
sudo apt install -y apache2 php mariadb-server libapache2-mod-php  php-cli  snmp snmpd rrdtool  php-{mysql,curl,net-socket,gd,intl,pear,imap,memcache,pspell,tidy,xmlrpc,snmp,mbstring,gmp,json,xml,common,ldap,intl}
```

## Step 2: Configure PHP

### Apache PHP Configuration

Edit the PHP configuration file for Apache:

```bash
sudo nano /etc/php/*/apache2/php.ini
```

Locate and update the following parameters:

```ini
date.timezone = Asia/Dhaka
memory_limit = 512M
max_execution_time = 60
```

### CLI PHP Configuration

Edit the PHP CLI configuration file:

```bash
sudo nano /etc/php/*/cli/php.ini
```

Add the timezone setting:

```ini
date.timezone = Asia/Dhaka
```

## Step 3: Configure MariaDB

### Create Cacti Database and User

Access the MariaDB shell:

```bash
sudo mysql -u root -p
```

Execute the following SQL commands:

```sql
CREATE DATABASE cactidb;
GRANT ALL PRIVILEGES ON cactidb.* TO 'cactiuser'@'localhost' IDENTIFIED BY 'YourPassword';
GRANT SELECT ON mysql.time_zone_name TO cactiuser@localhost;
FLUSH PRIVILEGES;
EXIT;
```

**Note:** Replace `YourPassword` with a strong password of your choice.

### Optimize MariaDB Configuration

Edit the MariaDB server configuration file:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Comment out the following lines:

```ini
#character-set-server  = utf8mb4
#collation-server      = utf8mb4_general_ci
```

Add the following optimization parameters:

```ini
innodb_file_format=Barracuda
innodb_large_prefix=1
collation-server=utf8mb4_unicode_ci
character-set-server=utf8mb4
innodb_doublewrite=OFF
max_heap_table_size=128M
tmp_table_size=128M
join_buffer_size=10M
sort_buffer_size = 10M
innodb_buffer_pool_size=2G
innodb_flush_log_at_timeout=3
innodb_read_io_threads=32
innodb_write_io_threads=16
innodb_io_capacity=5000
innodb_io_capacity_max=10000
innodb_buffer_pool_instances=9
```

### Import Timezone Data

Switch to root and import timezone information:

```bash
sudo su -
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root mysql
exit
```

## Step 4: Install Cacti

### Download and Extract Cacti

Navigate to the web directory and download Cacti:

```bash
cd /var/www/html
sudo wget https://www.cacti.net/downloads/cacti-latest.tar.gz
sudo tar -zxvf cacti-latest.tar.gz
sudo mv cacti-* cacti
```

### Configure Cacti Database Connection

Navigate to the Cacti configuration directory:

```bash
cd /var/www/html/cacti/include
cp config.php.dist config.php
sudo nano config.php
```

Update the database configuration parameters:

```php
$database_default  = 'cactidb';
$database_hostname = 'localhost';
$database_username = 'cactiuser';
$database_password = 'YourPassword';
```

**Note:** Use the same password you set in Step 3.

### Set File Permissions

Assign proper ownership to the Cacti directory:

```bash
sudo chown -R www-data:www-data /var/www/html/cacti
```

### Import Cacti Database Schema

Import the default database structure:

```bash
sudo mysql -u root cacti < /var/www/html/cacti/cacti.sql
```

## Step 5: Configure Cacti Daemon Service

### Create Systemd Service File

Create a new systemd service file for the Cacti daemon:

```bash
sudo nano /etc/systemd/system/cactid.service
```

Add the following configuration:

```ini
[Unit]
Description=Cacti Daemon Main Poller Service
After=network.target

[Service]
Type=forking
User=www-data
Group=www-data
EnvironmentFile=/etc/default/cactid
ExecStart=/var/www/html/cacti/cactid.php
Restart=always
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

### Enable and Start Services

Create the environment file and manage the services:

```bash
sudo touch /etc/default/cactid
sudo systemctl daemon-reload
sudo systemctl enable cactid
sudo systemctl restart cactid
sudo systemctl status cactid
```

Restart Apache and MariaDB:

```bash
sudo systemctl restart apache2 mariadb
```

## Step 6: Complete Web-Based Installation

Open your web browser and navigate to:

```
http://your-server-IP-address/cacti/
```

Follow the on-screen installation wizard to complete the Cacti setup.

## Troubleshooting

If you encounter any issues during installation:

- Verify all services are running: `sudo systemctl status apache2 mariadb cactid`
- Check Apache error logs: `sudo tail -f /var/log/apache2/error.log`
- Ensure file permissions are correct on the `/var/www/html/cacti` directory
- Verify database credentials are correct in the `config.php` file