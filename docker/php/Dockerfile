# Multi-stage builds are a new feature requiring Docker latest  on the daemon and client. 

# stage
FROM composer:latest as composer
LABEL stage=intermediate

COPY ./src /app

RUN composer install
# stage END

# stage
FROM php:8.3-fpm-alpine

WORKDIR /var/www/html
COPY --from=composer /app /var/www/html

RUN chown -R www-data:www-data /var/www/html

RUN apk update; \
    apk upgrade;

# RUN docker-php-ext-install mysqli mbstring pdo pdo_mysql ctype tokenizer xml openssl json
# stage END