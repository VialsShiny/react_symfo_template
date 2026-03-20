FROM php:8.3-apache

RUN apt-get update && apt-get install -y \
        libicu-dev libzip-dev zip unzip git \
    && docker-php-ext-install pdo pdo_mysql intl zip opcache

# Opcache + JIT
RUN echo "opcache.enable=1\n\
opcache.memory_consumption=256\n\
opcache.max_accelerated_files=20000\n\
opcache.revalidate_freq=0\n\
opcache.validate_timestamps=0\n\
opcache.jit=tracing\n\
opcache.jit_buffer_size=256M" \
    > /usr/local/etc/php/conf.d/opcache.ini

# APCu
RUN pecl install apcu && docker-php-ext-enable apcu
RUN echo "apc.enable_cli=1" > /usr/local/etc/php/conf.d/apcu.ini

RUN a2enmod rewrite

RUN sed -i 's|/var/www/html|/var/www/html/public|g' \
    /etc/apache2/sites-available/000-default.conf

RUN echo '<Directory /var/www/html/public>\n\
    AllowOverride All\n\
    Require all granted\n\
</Directory>' >> /etc/apache2/sites-available/000-default.conf

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

WORKDIR /var/www/html

COPY composer.json composer.lock ./
RUN composer install --optimize-autoloader --no-scripts

RUN echo '#!/bin/bash\n\
cat > /var/www/html/public/.htaccess << EOF\n\
DirectoryIndex index.php\n\
<IfModule mod_rewrite.c>\n\
    RewriteEngine On\n\
    RewriteCond %{REQUEST_FILENAME} !-f\n\
    RewriteRule ^ index.php [QSA,L]\n\
</IfModule>\n\
EOF\n\
sleep 2\n\
php bin/console cache:clear --env=dev 2>/dev/null || true\n\
php bin/console cache:warmup --env=dev 2>/dev/null || true\n\
apache2-foreground' > /usr/local/bin/docker-entrypoint.sh \
    && chmod +x /usr/local/bin/docker-entrypoint.sh

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]