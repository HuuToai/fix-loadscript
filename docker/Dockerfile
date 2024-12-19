FROM php:8.3-fpm-alpine

ENV TZ=Asia/Ho_Chi_Minh

# Install system dependencies and build tools
RUN apk add --no-cache \
#    imagemagick-dev \
    libgomp \
    autoconf \
    g++ \
    make \
    bash \
#    build-base \
    libpng-dev \
    libjpeg-turbo-dev \
    libwebp-dev \
    freetype-dev \
    oniguruma-dev \
    icu-dev \
    mysql-client \
    tzdata \
    libxml2-dev \
    libzip-dev \
    zlib-dev \
    curl-dev \
    ffmpeg \
    linux-headers

# Set timezone
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Add system configurations
RUN mkdir -p /etc/security

RUN echo " \
    net.ipv6.conf.all.disable_ipv6 = 1 \n \
    net.ipv6.conf.default.disable_ipv6 = 1 \n \
    net.ipv6.conf.lo.disable_ipv6 = 1 \n \
    kernel.shmall = 4294967296 \n \
    net.ipv4.tcp_window_scaling = 1 \n \
    net.core.rmem_max = 16777216 \n \
    net.core.wmem_max = 16777216 \n \
    net.ipv4.tcp_rmem = 4096 87380 16777216 \n \
    net.ipv4.tcp_wmem = 4096 65536 16777216 \n \
    net.ipv4.tcp_timestamps = 0 \n \
    net.ipv4.tcp_sack = 1 \n \
    net.core.netdev_max_backlog = 10000 \n \
    net.core.somaxconn = 10000 \n \
    net.ipv4.tcp_no_metrics_save = 1 \n \
    net.ipv4.tcp_fin_timeout = 30 \n \
    net.ipv4.tcp_synack_retries = 1 \n \
    vm.swappiness = 1 \n \
" >> /etc/sysctl.conf && \
sysctl -p

# Add file limits configuration
RUN echo " \
    root soft nofile 1000000 \n \
    root hard nofile 1000000 \n \
    www-data soft nofile 1000000 \n \
    www-data hard nofile 1000000 \n \
" >> /etc/security/limits.conf

# Install PHP extensions
RUN docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp \
    && docker-php-ext-install gd \
    pdo \
    pdo_mysql \
    mbstring  \
    exif \
    pcntl  \
    bcmath  \
    opcache intl \
    soap \
    xml \
    zip \
    curl \
    sockets \
    && pecl install mongodb redis \
    && docker-php-ext-enable mongodb \
    && docker-php-ext-enable redis

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /opt/sources/web

# Add the local user and group matching your host's user and group IDs
ARG UID=1000
ARG GID=1000

RUN apk --no-cache add shadow && groupmod -g $GID www-data \
    && usermod -u $UID -g $GID www-data \
    && chown -R www-data:www-data /opt/sources/web


# Copy existing application directory contents
COPY . /opt/sources/web/

COPY ./docker/php/php-conf-88.ini /usr/local/etc/php/conf.d/docker-php-custom.ini

WORKDIR /opt/sources/web

# Install PHP dependencies
RUN composer self-update --stable

# Expose the port for the PHP-FPM service
EXPOSE 9000

# Set the user to www-data for running PHP-FPM
USER www-data

# Start PHP-FPM
CMD ["php-fpm"]
