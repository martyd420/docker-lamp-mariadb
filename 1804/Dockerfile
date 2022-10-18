FROM arm64v8/ubuntu:18.04
ENV REFRESHED_AT 2021-07-04

ENV DOCKER_USER_ID 501 
ENV DOCKER_USER_GID 20

ENV BOOT2DOCKER_ID 1000
ENV BOOT2DOCKER_GID 50

ENV PHPMYADMIN_VERSION=5.1.1
ENV SUPERVISOR_VERSION=4.2.2

# Tweaks to give Apache/PHP write permissions to the app
RUN usermod -u ${BOOT2DOCKER_ID} www-data && \
    usermod -G staff www-data && \
    useradd -r mysql && \
    usermod -G staff mysql && \
    groupmod -g $(($BOOT2DOCKER_GID + 10000)) $(getent group $BOOT2DOCKER_GID | cut -d: -f1) && \
    groupmod -g ${BOOT2DOCKER_GID} staff

# Install packages
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get -y update
RUN apt-get -y dist-upgrade
RUN apt-get -y install software-properties-common
RUN add-apt-repository -y ppa:ondrej/php
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4F4EA0AAE5267A6C
RUN apt-get update
RUN apt-get -y upgrade

RUN apt-get -y install postfix
RUN apt-get -y install python3-setuptools
RUN apt-get -y install wget
RUN apt-get -y install git
RUN apt-get -y install apache2
RUN apt-get -y install openssl
RUN apt-get -y install libapache2-mod-fcgid
RUN apt-get -y install mysql-client
RUN apt-get -y install mariadb-server
RUN apt-get -y install pwgen
RUN apt-get -y install zip
RUN apt-get -y install unzip
RUN apt-get -y install curl
RUN apt-get -y install php5.6
RUN apt-get -y install php5.6-fpm
RUN apt-get -y install php5.6-xdebug
RUN apt-get -y install php5.6-mysql
RUN apt-get -y install php5.6-apcu
RUN apt-get -y install php5.6-gd
RUN apt-get -y install php5.6-xml
RUN apt-get -y install php5.6-common
RUN apt-get -y install php5.6-json
RUN apt-get -y install php5.6-mbstring
RUN apt-get -y install php5.6-gettext
RUN apt-get -y install php5.6-zip
RUN apt-get -y install php5.6-curl
RUN apt-get -y install php5.6-cli
RUN apt-get -y install php5.6-bcmath
RUN apt-get -y install php5.6-bz2
RUN apt-get -y install php5.6-fpm
RUN apt-get -y install php5.6-gd
RUN apt-get -y install php5.6-mcrypt
RUN apt-get -y install php7.4
RUN apt-get -y install php7.4-fpm
RUN apt-get -y install php7.4-xdebug
RUN apt-get -y install php7.4-mysql
RUN apt-get -y install php7.4-apcu
RUN apt-get -y install php7.4-gd
RUN apt-get -y install php7.4-xml
RUN apt-get -y install php7.4-common
RUN apt-get -y install php7.4-json
RUN apt-get -y install php7.4-mbstring
RUN apt-get -y install php7.4-gettext
RUN apt-get -y install php7.4-zip
RUN apt-get -y install php7.4-curl
RUN apt-get -y install php7.4-cli
RUN apt-get -y install php7.4-bcmath
RUN apt-get -y install php7.4-bz2
RUN apt-get -y install php7.4-fpm
RUN apt-get -y install php7.4-gd
RUN apt-get -y install php7.4-mcrypt
RUN apt-get -y install php8.0
RUN apt-get -y install php8.0-fpm
RUN apt-get -y install php8.0-xdebug
RUN apt-get -y install php8.0-mysql
RUN apt-get -y install php8.0-apcu
RUN apt-get -y install php8.0-gd
RUN apt-get -y install php8.0-xml
RUN apt-get -y install php8.0-common
RUN apt-get -y install php8.0-mbstring
RUN apt-get -y install php8.0-gettext
RUN apt-get -y install php8.0-zip
RUN apt-get -y install php8.0-curl
RUN apt-get -y install php8.0-cli
RUN apt-get -y install php8.0-bcmath
RUN apt-get -y install php8.0-bz2
RUN apt-get -y install php8.0-fpm
RUN apt-get -y install php8.0-gd
RUN apt-get -y install php8.0-mcrypt

RUN apt-get -y autoremove 
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

# Install supervisor 4
RUN curl -L https://pypi.io/packages/source/s/supervisor/supervisor-${SUPERVISOR_VERSION}.tar.gz | tar xvz && \
  cd supervisor-${SUPERVISOR_VERSION}/ && \
  python3 setup.py install

# Add image configuration and scripts
ADD supporting_files/start-apache2.sh /start-apache2.sh
ADD supporting_files/start-mysqld.sh /start-mysqld.sh
ADD supporting_files/run.sh /run.sh
RUN chmod 755 /*.sh
ADD supporting_files/supervisord-apache2.conf /etc/supervisor/conf.d/supervisord-apache2.conf
ADD supporting_files/supervisord-mysqld.conf /etc/supervisor/conf.d/supervisord-mysqld.conf
ADD supporting_files/supervisord.conf /etc/supervisor/supervisord.conf
ADD supporting_files/mysqld_innodb.cnf /etc/mysql/conf.d/mysqld.cnf


# Remove pre-installed database
RUN rm -rf /var/lib/mysql

# Add MySQL utils
ADD supporting_files/create_mysql_users.sh /create_mysql_users.sh

# Add phpmyadmin
RUN wget -O /tmp/phpmyadmin.tar.gz https://files.phpmyadmin.net/phpMyAdmin/${PHPMYADMIN_VERSION}/phpMyAdmin-${PHPMYADMIN_VERSION}-all-languages.tar.gz
RUN tar xfvz /tmp/phpmyadmin.tar.gz -C /var/www
RUN ln -s /var/www/phpMyAdmin-${PHPMYADMIN_VERSION}-all-languages /var/www/phpmyadmin
RUN mv /var/www/phpmyadmin/config.sample.inc.php /var/www/phpmyadmin/config.inc.php

ENV MYSQL_PASS:-$(pwgen -s 12 1)

# config to enable .htaccess
ADD supporting_files/apache_default /etc/apache2/sites-available/000-default.conf

RUN a2enmod rewrite

# Configure /app 
RUN mkdir -p /app && rm -fr /var/www/html && ln -s /app /var/www/html
ADD app/ /app

#Environment variables to configure php
ENV PHP_UPLOAD_MAX_FILESIZE 512M
ENV PHP_POST_MAX_SIZE 520M
ENV PHP_VERSION 8.0

VOLUME  ["/var/lib/mysql", "/app" ]

EXPOSE 80 3306
CMD ["/run.sh"]