FROM php:8.0-fpm-alpine

LABEL org.opencontainers.image.authors="Alexandre Jardin <info@ajardin.fr>"
LABEL org.opencontainers.image.title="Custom PHP 8.0 image for an application supported by Origami."
LABEL org.opencontainers.image.description="https://github.com/ajardin/docker-images/blob/master/php/README.md"
LABEL org.opencontainers.image.source="https://raw.githubusercontent.com/ajardin/docker-images/master/php/8.0/Dockerfile"

# Install application requirements
RUN \
    apk update && \
    apk add --no-cache \
        bash \
        freetype-dev \
        git \
        icu-dev \
        imap-dev \
        jq \
        libjpeg-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
        libssh2-dev \
        libxml2-dev \
        libxml2-utils \
        libxslt-dev \
        libwebp-dev \
        libzip-dev \
        make \
        npm \
        openssh-client \
        patch \
        perl \
        postgresql-libs \
        shadow \
        ssmtp \
        tidyhtml-dev \
        yarn && \
    apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		postgresql-dev && \
    docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp && \
    docker-php-ext-install -j$(nproc) \
        bcmath \
        exif \
        imap \
        intl \
        gd \
        opcache \
        pcntl \
        pdo_mysql \
        pdo_pgsql \
        soap \
        sockets \
        tidy \
        xsl \
        zip && \
    yes "" | pecl install apcu redis ssh2-beta && \
    docker-php-ext-enable apcu redis ssh2 && \
    perl -pi -e "s/mailhub=mail/mailhub=maildev/" /etc/ssmtp/ssmtp.conf && \
    perl -pi -e "s|;pm.status_path = /status|pm.status_path = /php_fpm_status|g" /usr/local/etc/php-fpm.d/www.conf && \
    yarn global add grunt-cli gulp && \
    apk del .build-deps

# Workaround because the iconv extension behavior
RUN apk add gnu-libiconv --update-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing/ --allow-untrusted
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

# Install netz98/n98-magerun2
RUN \
    curl -sS https://files.magerun.net/n98-magerun2.phar --output /usr/local/bin/magerun2 && \
    chmod +x /usr/local/bin/magerun2

# Assign a new UID/GID to avoid using a generated value
RUN \
    usermod -u 1000 www-data && \
    groupmod -g 1000 www-data

# Install Composer globally
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_MEMORY_LIMIT -1

# Install custom entrypoint
COPY --from=ajardin/php-entrypoint:latest /usr/local/bin/docker-custom-entrypoint /usr/local/bin/docker-custom-entrypoint
RUN chmod 777 /usr/local/bin/docker-custom-entrypoint
CMD ["php-fpm"]
ENTRYPOINT ["docker-custom-entrypoint"]
