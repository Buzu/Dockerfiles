#
# Creates an image suitale to run Drupal 9.
#
# Things to consider:
#  It runs on php 7.4
#  It does not install Drupal. It assumes you have cloned your project from somewhere else.
#  It does not do anything about a DB. Use a DB container on the same network, or pod.
#  It does not install composer. Use a composer container for that.
#  It does not install node. Use a node container for that.
#  This is intended for development containers only. Do not use to create prouction containers.
#  This is written and tested for podman, but it should work the same with Docker.
#
# Laregely based on the official Drupal 9.2-apache Dockerfile
# https://github.com/docker-library/drupal/blob/e5eeb6c9a27430505f8a1e213306273104fa79f9/9.2/php8.0/apache-buster/Dockerfile

From php:8.1-apache

# install the PHP extensions we need
RUN set -eux; \
	\
	if command -v a2enmod; then \
		a2enmod rewrite; \
	fi; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	\
	apt-get update; \
	apt-get install -y --no-install-recommends \
		libfreetype6-dev \
		libjpeg-dev \
		libpng-dev \
		libpq-dev \
		libzip-dev \
		imagemagick \
		ghostscript \
		libmagickwand-dev \
	; \
	\

	pecl install imagick \
	; \
	\

	docker-php-ext-configure gd \
		--with-freetype \
		--with-jpeg=/usr \
	; \
	\
	docker-php-ext-install -j "$(nproc)" \
		gd \
		opcache \
		pdo_mysql \
		pdo_pgsql \
		zip \
	; \
	\

	docker-php-ext-enable \
		imagick \
	; \
	\

# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark; \
	ldd "$(php -r 'echo ini_get("extension_dir");')"/*.so \
		| awk '/=>/ { print $3 }' \
		| sort -u \
		| xargs -r dpkg-query -S \
		| cut -d: -f1 \
		| sort -u \
		| xargs -rt apt-mark manual; \
	\
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	rm -rf /var/lib/apt/lists/*

# Change this if your public files live somewhere else that is not {your_project_root}/web
ENV APACHE_DOCUMENT_ROOT /var/www/html/web

RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

ENV PATH=${PATH}:/var/www/html/vendor/bin
