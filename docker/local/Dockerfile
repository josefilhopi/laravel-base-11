FROM php:8.3-cli-alpine

WORKDIR /var/www/html

RUN apk add --no-cache \
        bash \
        curl \
        git \
        icu-libs \
        icu-dev \
        libzip \
        libzip-dev \
        oniguruma \
        oniguruma-dev \
        sqlite \
        sqlite-libs \
        sqlite-dev \
        unzip \
    && docker-php-ext-install \
        intl \
        mbstring \
        pcntl \
        pdo \
        pdo_mysql \
        pdo_sqlite \
        zip \
    && apk del icu-dev libzip-dev oniguruma-dev sqlite-dev

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

EXPOSE 8000
