FROM composer:2.4.4 as composer
FROM php:8.1.12-fpm

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    locales \
    libzip-dev \
    git

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install extensions
RUN docker-php-ext-install pdo_mysql && \
    docker-php-ext-install zip && \
    docker-php-ext-install exif

# Install GD
RUN apt-get -y update && \
    apt-get --no-install-recommends install -y libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libwebp-dev && \
    rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-configure gd --enable-gd --with-freetype --with-jpeg --with-webp && \
    docker-php-ext-install gd

# Add user for laravel application
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www

# php-fpm configuration
COPY docker/configs/local.ini /usr/local/etc/php/php.ini

# Install dependencies
WORKDIR /var/www
COPY --from=composer /usr/bin/composer /usr/bin/composer
COPY composer.json composer.lock* /var/www/
RUN composer install --no-scripts --no-autoloader --ansi --no-interaction

# Copy app code to /var/www
COPY --chown=www:www-data . /var/www
COPY .env.example /var/www/.env
RUN mkdir -p /var/www/storage/framework/sessions ; mkdir -p /var/www/storage/framework/cache; mkdir -p /var/www/storage/framework/views; chown -R www:www-data /var/www/storage/framework
RUN chmod -R 0775 /var/www/storage

RUN composer dump-autoload -o

# Expose port 9000 and start php-fpm server
EXPOSE 9000
CMD ["php-fpm"]
