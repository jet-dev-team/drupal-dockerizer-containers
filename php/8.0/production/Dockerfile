FROM php:8.0-apache

# Surpresses debconf complaints of trying to install apt packages interactively
# https://github.com/moby/moby/issues/4032#issuecomment-192327844

# Upload progress not work in php-8.0 for now.

ARG DEBIAN_FRONTEND=noninteractive

# Install useful tools and install important libaries
RUN apt-get -y update && \
  apt-get -y --no-install-recommends --fix-missing install vim nano wget sudo\
  libfreetype6-dev \
  libjpeg62-turbo-dev \
  libpng-dev \
  libmagickwand-dev \
  dialog \
  libsqlite3-dev \
  libsqlite3-0 \
  default-mysql-client \
  zlib1g-dev \
  libzip-dev \
  libicu-dev \
  apt-utils \
  build-essential \
  git \
  curl \
  libmemcached-dev \
  libonig-dev \
  libcurl4 \
  libcurl4-openssl-dev \
  zip \
  openssl \
  libxml2-dev

# Clear Apt
RUN rm -rf /var/lib/apt/lists/*

# Opcache enviroment
ENV PHP_OPCACHE_VALIDATE_TIMESTAMPS="1" \
  PHP_OPCACHE_MAX_ACCELERATED_FILES="10000" \
  PHP_OPCACHE_MEMORY_CONSUMPTION="192" \
  PHP_OPCACHE_MAX_WASTED_PERCENTAGE="10"

# Update pecl
RUN pecl channel-update pecl.php.net

# Install pecl modules
RUN pecl install \
      memcached \
      apcu

RUN git clone https://github.com/Imagick/imagick/ /usr/src/php/ext/imagick/ && \
  docker-php-ext-configure imagick

# Enable php modules
RUN docker-php-ext-enable memcached && \
  docker-php-ext-enable apcu

# Install php modulues
RUN docker-php-ext-install pdo_mysql && \
  docker-php-ext-install opcache && \
  docker-php-ext-install imagick && \
  docker-php-ext-install soap && \
  docker-php-ext-install mysqli && \
  docker-php-ext-install gettext && \
  docker-php-ext-install exif

# Install Freetype
RUN docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg && \
  docker-php-ext-install gd

# Enable apache modules
RUN a2enmod rewrite headers
RUN a2enmod ssl

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Set memory limit
RUN echo "memory_limit = 512M" >> /usr/local/etc/php/conf.d/memory_limit.ini

# Enable apcu in cofig
RUN echo "apc.enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini

# Cleanup
RUN rm -rf /usr/src/*

# Create dockerizer user
RUN useradd -ms /bin/bash dockerizer
RUN groupmod --gid 1000 dockerizer && usermod --uid 1000 --gid 1000 dockerizer

RUN chmod +x /usr/local/bin/docker-php-entrypoint

RUN chown -R dockerizer:dockerizer /var/www

RUN echo dockerizer ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/dockerizer
RUN chmod 0440 /etc/sudoers.d/dockerizer

ENV APACHE_RUN_USER dockerizer
ENV APACHE_RUN_GROUP dockerizer
