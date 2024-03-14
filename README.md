# Cacti
My description how to setup Cacti on web server APACHE/NGINX.

NGINX

1. First its install all needed:


#########################################################################################
sudo apt install nginx

sudo apt install -y php-xml php-ldap php-mbstring php-gd php-gmp

sudo apt install php php-fpm php-mysql php-cli php-snmp php-xml php-mbstring  php-intl

sudo apt install -y mariadb-server mariadb-client

#########################################################################################
2. Add settings for conf MariaDB:

#########################################################################################
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
#########################################################################################

Conf settings to add:

#########################################################################################

collation-server = utf8mb4_unicode_ci
max_heap_table_size = 128M
tmp_table_size = 64M
join_buffer_size = 64M
innodb_file_format = Barracuda
innodb_large_prefix = 1
innodb_buffer_pool_size = 1000M
innodb_flush_log_at_timeout = 3
innodb_read_io_threads = 32
innodb_write_io_threads = 16
innodb_io_capacity = 5000
innodb_io_capacity_max = 10000
innodb_doublewrite = OFF
sort_buffer_size = 2M

#########################################################################################

It will be change later when first start webpage.

Restart MariaDB:

#########################################################################################

sudo systemctl restart mariadb

#########################################################################################

3. Find and change settings php.ini:

#########################################################################################

sudo nano /etc/php/8.1/cli/php.ini

  date.timezone = Europe/Kiyv
  memory_limit = 512M
  max_execution_time = 60

#########################################################################################

4. Create base in MariaDB:

#########################################################################################

sudo mysql -u root -p

  create database cacti;
  CREATE USER 'cacti'@'localhost' IDENTIFIED BY 'Cactus1_';
  GRANT ALL PRIVILEGES ON *.* TO 'cacti'@'localhost';
  flush privileges;
  exit

sudo mysql -u root -p mysql < /usr/share/mysql/mysql_test_data_timezone.sql

  sudo mysql -u root -p
  GRANT SELECT ON mysql.time_zone_name TO cacti@localhost;
  flush privileges;
  exit


#########################################################################################

5. Download and install Cacti from cacti.net:

#########################################################################################

wget https://www.cacti.net/downloads/cacti-latest.tar.gz
tar -zxvf cacti-latest.tar.gz
sudo mv cacti-1* /opt/cacti
sudo mysql -u root -p cacti < /opt/cacti/cacti.sql
sudo nano /opt/cacti/include/config.php

#########################################################################################

/* make sure these values reflect your actual database/host/user/password */
$database_type = "mysql";
$database_default = "cacti";
$database_hostname = "localhost";
$database_username = "cacti";
$database_password = "cacti";
$database_port = "3306";
$database_ssl = false;


6. Create or change file:

#########################################################################################

sudo nano /etc/cron.d/cacti
  */5 * * * * www-data php /opt/cacti/poller.php > /dev/null 2>&1

#########################################################################################

7. Nginx conf:

#########################################################################################

sudo nano /etc/nginx/sites-available/cacti
    server {
        listen 80;
        server_name 127.0.0.3;

        root /opt/;
        index index.php index.html index.htm;

         location ~ \.php$ {
                try_files $uri /index.php =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                include fastcgi_params;
        }

        location ~/\.ht {
                deny all;
        }
  }

sudo ln -s /etc/nginx/sites-available/cacti /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
sudo touch /opt/cacti/log/cacti.log
sudo chown -R www-data:www-data /opt/cacti/

#########################################################################################

8. Install snmpd:

#########################################################################################

sudo apt-get install snmpd

nano /etc/default/snmpd

export MIBS=ALL

  
nano /etc/snmp/snmpd.conf
search Access control and copy past this

# Views 
#   arguments viewname included [oid]
rocommunity secret 127.0.0.3(your localhost address)


sudo apt-get install snmp-mibs-downloader

sudo service snmpd restart

#########################################################################################

9. Now try to start webpage on your address(127.0.0.3)
I am connect one mikrotick and one windows server:

