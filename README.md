## Como configurar Laravel, Nginx, Redis e MySQL com Docker Compose
## Passo 1 — Fazendo download do Projeto e instalando dependências

Primeiramente, verifique se você está no seu diretório home e faça uma cópia da versão mais recente do Projeto para um diretório chamado environment:

    $ cd ~
    $ git clone git@github.com:acarlosos/environment-php8.git environment-php8

Vá até o diretório environment:

    $ cd ~/environment-php8

Vamos fazer uma cópia do arquivo .env-example para .env e inserir as configurações do banco de dados:

    $ cp .env-example .env

Hora de levantar os containers:

    $ docker-compose up -d

Você pode conseferir se o containner subiu corretamento com o commando docker ps:

    $ docker ps

Acesse o container laravel-app:

    $ docker exec -it CONTAINER_ID bash

Em seguida vamos baixar o projeto Laravel

    $ composer create-project laravel/laravel public

Acesse a pasta do projeto public:

    $ cd public

## Passo 2 - Modificando as configurações do ambiente e executando os contêineres

Você pode agora modificar o arquivo .env no contêiner app para incluir detalhes específicos sobre sua configuração.

    $ nano .env

Encontre o bloco que especifica o DB_CONNECTION e atualize-o para refletir as especificidades da sua configuração. Você modificará os seguintes campos:

O DB_HOST será seu contêiner de banco de dados db.
O DB_DATABASE será o banco de dados laravel.
O DB_USERNAME será o nome de usuário que você usará para o seu banco de dados. Neste caso, vamos usar laraveluser.
O DB_PASSWORD será a senha segura que você gostaria de usar para esta conta de usuário, vamos usar secret.

```/var/www/.env```
```
DB_CONNECTION=mysql
DB_HOST=project-db
DB_PORT=3391
DB_DATABASE=project
DB_USERNAME=project
DB_PASSWORD=secret
```

Salve suas alterações e saia do seu editor.

Com todos os seus serviços definidos no seu arquivo docker-compose, você precisa emitir um único comando para iniciar todos os contêineres, criar os volumes e configurar e conectar as redes:

    $ docker-compose up -d

Assim que o processo for concluído, utilize o comando a seguir para listar todos os contêineres em execução:
    $ docker ps

Você verá o seguinte resultado com detalhes sobre seus contêineres do app, webserver e db:

```Output```

```
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                         NAMES
ac05f1f4a3cf   mysql:8        "docker-entrypoint.s…"   49 seconds ago   Up 47 seconds   0.0.0.0:3306->3306/tcp, 33060/tcp             project-db
bfd339517826   project-app    "docker-php-entrypoi…"   54 seconds ago   Up 51 seconds   9000/tcp                                      app
8124d0d26a48   nginx:alpine   "/docker-entrypoint.…"   52 minutes ago   Up 52 minutes   0.0.0.0:8081->80/tcp, 0.0.0.0:4431->443/tcp   project-webserver
```

Usaremos agora o docker-compose exec para definir a chave do aplicativo para o aplicativo Laravel.
Este comando gerará uma chave e a copiará para seu arquivo .env, garantindo que as sessões do seu usuário e os dados criptografados permaneçam seguros:

    $ docker-compose exec app php artisan key:generate

Para colocar essas configurações em um arquivo de cache, que irá aumentar a velocidade de carregamento do seu aplicativo, execute:

    $ docker-compose exec app php artisan config:cache

Suas definições da configuração serão carregadas em /var/www/bootstrap/cache/config.php no contêiner.

Como passo final, visite http://localhost:8081 no navegador.
Você verá a seguinte página inicial para seu aplicativo Laravel:
<img src="https://i.ibb.co/rxPRtp4/Home.png" >

## Passo 3 - Criando um usuário para o MySQL

Para criar um novo usuário, execute uma bash shell interativa no contêiner db com o docker-compose exec:

    $ docker-compose exec db bash

Dentro do contêiner, logue na conta administrativa root do MySQL:

    root@6efb373db53c:/# mysql -u root -p

Você será solicitado a inserir a senha para a conta root do MySQL ( secret ).

    mysql> show databases;

Você verá o banco de dados laravel listado no resultado:

```
Output
+--------------------+
| Database           |
+--------------------+
| information_schema |
| project        |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.08 sec)
```

Em seguida, crie a conta de usuário que terá permissão para acessar esse banco de dados.

    mysql> GRANT ALL ON laravel_app.* TO 'new_user'@'%' IDENTIFIED BY 'secret';

Reinicie os privilégios para notificar o servidor MySQL das alterações:

    mysql> FLUSH PRIVILEGES;

Saia do MySQL:

    mysql>exit;

Por fim, saia do contêiner:

    root@6efb373db53c:/# exit

## Passo 4 - Migrando dados e teste com o console Tinker

Primeiramente, teste a conexão com o MySQL executando o comando Laravel artisan migrate, que cria uma tabela migrations no banco de dados de dentro do contêiner:

    $ docker-compose exec app php artisan migrate

Este comando irá migrar as tabelas padrão do Laravel. O resultado que confirma a migração será como este:

