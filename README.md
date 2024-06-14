git clone https://github.com/adimov-jb/environment-php8.git

cd environment-php8/

cp .env-example .env

docker-compose up -d

cd www

git clone https://github.com/adimov-jb/Desafio-Tecnico-Backend-PHP-2023.git .

ls

cp .env.example .env

docker exec -it project-app bash

composer install

php artisan key:gen

arterar os dados de banco no .env

DB_CONNECTION=mysql

DB_HOST=project-db

DB_PORT=3306

DB_DATABASE=project

DB_PASSWORD=secret

DB_USERNAME=project

php artisan optimize

php artisan migrate

php artisan db:seed
