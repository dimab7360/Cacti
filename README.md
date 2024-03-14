# Cacti
My description how to setup Cacti on web server APACHE/NGINX.
1. First its install all needed:
#########################################################################################
sudo apt install nginx
sudo apt install -y php-xml php-ldap php-mbstring php-gd php-gmp
sudo apt install php php-fpm php-mysql php-cli php-snmp php-xml php-mbstring  php-intl
sudo apt install -y mariadb-server mariadb-client
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
#########################################################################################
