# n4zim/taiga-docker
[Taiga](https://taiga.io/) is a project management platform for startups and agile developers & designers who want a
simple, beautiful tool that makes work truly enjoyable.

Both of the back and front Taiga Docker images can be found on Docker Hub :
https://hub.docker.com/r/n4zim/taiga/

This project is forked from [pedur/taiga-docker](https://github.com/pedur/taiga-docker)

## Running
A [postgres](https://registry.hub.docker.com/_/postgres/) container should be linked to the taiga-back container.
The taiga-back container will use the ``POSTGRES_USER`` and ``POSTGRES_PASSWORD`` environment variables that are
supplied to the postgres container.

For example to run the back :
```
docker run --name taiga-back --link postgres n4zim/taiga:back
```

Then the front :
```
docker run --name taiga-front --link taiga-back n4zim/taiga:front
```

## Docker-compose
For a complete Taiga installation (``n4zim/taiga:back`` and ``n4zim/taiga:front``) you can use the docker-compose.yml
included file configuration.

Make sure to update the credentials and tweak the parameters to your liking.

## Database Initialization
To initialize the database, use ``docker exec -it taiga-back bash`` and execute the following commands :
Or, find the name of your running Docker container with ``docker ps``.

```bash
cd /usr/local/taiga/taiga-back/
python manage.py loaddata initial_user
python manage.py loaddata initial_project_templates
python manage.py loaddata initial_role
```

If you want to do this on Docker Cloud you will need to find the name of your running instance with
``docker-cloud container ps``

And enter your container with ``docker-cloud container exec taigaback-1 bash``

## Environment

### Front
* ``PUBLIC_REGISTER_ENABLED`` defaults to ``true``
* ``API`` defaults to ``"/api/v1"``
* ``DEBUG`` defaults to ``false``
* ``SCHEME`` defaults to ``http``. If ``https`` is used either
* ``SSL_CRT`` and ``SSL_KEY`` needs to be set **or**
* ``/etc/nginx/ssl/`` volume attached with ``ssl.crt`` and ``ssl.key`` files
* ``SSL_CRT`` SSL certificate value. Valid only when ``SCHEME`` set to https.
* ``SSL_KEY`` SSL certificate key. Valid only when ``SCHEME`` set to https.
* ``DEFAULT_LANGUAGE`` defaults to ``"en"``

### Back
* ``SECRET_KEY`` defaults to ``"insecurekey"``, but you might want to change this.
* ``DEBUG`` defaults to ``False``
* ``TEMPLATE_DEBUG`` defaults to ``False``
* ``PUBLIC_REGISTER_ENABLED`` defaults to ``True``
* ``LANG`` defaults to ``"en_US.UTF-8"``

Database configuration :
* ``POSTGRES_HOST``. Use to override database host.
* ``POSTGRES_PORT`` defaults to ``"5432"``
* ``POSTGRES_NAME``. Use to override database name.
* ``POSTGRES_USER``. Use to override user specified in linked postgres container.
* ``POSTGRES_PASSWORD``. Use to override password specified in linked postgres container.

URLs for static files and media files from taiga-back :
* ``MEDIA_URL`` defaults to ``"http://$HOSTNAME/media/"``
* ``STATIC_URL`` defaults to ``"http://$HOSTNAME/static/"``

Domain configuration :
* ``API_SCHEME`` defaults to ``"http"``. Use ``https`` if ``n4zim/taiga:front`` is used and SSL enabled.
* ``API_DOMAIN`` defaults to ``"$HOSTNAME"``
* ``API_NAME`` defaults to ``"api"``
* ``FRONT_SCHEME`` defaults to ``"http"``. Use ``https`` if ``n4zim/taiga:front`` is used and SSL enabled.
* ``FRONT_DOMAIN`` defaults to ``"$HOSTNAME"``
* ``FRONT_NAME`` defaults to ``"front"``

E-mail configuration :
* ``EMAIL_HOST`` defaults to ``"localhost"``
* ``EMAIL_PORT`` defaults to ``"25"``
* ``EMAIL_USE_TLS`` defaults to ``False``
* ``EMAIL_HOST_USER`` defaults to ``""``
* ``EMAIL_HOST_PASSWORD`` defaults to ``""``
* ``DEFAULT_FROM_EMAIL`` defaults to ``"no-reply@example.com"``
