ARG PHP_VERSION=7.4
ARG APP_ENV=dev

FROM php:${PHP_VERSION}-fpm-alpine

ENV APP_ENV ${APP_ENV}

RUN set -eux; \
	apk add --no-cache --virtual .build-deps \
		$PHPIZE_DEPS \
		icu-dev \
		libzip-dev \
		postgresql-dev \
		zlib-dev \
	; \
	\
	docker-php-ext-configure zip; \
	docker-php-ext-install -j$(nproc) \
		intl \
		pdo_pgsql \
		zip \
	; \
	pecl install \
		apcu\
	; \
	pecl clear-cache; \
	docker-php-ext-enable \
		apcu \
		opcache \
	; \
    if [ $APP_ENV = "dev" ]; then \
            pecl install \
                xdebug \
            && docker-php-ext-enable \
                xdebug \
            && pecl clear-cache; \
    fi; \
	runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --no-cache --virtual .api-phpexts-rundeps $runDeps; \
	\
	apk del .build-deps

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

RUN if [ $APP_ENV = "prod" ]; then \
        mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"; \
    fi
COPY docker/php/conf.d/php.ini $PHP_INI_DIR/conf.d/php.ini

WORKDIR /app
#COPY composer.json composer.lock ./
#RUN set -eux; \
#	composer install --no-scripts --no-progress --no-suggest; \
#	composer clear-cache

EXPOSE 9000