![image](https://github.com/dimab7360/Cacti/assets/47431759/e38cdd9f-6e6d-4caa-9bee-4992ccc2ff21)

![image](https://github.com/dimab7360/Cacti/assets/47431759/d0bce1e6-0075-496d-955e-1593f7679f05)



APACHE

Install Cacti on Ubuntu Server
#########################################################################################

1. Install Apache & PHP
	sudo apt install -y apache2 php-mysql libapache2-mod-php
 
2. Install PHP Extensions
	sudo apt install -y php-xml php-ldap php-mbstring php-gd php-gmp

3. Install MariaDB
	sudo apt install -y mariadb-server mariadb-client

4. Install SNMP
	sudo apt install -y snmp php-snmp rrdtool librrds-perl
 
#########################################################################################
 
5. Database Tuning

#########################################################################################
	sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf

Add/Update

collation-server = utf8mb4_unicode_ci
max_heap_table_size = 128M
tmp_table_size = 64M
join_buffer_size = 64M
innodb_file_format = Barracuda
innodb_large_prefix = 1
innodb_buffer_pool_size = 512M
innodb_flush_log_at_timeout = 3
innodb_read_io_threads = 32
innodb_write_io_threads = 16
innodb_io_capacity = 5000
innodb_io_capacity_max = 10000

sudo systemctl restart mariadb

#########################################################################################

6. PHP Configuration

#########################################################################################

 sudo nano /etc/php/7.4/apache2/php.ini

	date.timezone = US/Central
	memory_limit = 512M
	max_execution_time = 60

	sudo nano /etc/php/7.4/cli/php.ini

	date.timezone = US/Central
	memory_limit = 512M
	max_execution_time = 60
 
#########################################################################################

7. Create Database

#########################################################################################
	
 sudo mysql -u root -p

	create database cacti;
	GRANT ALL ON cacti.* TO cacti@localhost IDENTIFIED BY 'cacti';
	flush privileges;
	exit
	
	sudo mysql -u root -p mysql < /usr/share/mysql/mysql_test_data_timezone.sql
	
	sudo mysql -u root -p
	GRANT SELECT ON mysql.time_zone_name TO cacti@localhost;
	flush privileges;
	exit
#########################################################################################

8. Download & Configure Cacti

#########################################################################################

 wget https://www.cacti.net/downloads/cacti-latest.tar.gz
	tar -zxvf cacti-latest.tar.gz
	sudo mv cacti-1* /opt/cacti
	sudo mysql -u root -p cacti < /opt/cacti/cacti.sql
	sudo nano /opt/cacti/include/config.php

/* make sure these values reflect your actual database/host/user/password */
$database_type = "mysql";
$database_default = "cacti";
$database_hostname = "localhost";
$database_username = "cacti";
$database_password = "cacti";
$database_port = "3306";
$database_ssl = false;

#########################################################################################

#########################################################################################

Create a crontab file to schedule the polling job.
	sudo nano /etc/cron.d/cacti
Add the following scheduler entry in the crontab so that Cacti can poll every five minutes
	*/5 * * * * www-data php /opt/cacti/poller.php > /dev/null 2>&1

#########################################################################################

9.Create a new site for the Cacti tool

#########################################################################################

 sudo nano /etc/apache2/sites-available/cacti.conf

Alias /cacti /opt/cacti

  <Directory /opt/cacti>
      Options +FollowSymLinks
      AllowOverride None
      <IfVersion >= 2.3>
      Require all granted
      </IfVersion>
      <IfVersion < 2.3>
      Order Allow,Deny
      Allow from all
      </IfVersion>

   AddType application/x-httpd-php .php

<IfModule mod_php.c>
      php_flag magic_quotes_gpc Off
      php_flag short_open_tag On
      php_flag register_globals Off
      php_flag register_argc_argv On
      php_flag track_vars On
      # this setting is necessary for some locales
      php_value mbstring.func_overload 0
      php_value include_path .
 </IfModule>

  DirectoryIndex index.php
</Directory>

#########################################################################################

Enable the created site.
	sudo a2ensite cacti
Restart Apache services.
	sudo systemctl restart apache2
Create a log file for Cacti and allow the Apache user (www-data) to write a data on to Cacti directory.
	sudo touch /opt/cacti/log/cacti.log
	sudo chown -R www-data:www-data /opt/cacti/

10. Visit the below URL to begin the installation of Cacti

#########################################################################################

 http://your_ip_address/cacti

	Username: admin
	Password: admin

#########################################################################################

I am connect one mikrotick and one windows server:

![image](https://github.com/dimab7360/Cacti/assets/47431759/e38cdd9f-6e6d-4caa-9bee-4992ccc2ff21)

![image](https://github.com/dimab7360/Cacti/assets/47431759/d0bce1e6-0075-496d-955e-1593f7679f05)

