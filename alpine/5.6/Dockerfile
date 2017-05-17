FROM php:5.6-fpm-alpine

ENV DRUSH_VERSION 8.1.2
ENV APP_ROOT /app/web

# Recreate user with correct params
RUN deluser www-data && \
	addgroup -g 82 -S www-data && \
	adduser -u 82 -D -S -s /bin/bash -G www-data www-data && \
	sed -i '/^www-data/s/!/*/' /etc/shadow

RUN set -xe && \
    apk add --update \
        bash \
        bzip2 \
        ca-certificates \
        c-client \
        git \
        gzip \
        icu-libs \
        imagemagick \
        imap \
        libbz2 \
        libjpeg-turbo \
        libmcrypt \
        libpng \
        libxslt \
        make \
        mariadb-client \
        libmemcached-libs \
        openssh \
        openssh-client \
        openssl \
        patch \
        postgresql-client \
        rsync \
        su-exec \
        tar \
        wget \
        yaml \

        # Temporarily packages to compile extensions
        autoconf \
        cmake \
        build-base \
        bzip2-dev \
        freetype-dev \
        icu-dev \
        imagemagick-dev \
        imap-dev \
        jpeg-dev \
        libjpeg-turbo-dev \
        libmemcached-dev \
        libmcrypt-dev \
        libpng-dev \
        libtool \
        libxslt-dev \
        openldap-dev \
        pcre-dev \
        postgresql-dev \
        yaml-dev && \

    docker-php-source extract && \

    docker-php-ext-install \
        bcmath \
        bz2 \
        calendar \
        exif \
        imap \
        intl \
        ldap \
        mcrypt \
        mysqli \
        opcache \
        pdo_mysql \
        pdo_pgsql \
        pgsql \
        phar \
        soap \
        sockets \
        xmlrpc \
        xsl \
        zip && \

    # GD
    docker-php-ext-configure gd \
        --with-gd \
        --with-freetype-dir=/usr/include/ \
        --with-png-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/ && \
      NPROC=$(getconf _NPROCESSORS_ONLN) && \
      docker-php-ext-install -j${NPROC} gd && \

    # PECL extensions
    pecl config-set php_ini "$PHP_INI_DIR/php.ini" && \

    # install memcached
    RUN pecl install memcached && \

    # pecl install extensions
    pecl install xdebug && \
    docker-php-ext-enable xdebug && \

    # Install composer
    wget -qO- https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \

    # Install drush
    wget -qO- https://github.com/drush-ops/drush/releases/download/$DRUSH_VERSION/drush.phar > /usr/local/bin/drush && \
    chmod +x /usr/local/bin/drush && \

    # Create working dir
    mkdir -p $APP_ROOT && \
    chown -R www-data:www-data /app/web && \

    # Clean up root crontab
    truncate -s 0 /etc/crontabs/root && \

    # Cleanup
    su-exec www-data composer clear-cache && \
    docker-php-source delete && \
    apk del --purge \
        *-dev \
        autoconf \
        build-base \
        cmake \
        libtool && \

    rm -rf \
        /usr/src/php.tar.xz \
        /usr/src/php/ext/memcached \
        /usr/src/php/ext/uploadprogress \
        /usr/include/php \
        /usr/lib/php/build \
        /var/cache/apk/* \
        /tmp/* \
        /root/.composer

# config composer PATH
ENV PATH "/home/www-data/.composer/vendor/bin:$PATH"

# config china mirror
RUN composer config -g repo.packagist composer https://packagist.phpcomposer.com

# set work dir
WORKDIR $APP_ROOT
EXPOSE 9000

CMD ["php-fpm"]
