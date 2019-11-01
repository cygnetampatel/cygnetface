FROM php:7.2-apache-stretch

ENV MAGENTO_VERSION 2.3.1
ENV INSTALL_DIR /var/www/html
ENV COMPOSER_HOME /var/www/.composer/

# Configure PHP
# php module build deps
RUN apt-get update && apt-get install -y \
  cron \
  git \
  gzip \
  libfreetype6-dev \
  libicu-dev \
  libjpeg62-turbo-dev \
  libmcrypt-dev \
  libpng-dev \
  libxslt1-dev \
  lsof \
  mysql-client \
  vim \
  zip \
  curl

# http://devdocs.magento.com/guides/v2.0/install-gde/system-requirements.html
RUN docker-php-ext-install bcmath \
  && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
  && docker-php-ext-install gd \
  && docker-php-ext-install intl \
  && docker-php-ext-install mbstring \
  && docker-php-ext-install opcache \
  && docker-php-ext-install pdo_mysql \
  && docker-php-ext-install soap \
  && docker-php-ext-install xsl \
  && docker-php-ext-install zip
  # && docker-php-ext-install sodium	
RUN curl -O https://download.libsodium.org/libsodium/releases/libsodium-1.0.18.tar.gz \
        && tar xfvz libsodium-1.0.18.tar.gz \
        && cd libsodium-1.0.18 \
        && ./configure \
        && make && make install \
        && pecl install -f libsodium
# Install Node, Npm and grunt
RUN apt-get install -y gnupg \
  && curl -sL https://deb.nodesource.com/setup_10.x | bash - \
  && apt-get install -y nodejs \
  && mkdir /var/www/.config /var/www/.npm \
  && npm i -g grunt-cli

# Install Yarn
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
    apt-get update && \
    apt-get install -qq -y build-essential nodejs yarn 

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
COPY auth.json $COMPOSER_HOME

# Add cron job
# ADD crontab /etc/cron.d/magento2-cron
# RUN chmod 0644 /etc/cron.d/magento2-cron \
#     && crontab -u www-data /etc/cron.d/magento2-cron

# Make sure the volume mount point is empty
RUN rm -rf /var/www/html/*

# Set www-data as owner for /var/www
RUN chown -R www-data:www-data /var/www/
RUN chmod -R g+w /var/www/

# # Add SSL certificate
# RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ssl-cert-snakeoil.key -out /etc/ssl/certs/ssl-cert-snakeoil.pem -subj "/C=AT/ST=Vienna/L=Vienna/O=Security/OU=Development/CN=example.com"
RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt -subj "/C=AT/ST=Vienna/L=Vienna/O=Security/OU=Development/CN=example.com"
RUN openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

COPY default-ssl.conf /etc/apache2/sites-available/default-ssl.conf

COPY httpd.conf /etc/apache2/httpd.conf

RUN a2enmod rewrite
RUN a2enmod ssl
RUN a2ensite default-ssl



# copy project files
COPY . /var/www/html
#COPY vhost.conf /etc/apache2/sites-available/000-default.conf
# COPY default-ssl.conf /etc/apache2/sites-available/default-ssl.

WORKDIR $INSTALL_DIR

RUN service apache2 restart


