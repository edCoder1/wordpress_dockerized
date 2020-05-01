Become a member
Sign in
(WordPress) WordPress Development with Docker-Compose and Wordmove
Kenta Kodashima
Kenta Kodashima
Jul 20, 2019 · 7 min read

As I described in this article, syncing your WordPress data between the production server and the local server used to be a painful task. Using docker-compose and Wordmove, however, it would be much easier. In this article, I will summarize how I manage WordPress websites.
My Environment

    OS: macOS Mojave Version 10.14.5
    Docker version 18.09.2
    docker-compose version 1.23.2

Requirements

    Already installed docker, docker-compose and Wordmove.
    Run docker -v && docker-compose -v to check if you already have installed Docker and docker-compose.
    Know at least what Docker is and what it does.
    Know how to configure SSH access. I don’t explain it since it depends on the environments.

1. Create the docker-compose.yml file

First, create your project directory to develop a WordPress website locally, and add the docker-compose.yml file to the root of your project directory. Then, configure the file as below.

version: '3'
services:
  database:
    image: mysql:5.7
    command:
      - "--character-set-server=utf8"
      - "--collation-server=utf8_unicode_ci"
    ports:
      - "${LOCAL_DB_PORT}:3306"
    restart: on-failure:5
    container_name: "${PRODUCTION_NAME}_db"
    volumes:
      - ./database:/var/lib/mysql
    environment:
      MYSQL_USER: "${MYSQL_USER}"
      MYSQL_DATABASE: "${MYSQL_DATABASE}"
      MYSQL_PASSWORD: "${MYSQL_PASSWORD}"
      MYSQL_ROOT_PASSWORD: "${MYSQL_ROOT_PASSWORD}"
    
  wordpress:
    depends_on:
      - database
    image: wordpress:latest
    container_name: "${PRODUCTION_NAME}_wordpress"
    ports:
      - "${LOCAL_SERVER_PORT}:80"
    restart: on-failure:5
    volumes:
      - ./public:/var/www/html
    environment:
      WORDPRESS_DB_HOST: "${WORDPRESS_DB_HOST}:3306"
      WORDPRESS_DB_PASSWORD: "${MYSQL_PASSWORD}"  phpmyadmin:
    depends_on:
      - database
    image: phpmyadmin/phpmyadmin
    ports:
      - "${LOCAL_PORT_DB_GUI}:80"
    restart: on-failure:5
    container_name: "${PRODUCTION_NAME}_phpmyadmin"
    environment:
      PMA_HOST: "${PMA_HOST}"  wordmove:
    tty: true
    depends_on:
      - wordpress
    image: welaika/wordmove
    restart: on-failure:5
    container_name: "${PRODUCTION_NAME}_wordmove"
    volumes:
      - ./config:/home/
      - ./public:/var/www/html
      - ~/.ssh:/home/.ssh
    environment:
      LOCAL_SERVER_PORT: "${LOCAL_SERVER_PORT}"
      PRODUCTION_URL: "${PRODUCTION_URL}"
      PRODUCTION_DIR_PATH: "${PRODUCTION_DIR_PATH}"
      PRODUCTION_DB_NAME: "${PRODUCTION_DB_NAME}"
      PRODUCTION_DB_USER: "${PRODUCTION_DB_USER}"
      PRODUCTION_DB_PASSWORD: "${PRODUCTION_DB_PASSWORD}"
      PRODUCTION_DB_HOST: "${PRODUCTION_DB_HOST}"
      PRODUCTION_DB_PORT: "${PRODUCTION_DB_PORT}"
      PRODUCTION_SSH_HOST: "${PRODUCTION_SSH_HOST}"
      PRODUCTION_SSH_USER: "${PRODUCTION_SSH_USER}"
      PRODUCTION_SSH_PORT: "${PRODUCTION_SSH_PORT}"

${} is the syntax to read variables from the .env file. The .env file stores all necessary values to use for our docker container. I will explain about the .env file in more details later, so let’s focus on the contents of the docker-compose.yml file now.

    depends_on:
    Specifies the relationship with another container. In the example above, the wordpress container depends on the database container to get and store data.
    image:
    Specifies which image from Docker to use. Although image: wordpress:latest will install the latest version, you can also install other versions by specifying which version you want. For example wordpress:5.0 .
    container_name:
    Specifies the container’s name. Although this is an optional line, it might be better to have this line since it becomes more clear which project you are working on by using the project name variable as a prefix for the names.
    restart: on-failure:5
    Tries to restart the service maximum of 5 time when it fails.
    volumes:
    Mirrors the contents of the container. In the example above, the wordpress container’s contents which are installed in var/www/html will be mirrored into the public directory inside your project directory. All changes to the contents in the public directory will also be reflected in the original var/www/html directory inside the container.
    In the example, the database is also mirrored into the database directory thanks to this line: ./database:/var/lib/mysql . You need to have this line if you want to persist data while working on the project. Otherwise, all modifications data would be gone when you type docker-compose down to call it a day.
    ports:
    Specifies which port to use. The syntax is local:container . If you write 8000:80 for the wordpress container configuration, you will be able to access your WordPress website by visiting http://localhost:8000 on your browser.
    environment:
    Sets some environment variables.

