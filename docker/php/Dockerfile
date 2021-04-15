#------------------------------------------------------------------------------
# Build assets
#------------------------------------------------------------------------------
FROM node:14.4-alpine3.12 AS assets

WORKDIR /srv

COPY ./package.json ./package-lock.json ./webpack.config.js ./
RUN set -eux \
    && npm install

COPY assets/ ./assets

RUN set -eux \
    && mkdir -p /srv/public/build \
    && npm run build
 
#------------------------------------------------------------------------------
# Install vendors
#------------------------------------------------------------------------------
FROM composer:2.0.12 AS composer

ENV COMPOSER_ALLOW_SUPERUSER=1

# install Symfony Flex globally to speed up download of Composer packages (parallelized prefetching)
RUN set -eux; \
    composer global require "symfony/flex" --prefer-dist --no-progress --no-suggest --classmap-authoritative; \
    composer clear-cache
ENV PATH="${PATH}:/root/.composer/vendor/bin"

COPY composer.json composer.lock symfony.lock ./
RUN set -eux; \
    composer install --prefer-dist --no-dev --no-scripts --no-progress --no-suggest --ignore-platform-reqs; \
    composer clear-cache
#------------------------------------------------------------------------------
# PHP FPM
#------------------------------------------------------------------------------
FROM php:fpm-alpine3.13 

WORKDIR /srv

COPY . .
COPY --from=composer /usr/bin/composer /usr/local/bin/composer
COPY --from=composer /app/vendor /srv/vendor
COPY --from=assets /srv/public/build /srv/public/build

RUN set -eux; \
    apk add --no-cache git zlib-dev \
                           libxml2-dev \
                           libzip-dev \
                           icu-dev

RUN docker-php-ext-install pdo pdo_mysql \
                           mysqli \
                           intl \
                           zip

RUN set -eux; \
	mkdir -p var/cache var/log; \
	composer dump-autoload --classmap-authoritative --no-dev; \
    chmod +x bin/console; \
    php -d memory_limit=-1 bin/console cache:warmup; \
    bin/console assets:install
    
CMD ["php-fpm"]