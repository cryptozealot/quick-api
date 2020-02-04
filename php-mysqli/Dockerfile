# php-mysqli/Dockerfile

FROM php:7.3.14-apache

RUN apt-get update \
      && docker-php-ext-install mysqli php-mysql php-curl \
      && a2enmod rewrite
