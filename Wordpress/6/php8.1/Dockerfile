#
# This docker file is heavily based on the official Wordpress docker file.
# https://github.com/docker-library/wordpress/blob/master/latest/php8.1/apache/Dockerfile
# Note that it, however, does not install wordpress.
#
# This Docker file is intended for dev images only. Do not use it in production.
#

FROM php:8.1-apache

# persistent dependencies
RUN set -eux; \
  apt-get update; \
  apt-get install -y --no-install-recommends \
# Ghostscript is required for rendering PDF previews
    ghostscript \
  ; \
  rm -rf /var/lib/apt/lists/*

# install the PHP extensions we need (https://make.wordpress.org/hosting/handbook/handbook/server-environment/#php-extensions)
RUN set -ex; \
  \
  savedAptMark="$(apt-mark showmanual)"; \
  \
  apt-get update; \
  apt-get install -y --no-install-recommends \
    libfreetype6-dev \
    libicu-dev \
    libjpeg-dev \
    libmagickwand-dev \
    libpng-dev \
    libwebp-dev \
    libzip-dev \
  ; \
  \
  docker-php-ext-configure gd \
    --with-freetype \
    --with-jpeg \
    --with-webp \
  ; \
  docker-php-ext-install -j "$(nproc)" \
    bcmath \
    exif \
    gd \
    intl \
    mysqli \
    zip \
  ; \
# https://pecl.php.net/package/imagick
  pecl install imagick-3.6.0; \
  docker-php-ext-enable imagick; \
  rm -r /tmp/pear; \
  \

# some misbehaving extensions end up outputting to stdout 🙈 (https://github.com/docker-library/wordpress/issues/669#issuecomment-993945967)
  out="$(php -r 'exit(0);')"; \
  [ -z "$out" ]; \
  err="$(php -r 'exit(0);' 3>&1 1>&2 2>&3)"; \
  [ -z "$err" ]; \
  \
  extDir="$(php -r 'echo ini_get("extension_dir");')"; \
  [ -d "$extDir" ]; \

# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
  apt-mark auto '.*' > /dev/null; \
  apt-mark manual $savedAptMark; \
  ldd "$extDir"/*.so \
    | awk '/=>/ { so = $(NF-1); if (index(so, "/usr/local/") == 1) { next }; gsub("^/(usr/)?", "", so); print so }' \
    | sort -u \
    | xargs -r dpkg-query -S \
    | cut -d: -f1 \
    | sort -u \
    | xargs -rt apt-mark manual; \
  \
  apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
  rm -rf /var/lib/apt/lists/*; \
  \
  ! { ldd "$extDir"/*.so | grep 'not found'; }; \

# Error logging has been modified to be more fitted for dev envs.
RUN { \
    echo 'error_reporting = E_ALL'; \
    echo 'display_errors = On'; \
    echo 'display_startup_errors = On'; \
    echo 'log_errors = On'; \
    echo 'error_log = /dev/stderr'; \
    echo 'log_errors_max_len = 1024'; \
    echo 'ignore_repeated_errors = On'; \
    echo 'ignore_repeated_source = Off'; \
    echo 'html_errors = On'; \
  } > /usr/local/etc/php/conf.d/error-logging.ini

RUN set -eux; \
  a2enmod rewrite expires; \
  \
# https://httpd.apache.org/docs/2.4/mod/mod_remoteip.html
  a2enmod remoteip; \
  { \
    echo 'RemoteIPHeader X-Forwarded-For'; \
# these IP ranges are reserved for "private" use and should thus *usually* be safe inside Docker
    echo 'RemoteIPTrustedProxy 10.0.0.0/8'; \
    echo 'RemoteIPTrustedProxy 172.16.0.0/12'; \
    echo 'RemoteIPTrustedProxy 192.168.0.0/16'; \
    echo 'RemoteIPTrustedProxy 169.254.0.0/16'; \
    echo 'RemoteIPTrustedProxy 127.0.0.0/8'; \
  } > /etc/apache2/conf-available/remoteip.conf; \
  a2enconf remoteip; \
# https://github.com/docker-library/wordpress/issues/383#issuecomment-507886512
# (replace all instances of "%h" with "%a" in LogFormat)
  find /etc/apache2 -type f -name '*.conf' -exec sed -ri 's/([[:space:]]*LogFormat[[:space:]]+"[^"]*)%h([^"]*")/\1%a\2/g' '{}' +

VOLUME /var/www/html

CMD ["apache2-foreground"]
