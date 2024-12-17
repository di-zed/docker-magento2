FROM php:8.2-fpm
RUN apt-get update && apt-get install -y zlib1g-dev libpng-dev libfreetype6-dev libjpeg62-turbo-dev zlib1g-dev \
    libicu-dev g++ libxml2 libxml2-dev mcrypt libmcrypt-dev libxslt1-dev git sudo libbz2-dev libzip-dev libonig-dev \
    wget gnupg2
RUN pecl install xdebug-3.2.1 mcrypt
RUN docker-php-ext-configure gd --with-freetype --with-jpeg
RUN docker-php-ext-install pdo_mysql mysqli intl dom mbstring gd soap zip xsl bcmath exif bz2 opcache sockets \
    && docker-php-ext-enable xdebug mcrypt
RUN docker-php-ext-configure pcntl --enable-pcntl \
  && docker-php-ext-install pcntl
# Composer:
ENV COMPOSER_ALLOW_SUPERUSER 1
RUN curl -sS https://getcomposer.org/installer | php -- \
        --version=2.2.21 \
        --filename=composer \
        --install-dir=/usr/local/bin && \
        echo "alias composer='composer'" >> /root/.bashrc
# Install Blackfire CLI tool and PHP Probe:
RUN wget -q -O - https://packages.blackfire.io/gpg.key | apt-key add -
RUN echo "deb http://packages.blackfire.io/debian any main" | tee /etc/apt/sources.list.d/blackfire.list
RUN apt-get update && apt-get install -y blackfire-agent blackfire-php
# Add colors to the terminal:
RUN echo "PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '" >> /root/.bashrc