2. Create the .env file inside the project directory

Next, we need the .env file to get values from. Since we need to handle sensitive information such as SSH access information, it is better to have this kind of information as environment variables separated from the actual configuration files for docker-compose or Wordmove. In this way, you can protect your privacy by adding .env to .gitignore while making the most of Git.

I prepared the .env file example with the same variable names used in the docker-compose.yml file example from the previous step. You can freely change the values to fit into your project and use this .env file with the docker-compose.yml file.

For the values under the Local MySQL container , Local WordPress container and Local phpmyadmin container sections, you can also hard code the values directly into the docker-compose.yml file since the values are not so sensitive. I just thought it would be handy when we reuse the configurations in other projects.

# -------------------------------------------
# Common
# -------------------------------------------
# Prefix for container names
PRODUCTION_NAME=your_project_name# -------------------------------------------
# Local MySQL container
# -------------------------------------------
# Change the values to whatever you want
# Port for local MySQL container
LOCAL_DB_PORT=3307
MYSQL_USER=wordpress
MYSQL_DATABASE=wordpress
MYSQL_PASSWORD=wordpress
MYSQL_ROOT_PASSWORD=wordpress# -------------------------------------------
# Local WordPress container
# -------------------------------------------
# Change the values to whatever you want
# Local server port for the wordpress container
LOCAL_SERVER_PORT=8000
# The database container would be WordPress DB host
WORDPRESS_DB_HOST=database
# Password for MySQL which is defined in the database container
WORDPRESS_DB_PASSWORD=wordpress# -------------------------------------------
# Local phpmyadmin container
# -------------------------------------------
# Change the values to whatever you want
# Local port for the phpmyadmin container
LOCAL_PORT_DB_GUI=8080
# The database container would be the host
PMA_HOST=database# -------------------------------------------
# WordMove container
# -------------------------------------------
# URL
PRODUCTION_URL=https://yourdomain.com
# Absolute path for production WordPress
PRODUCTION_DIR_PATH=/path/to/your/wp_project/
# Production DB name
PRODUCTION_DB_NAME=your_db_name
# Production DB user name
PRODUCTION_DB_USER=your_db_username
# Production DB password
PRODUCTION_DB_PASSWORD=your_db_password
# Production DB host
# localhost did not work for me and this issue saved me
# https://github.com/welaika/wordmove/issues/193
PRODUCTION_DB_HOST=127.0.0.1
# Production DB port
PRODUCTION_DB_PORT=your_db_port_on_production
# Production SSH host
PRODUCTION_SSH_HOST=your_domain_name
# Production SSH user
PRODUCTION_SSH_USER=your_ssh_user_name
# Production SSH port
PRODUCTION_SSH_PORT=your_ssh_port

3. Create the config directory inside the project and add the movefile.yml file

Now, create the config directory inside the project and also create the movefile.yml file inside the config directory. Then, configure the file as below.

global:
  sql_adapter: default

local:
  vhost: "http://localhost:<%= ENV['LOCAL_SERVER_PORT'] %>"
  wordpress_path: "/var/www/html/"
  database:
    name: "wordpress"
    user: "root"
    password: "wordpress"
    host: "database"
    mysqldump_options: "--hex-blob"

