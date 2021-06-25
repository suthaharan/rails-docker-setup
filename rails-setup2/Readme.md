* Reference of the build from ...
Link [hhttps://dev.to/itnext/setting-up-ruby-on-rails-with-docker-and-mysql-3mpc]

* Build the project after pulling the given skeleton in the git
```
$ docker-compose run app rails new . --force --database=mysql --skip-bundle
```

* Change the database settings under config/database.yml file
```
default: &default
  adapter: mysql2
  encoding: utf8
  pool: 5
  database: <%= ENV['DB_NAME'] %>
  username: <%= ENV['DB_USER'] %>
  password: <%= ENV['DB_PASSWORD'] %>
  host: <%= ENV['DB_HOST'] %>

development:
  <<: *default

test:
  <<: *default

production:
  <<: *default
```


* Build docker
```
$ docker-compose build
```

* Start docker
```
$ docker-compose up
```

* View the application on the browser URL by visiting web page [http://localhost:3001]


* Stop docker
```
$ docker-compose stop
```

* To rebuild the dockerfile and restart the app
```
$ docker-compose up --build
```
