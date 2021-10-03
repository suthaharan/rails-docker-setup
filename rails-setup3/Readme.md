# Rails 6 with MySQL, PHPMyAdmin

* Prepare the necessary files

First, create an arbitrary directory and create a file.
```
mkdir rails_app
cd rails_app
```
* Create multiple files at once

touch {Dockerfile,docker-compose.yml,Gemfile,Gemfile.lock,entrypoint.sh,.env}


* Dockerfile
```
FROM ruby:2.7.2

ENV LANG C.UTF-8
ENV APP_ROOT /app

# install required libraries
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
  echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
  apt-get update -qq && \
  apt-get install -y --no-install-recommends \
  build-essential \
  nodejs \
  yarn && \
  apt-get clean && \
  rm --recursive --force /var/lib/apt/lists/*

# create working directory
RUN mkdir $APP_ROOT
WORKDIR $APP_ROOT

# bundle install
COPY Gemfile $APP_ROOT/Gemfile
COPY Gemfile.lock $APP_ROOT/Gemfile.lock
RUN bundle install --jobs 4 --retry 3

# create app in container
COPY . $APP_ROOT

# script to be executed every time the container starts
COPY entrypoint.sh /usr/bin/
RUN chmod +x /usr/bin/entrypoint.sh
ENTRYPOINT ["entrypoint.sh"]
EXPOSE 3000

# Start the main process
CMD ["rails", "server", "-b", "0.0.0.0"]
```
* docker-compose.yml

```
version: '3.7'
services:
    db:
        image: mysql:8.0.20
        container_name: railsdbv2
        restart: always
        volumes:
            - mysql:/var/lib/mysql:delegated
        ports:
            - '3307:3306'
        command: --default-authentication-plugin=mysql_native_password
        env_file: .env
    web:
        build:
            context: .
            dockerfile: Dockerfile
        command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -p 3000 -b '0.0.0.0'"
        tty: true
        stdin_open: true
        env_file: .env
        depends_on:
            - db
        ports:
            - '3000:3000'
        volumes:
            - .:/app:cached
            - bundle:/usr/local/bundle:delegated
            - node_modules:/app/node_modules
            - tmp-data:/app/tmp/sockets
    phpmyadmin:
        depends_on:
            - db
        image: phpmyadmin/phpmyadmin
        container_name: railsphpadmin
        restart: always
        ports:
            - "3333:80"
        environment:
            PMA_HOST: db
            MYSQL_ROOT_PASSWORD: password
            TZ: 'America/Toronto'
volumes:
    mysql:
    bundle:
    node_modules:
    tmp-data:
```

* .env

MYSQL_ROOT_PASSWORD=password
TZ=Toronto


* Gemfile
source 'https://rubygems.org'
gem 'rails', '6.0.3'


* Gemfile.lock
#Leave empty

* entrypoint.sh

#!/bin/bash
set -e

# Remove a potentially pre-existing server.pid for Rails.
rm -f /myapp/tmp/pids/server.pid

# Then exec the container's main process (what's set as CMD in the Dockerfile).
exec "$@"


* Create a Rails project

Once you have created the files you need, run the rails new command.

$ docker-compose run --rm web rails new . --force --no-deps --database=mysql --skip-turbolinks --skip-test

* Edit database.yml

```
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: <%= ENV.fetch('MYSQL_USERNAME') { 'root' } %>
  password: <%= ENV.fetch('MYSQL_PASSWORD') { 'password' } %>
  host: <%= ENV.fetch('MYSQL_HOST') { 'db' } %>

development:
  <<: *default
  database: rails_app_dev

test:
  <<: *default
  database: rails_app_test

production:
  <<: *default
  database: rails_app_prd
  username: myuser
  password: mypass
```

Describe password etc. so that you can access with the information set in docker-compose.yml. host is a db container.

* Build the Docker image and create a DB
docker-compose build
docker-compose run web bin/rails db:create


* Yay! You’re on Rails!
docker-compose up -d

Start the container, and when the initial Rails screen is displayed, you're done.