ARG PHP_VERSION=7.2
ARG TARGET_ENVIRONMENT=production
ARG TARGET_SERVICE=php

FROM t3easy/php:${PHP_VERSION}-development AS builder
COPY . /app
# Remove .docker because it cannot be excluded with .dockerignore
RUN rm -rf /app/.docker \
    && mkdir -p /app/config \
    && mkdir -p /app/var
RUN composer install --no-ansi --no-interaction --no-dev --no-progress --classmap-authoritative

FROM t3easy/php:${PHP_VERSION}-${TARGET_ENVIRONMENT} AS php-base
COPY .docker/cron.sh /usr/local/bin/cron
RUN rm /var/spool/cron/crontabs/root \
 && echo '*/15 * * * * php -f /app/vendor/bin/typo3 scheduler:run' > /var/spool/cron/crontabs/www-data
COPY .docker/ca-certificates /usr/local/share/ca-certificates
RUN update-ca-certificates
ENV PATH "/app/vendor/bin:$PATH"

FROM php-base as php-production
COPY --chown=82 --from=builder /app .
ENV TYPO3_CONTEXT Production

FROM php-base as php-development
ENV TYPO3_CONTEXT Development

FROM nginx:alpine as web-base
COPY .docker/web/typo3.conf /etc/nginx/
COPY .docker/web/default.conf /etc/nginx/conf.d/
WORKDIR /app

FROM web-base as web-production
COPY --chown=101 --from=builder /app /app

FROM web-base as web-development

FROM ${TARGET_SERVICE}-${TARGET_ENVIRONMENT} as runtime