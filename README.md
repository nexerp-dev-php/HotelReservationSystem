How to create a Laravel project?
--------------------------------
1. Download and install Composer
2. Run the following command
    composer create-project laravel/laravel [target folder]

How to deploy Laravel project to Docker?
----------------------------------------
1. Create Dockerfile

FROM php:8.2-fpm

# Install dependencies
RUN apt-get update && apt-get install -y \
    libpng-dev libjpeg-dev libonig-dev libxml2-dev zip unzip curl git \
    && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

WORKDIR /var/www

COPY . .

RUN composer install

CMD ["php-fpm"]


2. Create docker-compose.yml

version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: laravel-app
    volumes:
      - .:/var/www
    depends_on:
      - db

  db:
    image: mysql:8.0
    container_name: laravel-db
    environment:
      MYSQL_DATABASE: laravel
      MYSQL_ROOT_PASSWORD: secret
    ports:
      - "3306:3306"

  nginx:
    image: nginx:alpine
    container_name: laravel-nginx
    ports:
      - "8080:80"
    volumes:
      - .:/var/www
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app


3. Create nginx.conf

server {
    listen 80;
    index index.php index.html;
    root /var/www/public;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}


4. Update .env for MySQL

DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=secret

5. Run docker deployment command

docker-compose up -d

6. Run Laravel setup command

docker exec -it laravel-app php artisan key:generate
docker exec -it laravel-app php artisan migrate

7. Set Correct Permissions

docker exec -it laravel-app bash
chmod -R 775 storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache

8. Stop and restar Laravel-app container

Docker Commands
---------------

1. List containers

docker ps -a

2. Stop container

docker stop [container ID/name]

3. Start container

docker start [container id/name]

4. Restart container

docker restart [container id/name]

5. Accessing container's console

docker exec -it [container name] bash

Redeploy App to Docker
----------------------

1. Shut down all container

docker-compose down

2. Rebuild the project

docker-compose build
docker-compose build --no-cache

3. Start fresh containers

docker-compose up -d

4. Run Laravel setup command

docker exec -it laravel-app php artisan migrate
docker exec -it laravel-app php artisan config:cache
docker exec -it laravel-app php artisan route:cache

5. Clear old data (optional)

docker volume prune

6. Stop and remove all containers

docker -rm -f $(docker ps -aq)

How to Setup Laravel Breeze (Laravel Default Authentication)?
-------------------------------------------------------------

1. sudo composer require laravel/breeze --dev

2. php artisan breeze:install

Note: After successfully setup Breeze, all default settings and files would be populated into the project.

3. For Cloud environment, configure the trusted proxies setting as below:
Laravel 10:
Browse to app > Http > Middleware > TrustProies.php

Change below line of code:
protected $proxies = '*';

Laravel 11 and above:
To configure trusted proxies, use the trustProxies method in your application's bootstrap/app.php file. This allows you to specify the proxies and headers your application should trust. For example:

->withMiddleware(function (Middleware $middleware) {
   $middleware->trustProxies(at: [
       '192.168.1.1',
       '10.0.0.0/8',
   ]);
});

You can also define which proxy headers should be trusted. For instance, if you're using AWS Elastic Load Balancing, you can configure the headers as follows:

->withMiddleware(function (Middleware $middleware) {
   $middleware->trustProxies(headers: Request::HEADER_X_FORWARDED_AWS_ELB);
});

For other environments, you can combine multiple headers:

->withMiddleware(function (Middleware $middleware) {
$middleware->trustProxies(headers:
Request::HEADER_X_FORWARDED_FOR |
Request::HEADER_X_FORWARDED_HOST |
Request::HEADER_X_FORWARDED_PORT |
Request::HEADER_X_FORWARDED_PROTO
);
});

If you're using cloud providers like AWS or other environments where proxy IPs are dynamic, you can trust all proxies by using '*':

->withMiddleware(function (Middleware $middleware) {
   $middleware->trustProxies(at: '*');
});

4. docker-compose build

5. docker-compose up -d

6. docker exec -it laravel-app php artisan migrate
   docker exec -it laravel-app php artisan config:cache
   docker exec -it laravel-app php artisan route:cache

