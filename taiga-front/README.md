# pedur/taiga-front

[Taiga](https://taiga.io/) is a project management platform for startups and agile developers & designers who want a simple, beautiful tool that makes work truly enjoyable.

This Docker image can be used for running the Taiga frontend. It works together with the [pedur/taiga-back](https://registry.hub.docker.com/u/pedur/taiga-back/) image.

This project is forked from [htdvisser/taiga-docker](https://github.com/htdvisser/taiga-docker)

## Running

A [pedur/taiga-back](https://registry.hub.docker.com/u/pedur/taiga-back/) container should be linked to the taiga-front container. Also connect the volumes of this the taiga-back container if you want to serve the static files for the admin panel.

```
docker run --name taiga_front_container_name --link taiga_back_container_name:taigaback --volumes-from taiga_back_container_name pedur/taiga-front
```

## Docker-compose

For a complete taiga installation (``pedur/taiga-back`` and ``pedur/taiga-front-dist``) you can use this docker-compose configuration:

```
data:
  image: tianon/true
  volumes:
    - /var/lib/postgresql/data
    - /usr/local/taiga/media
    - /usr/local/taiga/static
    - /usr/local/taiga/logs
db:
  image: postgres
  environment:
    POSTGRES_USER: taiga
    POSTGRES_PASSWORD: password
  volumes_from:
    - data
taigaback:
  image: pedur/taiga-docker-back
  hostname: dev.example.com
  environment:
    SECRET_KEY: examplesecretkey
    EMAIL_USE_TLS: 1
    EMAIL_HOST: smtp.gmail.com
    EMAIL_PORT: 587
    EMAIL_HOST_USER: example@gmail.com
    EMAIL_HOST_PASSWORD: example
  links:
    - db:postgres
  volumes_from:
    - data
taigafront:
  image: pedur/taiga-docker-front
  hostname: dev.example.com
  links:
    - taigaback
  volumes_from:
    - data
  ports:
    - 0.0.0.0:80:80
taigalog:
  image: gliderlabs/logspout
  volumes:
    - /var/run/docker.sock:/tmp/docker.sock
  environment:
    - "HOSTNAME=SYSLOGGER"
  command: syslog://<REMOTE-SYSLOG-SERVER>

```
Make sure to update the credentials and tweak the parameters to your liking.

## Docker Cloud

If you want to run this on Docker Cloud, you can use the following Stack file:

```
data:
  image: tianon/true
  volumes:
    - /var/lib/postgresql/data
    - /usr/local/taiga/media
    - /usr/local/taiga/static
    - /usr/local/taiga/logs
db:
  image: postgres
  environment:
    POSTGRES_USER: taiga
    POSTGRES_PASSWORD: password
  target_num_containers: 1
  restart: always
  autoredeploy: true
  volumes_from:
    - data
taigaback:
  image: pedur/taiga-docker-back
  hostname: dev.example.com
  environment:
    SECRET_KEY: examplesecretkey
    EMAIL_USE_TLS: 1
    EMAIL_HOST: smtp.gmail.com
    EMAIL_PORT: 587
    EMAIL_HOST_USER: example@gmail.com
    EMAIL_HOST_PASSWORD: example
  target_num_containers: 1
  restart: always
  autoredeploy: true
  deployment_strategy: high_availability
  links:
    - db:postgres
  volumes_from:
    - data
taigafront:
  image: pedur/taiga-docker-front
  hostname: dev.example.com
  links:
    - taigaback
  volumes_from:
    - data
  ports:
    - "80"
  target_num_containers: 1
  restart: always
  autoredeploy: true
  deployment_strategy: high_availability
taigalog:
  image: gliderlabs/logspout
  volumes:
    - /var/run/docker.sock:/tmp/docker.sock
  environment:
    - "HOSTNAME=SYSLOGGER"
  command: syslog://<REMOTE-SYSLOG-SERVER>
```
Make sure to update the credentials and tweak the parameters to your liking.

## SSL Support

HTTPS can be enabled by setting ``SCHEME`` to ``https`` and filling ``SSL_CRT``
and ``SSL_KEY`` env variables (see Environment section below). *http* (port 80)
requests will be redirected to *https* (port 443).

Example:

```
data:
  ...
db:
  ...
taigaback:
  image: pedur/taiga-back:stable
  hostname: dev.example.com
  environment:
    ...
    API_SCHEME: https
    FRONT_SCHEME: https
  links:
    - db:postgres
  volumes_from:
    - data
taigafront:
  image: pedur/taiga-front-dist:stable
  hostname: dev.example.com
  environment:
    SCHEME: https
    SSL_CRT: |
        -----BEGIN CERTIFICATE-----
        ...
        -----END CERTIFICATE-----
    SSL_KEY: |
        -----BEGIN RSA PRIVATE KEY-----
        ...
        -----END RSA PRIVATE KEY-----
  links:
    - taigaback
  volumes_from:
    - data
  ports:
    - 0.0.0.0:80:80
    - 0.0.0.0:443:443
```

## Environment

* ``PUBLIC_REGISTER_ENABLED`` defaults to ``true``
* ``API`` defaults to ``"/api/v1"``
* ``DEBUG`` defaults to ``false``
* ``SCHEME`` defaults to ``http``. If ``https`` is used either
  * ``SSL_CRT`` and ``SSL_KEY`` needs to be set **or**
  * ``/etc/nginx/ssl/`` volume attached with ``ssl.crt`` and ``ssl.key`` files
* ``SSL_CRT`` SSL certificate value. Valid only when ``SCHEME`` set to https.
* ``SSL_KEY`` SSL certificate key. Valid only when ``SCHEME`` set to https.


## Logging

Note: If you want to setup remote logging, I would recommend using [papertrail](https://papertrailapp.com)
After creating your account, you can find your log endpoint in [your settings](https://papertrailapp.com/account/destinations)
Replace <REMOTE-SYSLOG-SERVER> in the above Compose file or Cloud Stack file with the URL found there.

Of course, this can also be a self hosted syslog server.
