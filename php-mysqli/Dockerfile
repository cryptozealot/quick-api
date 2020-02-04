# php-mysqli/Dockerfile

FROM php:7.3.14-apache

RUN apt-get update \
      && apt-get install -y mysql-client libmysqlclient-dev \
      && docker-php-ext-install mysqli php-mysql php-curl \
      && a2enmod rewrite
