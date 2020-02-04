# php-mysqli/Dockerfile

FROM php:7.3.14-apache

RUN apt-get update \
      && apt-get install -y libmariadb-dev-compat libmariadb-dev \
      && docker-php-ext-install mysqli pdo pdo_mysql \
      && a2enmod rewrite
