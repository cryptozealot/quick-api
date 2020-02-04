# php-mysqli/Dockerfile

FROM php:7.1.7-apache

RUN apt-get update \
      && apt-get install -y mysql-client libmysqlclient-dev php-mysql php-curl \
      && docker-php-ext-install mysqli \
      && a2enmod rewrite
