# ---------------- FRONTEND BUILD ----------------
FROM node:20 AS frontend

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY resources ./resources
COPY vite.config.* ./
COPY tailwind.config.* ./

RUN npm run build


# ---------------- COMPOSER BUILD ----------------
FROM php:8.4-cli AS vendor

WORKDIR /app

RUN apt-get update && apt-get install -y \
    unzip zip git curl libzip-dev libpng-dev libjpeg-dev libfreetype6-dev \
    && docker-php-ext-install zip gd

COPY . .

RUN curl -sS https://getcomposer.org/installer | php -- \
    --install-dir=/usr/local/bin --filename=composer

RUN composer install \
    --no-dev \
    --optimize-autoloader \
    --no-interaction


# ---------------- RUNTIME APP ----------------
FROM php:8.4-fpm AS app

WORKDIR /var/www

RUN apt-get update && apt-get install -y \
    git unzip zip curl \
    libzip-dev libpng-dev libjpeg-dev libfreetype6-dev \
    libonig-dev libxml2-dev \
    && docker-php-ext-install pdo pdo_mysql zip gd

COPY . .

# inject vendor
COPY --from=vendor /app/vendor ./vendor

# inject frontend build
COPY --from=frontend /app/public/build ./public/build

RUN chown -R www-data:www-data storage bootstrap/cache

CMD ["php-fpm"]


# ---------------- NGINX ----------------
FROM nginx:alpine AS nginx

COPY docker/nginx/default.conf /etc/nginx/conf.d/default.conf

COPY public /var/www/public

COPY --from=frontend /app/public/build /var/www/public/build
