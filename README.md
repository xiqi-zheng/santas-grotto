[![CircleCI](https://circleci.com/gh/dschien/santas-grotto.svg?style=svg&circle-token=7d61325169687c2a3091138240b672604e77a912)](https://circleci.com/gh/dschien/santas-grotto) [![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fdschien%2Fsantas-grotto.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fdschien%2Fsantas-grotto?ref=badge_shield)

Please note that this project is released with a Contributor Code of Conduct. By participating in this project you agree to abide by its terms.

# Santa's Grotto App


# Dependencies
* Java 11 
* Maven (`sudo apt-get install maven`)
* Google Maps Geolocation API (configuration below)
* Postgresql 

## Google Maps API
* Get key: [Google developers API](https://developers.google.com/maps/documentation/javascript/get-api-key)
* Store key in `~/.secret.properties` (referenced in [here](https://github.com/dschien/santas-grotto/blob/8832cdd7fbd1031d0f13cc1f5aac32c5a179f589/src/main/java/ac/uk/bristol/cs/santa/grotto/SantasGrottoApp.java#L8))

# Development
* get the repo  
`git clone git@github.com:dschien/santas-grotto.git && cd santas-grotto`


## Run with 'dev' environment
`SPRING_PROFILES_ACTIVE=dev mvn spring-boot:run`

## Configure Database
(installation instructions [here](https://www.postgresql.org/download/))

Or with Docker:
```bash
docker run --rm -p 5432:5432 -it \
-e POSTGRES_PASSWORD=XXX -e POSTGRES_USER=XXX \
--name postgres-db \
-v /tmp/pgdata:/var/lib/postgresql/data
-d postgres:latest
```

for the server and with 
`docker run -it --rm --link postgres postgres /bin/bash` to get a `psql` client.

Create a user 
`createuser santa -P --createdb -h postgres -U root`

and a DB 
`createdb grottodb -U santa -h postgres` 



**with `psql` client**
* create user
`CREATE ROLE santa WITH LOGIN PASSWORD 'hohoho';`
* create db
`CREATE DATABASE grottodb;`
`GRANT ALL PRIVILEGES ON DATABASE grottodb TO santa;`

## Run
`mvn spring-boot:run`

Open in the browser:
[http://localhost:8080/](http://localhost:8080/)


# Deployment

(Create a systemd service for Spring)[https://www.baeldung.com/spring-boot-app-as-a-service]
 
## SSL certs

### With Let's encrypt:
https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx

### SSL Termination with Nginx

server {

        index index.html index.htm index.nginx-debian.html;

        server_name YOURNAME HERE;

        location / {
                proxy_pass http://127.0.0.1:8080;
        }



    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/cms.spe-hub.net/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/cms.spe-hub.net/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot



}


server {
    if ($host = YOUR NAME HERE) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80 default_server;
        listen [::]:80 default_server;

        server_name YOUR NAME HERE;
        return 404; # managed by Certbot


}