```
Output

  INFO  Preparing database.

  Creating migration table ....................................................................................................... 326ms DONE

  INFO  Running migrations.

  2014_10_12_000000_create_users_table ........................................................................................... 164ms DONE
  2014_10_12_100000_create_password_resets_table ................................................................................. 325ms DONE
  2019_08_19_000000_create_failed_jobs_table ...................................................................................... 64ms DONE
  2019_12_14_000001_create_personal_access_tokens_table .......................................................................... 254ms DONE
```

Assim que a migração for concluída, você pode fazer uma consulta para verificar se está devidamente conectado ao banco de dados usando o comando tinker:

    $ docker-compose exec app php artisan tinker

Teste a conexão do MySQL obtendo os dados que acabou de migrar:

    >>> \DB::table('migrations')->get();

Você verá um resultado que se parece com este:

```
Output
= Illuminate\Support\Collection {#6154
    all: [
      {#6163
        +"id": 1,
        +"migration": "2014_10_12_000000_create_users_table",
        +"batch": 1,
      },
      {#6165
        +"id": 2,
        +"migration": "2014_10_12_100000_create_password_resets_table",
        +"batch": 1,
      },
      {#6166
        +"id": 3,
        +"migration": "2019_08_19_000000_create_failed_jobs_table",
        +"batch": 1,
      },
      {#6167
        +"id": 4,
        +"migration": "2019_12_14_000001_create_personal_access_tokens_table",
        +"batch": 1,
      },
    ],
  }
>>>
```
Para sair digite o comando abaixo:
    $ exit
## O arquivo do Docker Compose

No arquivo docker-compose, você tem três serviços: app, webserver e db.  Certifique-se de substituir a senha root para o MYSQL_ROOT_PASSWORD, definida como uma variável de ambiente sob o serviço db, por uma senha forte da sua escolha:

```
~/laravel-app/docker-compose.yml

version: '3'
services:

  #PHP Service
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: digitalocean.com/php
    container_name: app
    restart: unless-stopped
    tty: true
    environment:
      SERVICE_NAME: app
      SERVICE_TAGS: dev
    working_dir: /var/www
    volumes:
      - ./www:/var/www
      - ./php/local.ini:/usr/local/etc/php/conf.d/local.ini
    networks:
      - app-backend

  #Nginx Service
  webserver:
    image: nginx:alpine
    container_name: webserver
    restart: unless-stopped
    tty: true
    ports:
      - "8081:80"
      - "4431:443"
    volumes:
      - ./www:/var/www
      - ./nginx/conf.d/:/etc/nginx/conf.d/
    networks:
      - app-backend

  #MySQL Service
  db:
    image: mysql:8
    container_name: db
    restart: unless-stopped
    tty: true
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: ${DB_DATABASE}
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_PASSWORD: ${DB_PASSWORD}
      MYSQL_USER: ${DB_USERNAME}
      SERVICE_TAGS: dev
      SERVICE_NAME: mysql
    volumes:
      - dbdata:/var/lib/mysql
      - ./mysql/my.cnf:/etc/mysql/my.cnf
    networks:
      - app-backend

#Docker Networks
networks:
  app-backend:
    driver: bridge

#Volumes
volumes:
  dbdata:
    driver: local

```
Os serviços aqui definidos incluem:

- app: Contém o aplicativo Laravel e executa uma imagem personalizada do Docker laravel-app, que você definirá no Passo 4. Ela também define o working_dir no contêiner para /var/www.
- webserver: Serviço extrai a imagem nginx:alpine do Docker e expõe as portas 80 e 443.
- db: Serviço extrai a imagem mysql:8 do Docker e define algumas variáveis de ambiente, incluindo um banco de dados chamado laravel_app para o seu aplicativo e a senha da** root** do banco de dados. Você pode dar o nome que quiser ao banco de dados e deve substituir o your_mysql_root_password pela senha forte escolhida. Esta definição de serviço também mapeia a porta 3306 no host para a porta 3306 no contêiner.

Cada propriedade container_name define um nome para o contêiner, que corresponde ao nome do serviço.

Para facilitar a comunicação entre contêineres, os serviços estão conectados a uma rede bridge chamada app-backend. Uma rede bridge utiliza um software bridge que permite que os contêineres conectados à mesma rede bridge se comuniquem uns com os outros. O driver da bridge instala automaticamente regras na máquina do host para que contêineres em redes bridge diferentes não possam se comunicar diretamente entre eles. Isso cria um nível de segurança mais elevado para os aplicativos, garantindo que apenas serviços relacionados possam se comunicar uns com os outros. Isso também significa que você pode definir várias redes e serviços que se conectam a funções relacionadas: os serviços de aplicativo front-end podem usar uma rede frontend, por exemplo, e os serviços back-end podem usar uma rede backend.

## Persistindo os dados

O Docker tem recursos poderosos e convenientes para persistir os dados. No nosso aplicativo, vamos usar volumes e bind mounts para persistir o banco de dados, o aplicativo e os arquivos de configuração. Os volumes oferecem flexibilidade para backups e persistência além do ciclo de vida de um contêiner, enquanto os bind mounts facilitam alterações no código durante o desenvolvimento, fazendo alterações nos arquivos do host ou diretórios imediatamente disponíveis nos seus contêineres. Nossa configuração usa ambos.

```
~/laravel-app/docker-compose.yml
...
#MySQL Service
db:
  ...
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - app-backend
  ...

```