production:
  vhost: "<%= ENV['PRODUCTION_URL'] %>"
  wordpress_path: "<%= ENV['PRODUCTION_DIR_PATH'] %>"

  database:
    name: "<%= ENV['PRODUCTION_DB_NAME'] %>"
    user: "<%= ENV['PRODUCTION_DB_USER'] %>"
    password: "<%= ENV['PRODUCTION_DB_PASSWORD'] %>"
    host: "<%= ENV['PRODUCTION_DB_HOST'] %>"
    port: "<%= ENV['PRODUCTION_DB_PORT'] %>"
    mysqldump_options: "--hex-blob"

  exclude:
    - '.git/'
    - '.gitignore'
    - '.gitmodules'
    - '.env'
    - 'node_modules/'
    - 'bin/'
    - 'tmp/*'
    - 'Gemfile*'
    - 'Movefile'
    - 'movefile'
    - 'movefile.yml'
    - 'movefile.yaml'
    - 'wp-config.php'
    - 'wp-content/*.sql.gz'
    - '*.orig'
    - 'wp-content/uploads/backwpup*/*'
    - '.htaccess'
    - '.idea/'

  ssh:
    host: "<%= ENV['PRODUCTION_SSH_HOST'] %>"
    user: "<%= ENV['PRODUCTION_SSH_USER'] %>"
    port: "<%= ENV['PRODUCTION_SSH_PORT'] %>"
    rsync_options: "--verbose"

If you have any files or directories which you don’t want to include when you push your website or pull data from the server, you can add the file or directory names under the exclude section.

You can check their documentation for more details.

welaika/wordmove — movefile.yml configurations explained:
https://github.com/welaika/wordmove/wiki/movefile.yml-configurations-explained
4. Develop the website

Now, our configurations for both docker-compose and Wordmove are ready. All you need to do now is to develop your website.

Note: In case you are not sure about information for the production such as URL, domain name, etc., you can just comment out the configuration for the wordmove container and environment variables for the wordmove container in the .env file. When you get that information, just uncomment those lines.

The following command creates docker containers which we configured in docker-compose.yml .

docker-compose up -d

Use the -d flag to run the containers in the background. If you want to be able to see the services’ log messages for debugging purposes, just execute the command without the flag.

If you are following the same steps, your project structure should look as below now.

project
 - public # Stores website files
 - config # The configuration for Wordmove is inside
 - database # Persists data
 - docker-compose.yml
 - .env # This is an invisible file

When you want to stop working and call it a day, you can type the following command.

docker-compose down

The down command stops containers and removes containers, networks, volumes, and images created by the upcommand.

Since this does not remove the volumes, your website files still exist inside the public directory and your data lives inside the database directory. Next time you run docker-compose up -d , it will retrieve data from the database directory(volume) and the public directory(volume). Therefore, those directories are very important and you should be careful when you handle them.
5. Deployment

After developing your website, you finally deploy your website to your production server. Make sure your containers are up and running, then enter into the wordmove container with the following command to be able to execute bash commands.

docker exec -it wordmove_container_name /bin/bash

The following command pushes all data to the production including database and all files.

Note: The command will override the files, therefore, make sure you don’t have any important files inside your production directory.

wordmove push -e production --all

When you want to pull data from the production, you can use the following command.

wordmove pull -e production --all

You can even push/pull only the specific part of data using flags.

--all # All files and data
-w    # Only the '/wp-content/' directory
-u    # Only the '/wp-content/upload/' directory
-t    # Only the '/wp-content/theme/ directory
-p    # Only the '/wp-content/plugins/' directory 
-l    # Only the '/wp-content/language/' directory
-d    # Only the databse
-s    # dry run (does not actually transfer the files)
-e    # Specifies the environment to push or pull
-c    # Only the 'wp-config.php' file

Please be aware that there is no way to undo the command. Therefore, you should pay extra attention to which command you are using and make sure you have proper backups.

You can learn more about the options in their documentation.

welaika/wordmove — Usage and flags explained:
https://github.com/welaika/wordmove/wiki/Usage-and-flags-explained

That’s everything for this article! Happy coding!
References:

    Wordpress development made easy using Docker:
    https://medium.com/seriouslydigital/wordpress-development-made-easy-440b564185f2
    Docker docs — Compose file version 3 reference:
    https://docs.docker.com/compose/compose-file/#environment
    welaika/wordmove — movefile.yml configurations explained:
    https://github.com/welaika/wordmove/wiki/movefile.yml-configurations-explained
    welaika/wordmove — Usage and flags explained:
    https://github.com/welaika/wordmove/wiki/Usage-and-flags-explained
    mysqldump: Got error: 1045: Access denied for user ‘root’@’localhost’ (using password: YES) when trying to connect:
    https://github.com/welaika/wordmove/issues/193
    Setting up the best Wordpress development environment (written in Japanese):
    https://qiita.com/ryo2132/items/d75e1846aa181676406e

    Docker
    WordPress
    Docker Compose
    Software Development
    Web Development

Kenta Kodashima

Written by
Kenta Kodashima
I'm a Full-Stack Developer based in Vancouver. Personal Interests: Books, Philosophy, Piano, Movies, Music, Studio Ghibli.
