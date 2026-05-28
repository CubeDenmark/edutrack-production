# ---------- FRONTEND BUILD ----------
FROM node:20 AS frontend

WORKDIR /app

COPY package*.json ./

# RUN npm ci
RUN --mount=type=cache,target=/root/.npm npm ci

# Copy only frontend-related files (NOT full repo first)
COPY resources ./resources
COPY vite.config.* ./
COPY tailwind.config.* ./
COPY tsconfig.json ./

RUN rm -rf public/build && npm run build


# ---------- PHP APP ----------
FROM php:8.4-fpm AS app

RUN apt-get update && apt-get install -y \
    git unzip zip curl \
    libzip-dev libpng-dev libjpeg-dev \
    libfreetype6-dev libonig-dev libxml2-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install pdo pdo_mysql zip gd

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www

COPY . .

COPY --from=frontend /app/public/build /var/www/public/build

# RUN composer install --no-dev --optimize-autoloader
RUN --mount=type=cache,target=/tmp/composer-cache \
    composer install --no-dev --optimize-autoloader

RUN chown -R www-data:www-data storage bootstrap/cache

EXPOSE 9000

CMD ["php-fpm"]


# ---------- NGINX ----------
FROM nginx:alpine AS nginx

WORKDIR /var/www

# copy Laravel public folder
COPY public ./public

# copy vite build assets
COPY --from=frontend /app/public/build /var/www/public/build

# nginx config
COPY docker/nginx/default.conf /etc/nginx/conf.d/default.conf
