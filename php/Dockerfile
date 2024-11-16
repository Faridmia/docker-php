# Base image: PHP 8.1 FPM
FROM php:8.1-fpm AS app

# Copy the PHP extension installer from mlocati repository
COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

# Install necessary PHP extensions
RUN set -eux; \
    install-php-extensions pdo pdo_mysql xdebug;

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    unzip \
    zip \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Allow superuser access for Composer
ENV COMPOSER_ALLOW_SUPERUSER=1

# Copy Composer from the official Composer image
COPY --from=composer:2.4 /usr/bin/composer /usr/bin/composer

# Set the working directory
WORKDIR /var/www/html

# Copy Composer files (composer.json and composer.lock) to the container
COPY ./app/composer.* ./

# Install PHP dependencies
RUN composer install --prefer-dist --no-dev --no-scripts --no-progress --no-interaction || \
    composer install --ignore-platform-reqs --no-scripts --no-progress --no-interaction

# Copy the entire application code
COPY ./app ./

# Optimize Composer autoload
RUN composer dump-autoload --optimize

# Development stage (for dev-specific dependencies)
FROM app AS app_dev

# Set Xdebug mode to 'off' by default
ENV XDEBUG_MODE=off

# Copy the Xdebug configuration file
COPY ./php/conf.d/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

# Expose PHP-FPM port
EXPOSE 9000

# Start PHP-FPM
CMD ["php-fpm"]
