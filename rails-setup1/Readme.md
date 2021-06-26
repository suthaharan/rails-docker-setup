
# Rails 5 with PostgreSQL

* Reference of the build from ...
Link [https://docs.docker.com/samples/rails/]

* Build the project after pulling the given skeleton in the git
```
$ docker-compose run app rails new . --force --database=mysql --skip-bundle
```

* Change the database settings under config/database.yml file
```
default: &default
  adapter: postgresql
  encoding: unicode
  host: db
  username: postgres
  password: password
  pool: 5

development:
  <<: *default
  database: myapp_development

test:
  <<: *default
  database: myapp_test
```


* Build docker
```
$ docker-compose build
```

* Build the database on another terminal window
```
$ docker-compose run web rake db:create
```

* Start docker
```
$ docker-compose up
```

* View the application on the browser URL by visiting web page [http://localhost:3000]


* Stop docker
```
$ docker-compose stop
```

* To rebuild the dockerfile and restart the app
```
$ docker-compose up --build
```

* If gemfile is modified, 
```
$ docker-compose run web bundle install
```
