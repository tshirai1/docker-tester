FROM php:8.3-fpm-bullseye AS base

WORKDIR /workspace

# timezone environment
ENV TZ="Asia/Tokyo" \
  # locale
  LANG=ja_JP.UTF-8 \
  LANGUAGE=ja_JP:ja \
  LC_ALL=ja_JP.UTF-8 \
  # composer environment
  COMPOSER_ALLOW_SUPERUSER=1 \
  COMPOSER_HOME=/composer

COPY --from=composer:2.7 /usr/bin/composer /usr/bin/composer

# hadolint ignore=DL3008
RUN <<EOF
  apt-get update
  apt-get -y install --no-install-recommends \
    locales \
    git \
    unzip \
    libzip-dev \
    libicu-dev \
    libonig-dev \
    libpq-dev \
    libfreetype6-dev libjpeg62-turbo-dev libpng-dev
  : 'Install Node.js'
  curl -sL https://deb.nodesource.com/setup_18.x | bash -
  : 'Install PHP Extensions'
  apt-get install -y nodejs 
  locale-gen en_US.UTF-8
  localedef -f UTF-8 -i en_US en_US.UTF-8
  docker-php-ext-install \
    intl \
    pdo_mysql \
    zip \
    bcmath
  # permission denied bind mount in Linux environment
  apt-get clean
  rm -rf /var/lib/apt/lists/*
  docker-php-ext-configure gd --with-freetype --with-jpeg
  docker-php-ext-install -j$(nproc) gd
  composer config -g process-timeout 3600
  composer config -g repos.packagist composer https://packagist.org
  npm update
EOF

FROM base AS development

COPY ./infra/docker/php/php.development.ini /usr/local/etc/php/php.ini

FROM development AS development-xdebug

RUN <<EOF
  pecl install xdebug
  docker-php-ext-enable xdebug
EOF

COPY ./infra/docker/php/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

FROM base AS deploy

COPY ./infra/docker/php/php.deploy.ini /usr/local/etc/php/php.ini
COPY ./src /workspace

RUN <<EOF
  composer install -q -n --no-ansi --no-dev --no-scripts --no-progress --prefer-dist
  chmod -R 777 storage bootstrap/cache
  php artisan optimize:clear
  php artisan optimize
EOF
