#!/bin/bash

exec > /var/log/wordpress.log 2>&1

mkdir -p /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-01dc000d3b226bc66 fs-00391886ce8ea46f4:/ /var/www/

# Install and start Apache
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd

# Install PHP and necessary extensions
sudo yum module reset php -y
sudo yum module enable php:remi-7.4 -y
sudo yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
sudo systemctl start php-fpm
sudo systemctl enable php-fpm

wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php

mkdir -p /var/www/html/
cp -R /wordpress/* /var/www/html/

cd /var/www/html/
touch healthstatus

sed -i "s/localhost/database-1.cnquc6icq2na.eu-west-2.rds.amazonaws.com/g" wp-config.php
sed -i "s/username_here/kosenuel/g" wp-config.php
sed -i "s/password_here/devopsacts/g" wp-config.php
sed -i "s/database_name_here/wordpressdb/g" wp-config.php

chcon -t httpd_sys_rw_content_t /var/www/html/ -R

# Restart Apache
systemctl restart httpd


