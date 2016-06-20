# pedur/taiga-back

[Taiga](https://taiga.io/) is a project management platform for startups and agile developers & designers who want a simple, beautiful tool that makes work truly enjoyable.

This Docker image can be used for running the Taiga backend. It works together with the [pedur/taiga-front](https://hub.docker.com/r/pedur/taiga-docker-front/) image.

This project is forked from [htdvisser/taiga-docker](https://github.com/htdvisser/taiga-docker)

## Running

A [postgres](https://registry.hub.docker.com/_/postgres/) container should be linked to the taiga-back container. The taiga-back container will use the ``POSTGRES_USER`` and ``POSTGRES_PASSWORD`` environment variables that are supplied to the postgres container.

```
docker run --name taiga_back_container_name --link postgres_container_name:postgres pedur/taiga-back
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
```
Make sure to update the credentials and tweak the parameters to your liking.

## Database Initialization

To initialize the database, use ``docker exec -it taiga-back bash`` and execute the following commands:
Or, find the name of your running Docker container with ``docker ps``.

```bash
cd /usr/local/taiga/taiga-back/
python manage.py loaddata initial_user
python manage.py loaddata initial_project_templates
python manage.py loaddata initial_role
```
If you want to do this on Docker Cloud you will need to find the name of your running instance with ``docker-cloud container ps``

And enter your container with ``docker-cloud container exec taigaback-1 bash``

## Environment

* ``SECRET_KEY`` defaults to ``"insecurekey"``, but you might want to change this.
* ``DEBUG`` defaults to ``False``
* ``TEMPLATE_DEBUG`` defaults to ``False``
* ``PUBLIC_REGISTER_ENABLED`` defaults to ``True``

URLs for static files and media files from taiga-back:

* ``MEDIA_URL`` defaults to ``"http://$HOSTNAME/media/"``
* ``STATIC_URL`` defaults to ``"http://$HOSTNAME/static/"``

Domain configuration:

* ``API_SCHEME`` defaults to ``"http"``. Use ``https`` if ``pedur/taiga-front-dist`` is used and SSL enabled.
* ``API_DOMAIN`` defaults to ``"$HOSTNAME"``
* ``API_NAME`` defaults to ``"api"``
* ``FRONT_SCHEME`` defaults to ``"http"``. Use ``https`` if ``pedur/taiga-front-dist`` is used and SSL enabled.
* ``FRONT_DOMAIN`` defaults to ``"$HOSTNAME"``
* ``FRONT_NAME`` defaults to ``"front"``

Email configuration:

* ``EMAIL_USE_TLS`` defaults to ``False``
* ``EMAIL_HOST`` defaults to ``"localhost"``
* ``EMAIL_PORT`` defaults to ``"25"``
* ``EMAIL_HOST_USER`` defaults to ``""``
* ``EMAIL_HOST_PASSWORD`` defaults to ``""``
* ``DEFAULT_FROM_EMAIL`` defaults to ``"no-reply@example.com"``

Database configuration:

* ``POSTGRES_DB_NAME``. Use to override database name.
* ``POSTGRES_USER``. Use to override user specified in linked postgres container.
* ``POSTGRES_PASSWORD``. Use to override password specified in linked postgres container.
