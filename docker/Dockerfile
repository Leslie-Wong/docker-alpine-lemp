FROM php:8.2.15-fpm-alpine3.19

ARG PHP_VERSION=8.2

ENV \
  ADMINER_VERSION=4.8.1

ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/

RUN \
  # prepare
  echo @legacy https://dl-cdn.alpinelinux.org/alpine/v3.19/community >> /etc/apk/repositories \
  # install
  && apk add -U --no-cache \
    openrc \
    composer \
    npm \
    nano \
    nginx \
    redis \
    openssl \
    supervisor

RUN apk add --no-cache --virtual .build-deps $PHPIZE_DEPS \
    && pecl install uploadprogress \
    && docker-php-ext-enable uploadprogress \
    && apk del .build-deps $PHPIZE_DEPS \
    && chmod uga+x /usr/local/bin/install-php-extensions && sync \
    && install-php-extensions bcmath \
                      ctype \
                      curl \
                      dom \
                      fileinfo \
                      filter \
                      gd \
                      hash \
                      iconv \
                      intl \
                      json \
                      libxml \
                      mbstring \
                      openssl \
                      pcre \
                      pdo_mysql \
                      simplexml \
                      soap \
                      sockets \
                      sodium \
                      spl \
                      tokenizer  \
                      xmlwriter \
                      xsl \
                      zip \
                      zlib \
                      xml

RUN \
  # adminer
  mkdir -p /var/www/adminer \
    && curl -sSLo /var/www/adminer/index.php \
      "https://github.com/vrana/adminer/releases/download/v$ADMINER_VERSION/adminer-$ADMINER_VERSION-en.php" \
  # cleanup
  && rm -rf /var/cache/apk/* /tmp/* /var/tmp/* /usr/share/doc/* /usr/share/man/*

RUN cd /var/www \
    && wget -c "https://files.phpmyadmin.net/phpMyAdmin/5.2.0/phpMyAdmin-5.2.0-all-languages.tar.gz" \
    && tar -zxvf phpMyAdmin-5.2.0-all-languages.tar.gz && mv phpMyAdmin-5.2.0-all-languages pma \
    && rm -rf phpMyAdmin-5.2.0-all-languages.tar.gz;

COPY phpmyadmin/config.inc.php /var/www/pma/config.inc.php
COPY php/zz-docker.conf /usr/local/etc/php-fpm.d/zz-docker.conf
# nginx config
# RUN rm -rf /etc/nginx/sites-available/* /etc/nginx/sites-enabled/*

COPY nginx/nginx.conf /etc/nginx/nginx.conf
COPY nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf
# COPY nginx/default.template.conf /etc/nginx/conf.d/01_default.conf
COPY nginx/src/. /var/www/html/
# COPY php/pool.d/ /etc/php//fpm/pool.d/

# RUN ln -s /etc/nginx/sites-available/* /etc/nginx/sites-enabled/

RUN chown -R www-data:www-data /var/www

RUN sed -i "s/Php8.0/php/g" /var/www/html/src/views/index.php
# supervisor config
COPY \
  nginx/nginx.ini \
  # pgsql/pgsql.ini \
  php/php-fpm.ini \
  # rabbitmq/rabbitmq.ini \
  redis/redis.ini \
    /etc/supervisor.d/

# entrypoint
# COPY docker-entrypoint.sh /docker-entrypoint.sh
# RUN chmod +x /docker-entrypoint.sh

# ports
EXPOSE 8000 6001 5173 80

COPY startup.sh .
RUN chmod +x ./startup.sh
# ENTRYPOINT ["./startup.sh"]
# commands
# ENTRYPOINT ["/docker-entrypoint.sh"]
# CMD ["/bin/sh", "-c", "/etc/init.d/php-fpm stop && update-rc.d php-fpm defaults && /etc/init.d/php-fpm start && nginx -g 'daemon off;'"]
CMD ["supervisord", "-n", "-j", "/supervisord.pid"]
# ENTRYPOINT ["./startup.sh"]
# CMD ["/bin/sh", "-c", "supervisord -n -j /supervisord.pid && nginx -g 'daemon off;'"]
# USER www-data
