---
title: How to Wait for Dependencies in Docker Compose
date: 2022-12-18 22:30
categories: [Programming]
tags: [docker, linux]
---

Every once in a while you may find yourself having to deal with services
that have some hard dependencies on each other. Service B (frontend) needs
to have service A (API) running before it starts. In most cases the best
way to deal with this sort of thing is to just restart service B after
a couple of probes for responsiveness. The service's exit code can also
be used for this purpose. Say, these solutions don't work all that well
for you and you really need to have your services wait for each other,
here is something you can do.

Let's work with a theoretical Django application. You need to run
migrations run when the database service is ready. First let's come
up with a Dockerfile for our application:

```Dockerfile
FROM python:3.10

RUN apt-get update -qq
# wget is required for probing HTTP and netcat for raw TCP
RUN apt-get install -y wget netcat

RUN curl -L -o /usr/local/bin/wait-for https://raw.githubusercontent.com/eficode/wait-for/v2.2.3/wait-for
RUN chmod +x /usr/local/bin/wait-for

WORKDIR /opt/django_app/

COPY . /opt/django_app/

RUN pip install poetry
RUN poetry config virtualenvs.create false
RUN poetry install $(test "$ENV" == production && echo "--no-dev") --no-interaction --no-ansi

CMD ["python", "manage"]
```

In the Dockerfile above, we are installing a tool,
[wait-for](https://github.com/eficode/wait-for), that polls an HTTP or
TCP service and runs a command when the service is available (TCP) or
responding with a `200 - OK` (for HTTP). The tool depends on wget
and netcat. If you are only interested in an HTTP service then you only
need to install wget, else you need netcat. Next thing, let's define
our `docker-compose.yml`.

```yaml
version: "3.9"

x-pg-env: &pg-env
  POSTGRES_USER: ${POSTGRES_USER:-iamgroot}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-iamgroot}
  POSTGRES_DB: ${POSTGRES_DB:-groot}

services:
  app:
    build: .
    command: python manage.py runserver
    ports:
      - 8000:8000
    environment:
      <<: *pg-env
    depends_on:
      - database
    links:
      - database
    restart: always
  db-migrations:
    build: .
    command: sh -c 'wait-for database:5432 --timeout 10 -- python manage.py migrate'
    environment:
      <<: *pg-env
    depends_on:
      - database
    links:
      - database
    restart: on-failure
  database:
    image: postgres
    environment:
      <<: *pg-env
    ports:
      - 5432:5432
    restart: always
```

From the compose file above, zone in on the `db-migrations` service.
We have defined a command that first of all tries to wait for postgres
to start. It waits for 10 seconds and fails if not successful within
that time. Otherwise, if a connection is made within that 10 seconds,
the command `python manage.py migrate` is executed. The above is not
a very good use case for this, because as you can see, if
`db-migrations` fails then it will get executed again. I have however
run into situations were I had no option but to do something like
this (e.g. start a frontend service only when the backend service
is running - frontend built with an implicit assumption that the backend
will never be unavailable. It happens more often than you would think.
When is the last time your frontend handled a network error not just
a status code?).

Happy holidays!!!
