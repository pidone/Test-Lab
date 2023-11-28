FROM php:8.2-apache

RUN pecl install xdebug-3.2.1 \
	&& docker-php-ext-enable xdebug

COPY "php.ini" "$PHP_INI_DIR/php.ini"
COPY www/ /var/www/html/
