FROM touch4it/php-nginx-symfony:php7.2-fpm-nginx-dev

COPY . /var/www/html

RUN chown -R www-data:www-data /var/www/html
RUN /bin/su -s /bin/bash -c 'composer install -o -a' www-data
RUN /bin/su -s /bin/bash -c 'composer dump-autoload --optimize --no-dev --classmap-authoritative' www-data
