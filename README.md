![Portada](./.github/assets/Portada111.png)



[![forthebadge made-with-python](http://ForTheBadge.com/images/badges/made-with-python.svg)](https://www.python.org/)

[![made-for-VSCode](https://img.shields.io/badge/Made%20for-VSCode-1f425f.svg)](https://code.visualstudio.com/)
![Python](https://img.shields.io/badge/python-3.7%20%7C%203.8-blue)
[![GitHub release](https://img.shields.io/github/release/jaisenbe58r/iaapi.svg)](https://GitHub.com/jaisenbe58r/iaapi/releases/)
[![GitHub commits](https://img.shields.io/github/commits-since/jaisenbe58r/iaapi/v0.0..svg)](https://GitHub.com/jaisenbe58r/iaapi/commit/)
[![Build status](https://ci.appveyor.com/api/projects/status/7vx20e0h5dxcyla2/branch/master?svg=true)](https://ci.appveyor.com/project/jaisenbe58r/iaapi/branch/master)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/2f5b0302acc04a3dac74d6815fdf66e5)](https://www.codacy.com/manual/jaisenbe58r/iaapi?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=jaisenbe58r/iaapi&amp;utm_campaign=Badge_Grade)
[![codecov](https://codecov.io/gh/jaisenbe58r/iaapi/branch/master/graph/badge.svg)](https://codecov.io/gh/jaisenbe58r/iaapi)
[![License](https://img.shields.io/badge/license-MIT-ORANGE.svg)](https://github.com/jaisenbe58r/iaapi/blob/master/LICENSE)
[![GitHub contributors](https://img.shields.io/github/contributors/jaisenbe58r/iaapi.svg)](https://GitHub.com/jaisenbe58r/iaapi/graphs/contributors/)
[![GitHub issues](https://img.shields.io/github/issues/jaisenbe58r/iaapi.svg)](https://GitHub.com/jaisenbe58r/iaapi/issues/)
[![Discuss](https://img.shields.io/badge/discuss-DISCORD-PURPLE.svg)](https://discord.gg/HUxahg)
[![GitHubIssues](https://img.shields.io/badge/issue_tracking-github-violet.svg)](https://github.com/jaisenbe58r/iaapi/issues)


# iAApi
Proyecto base para la creación de una API basada en FastAPI. El proyecto es una adaptación del template proporcionado por ```nsidnev``` en su repositorio de Github: [GitHub | nsidnev](https://github.com/nsidnev/fastapi-realworld-example-app)

# Quickstart

## Instalación y configuración de Docker

A continuación procedemos a instalar y configurar Docker que nos permitirá desplegar nuestros servicios como microservicios en Docker:

```cmd
#### Docker installation ####
# Reference: https://docs.docker.com/install/linux/docker-ce/centos/

# Install pre-requirements:
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# Add docker repo:
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install docker
sudo yum install -y docker-ce docker-ce-cli containerd.io

# Start Docker service
sudo systemctl start docker

# Validate Docker version
docker --version

# Post installation configuration
# sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker

# Download 'hello-world' docker image
docker pull hello-world

# Create a docker container from 'hello-world' image'
docker run hello-world

# List Docker objects
docker images #Images
docker ps -a  #Containers

#Stop Docker service
sudo systemctl stop docker
```
## Clonar Repositorio del proyecto

En primer lugar, clonamos el repositorio e instalamos todas las dependencias para crear y arrancar el entorno de trabajo basado en ``poetry``:

```
git clone https://github.com/jaisenbe58r/iaapi
cd iaapi/
poetry install
poetry shell
```

## Crear entorno de trabajo

Posteriormente creamos el archivo ``.env`` y añadimos las siguientes variables de entorno:

```
export POSTGRES_DB=rwdb POSTGRES_PORT=5432 POSTGRES_USER=postgres POSTGRES_PASSWORD=postgres

touch .env

echo DB_CONNECTION=postgresql://$POSTGRES_USER:$POSTGRES_PASSWORD@$POSTGRES_HOST:$POSTGRES_PORT/$POSTGRES_DB >> .env
echo SECRET_KEY=$(openssl rand -hex 32) >> .env

echo PROJECT_NAME=<your proyect name> >> .env
```

Para ejecutar la aplicación en modo ``debug`` añade:
```
echo DEBUG=True >> .env
```

## Creción del servicio FastAPI como microservicio

El servicio FastAPI se crea a partir de un contenedor docker y se desplega en el clúster de docker swarm dentro del entorno virtual de Producción. Este servicio es la API que recibe las peticiones ```HTTP``` de los clientes y se encarga de comunicarse directamente con los microservicios servidores del modelo para realizar las predicciones y posteriormente devolver el resultado al cliente.

En primer lugar se crea el archivo ``Dockerfile`` en la raiz del proyecto tal que así:

```Dockerfile
FROM python:3.8.1-slim

ENV PYTHONUNBUFFERED 1

EXPOSE 8000
WORKDIR /app


RUN apt-get update && \
    apt-get install -y --no-install-recommends netcat && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

COPY poetry.lock pyproject.toml ./
RUN pip install poetry==1.0.* && \
    poetry config virtualenvs.create false && \
    poetry install --no-dev

COPY docker/entrypoint.sh docker/entrypoint.sh

COPY . ./

RUN chmod +x ./docker/entrypoint.sh

CMD alembic upgrade head && \
    uvicorn --host=0.0.0.0 app.main:app
```

Para crear este servicio Fast API a partir del ``Dockerfile`` y subirlo a nuestra cuenta particular de ```Docker Hub``` procedemos de la siguiente manera:

```
docker build -t jaisenbe58r/iaapi:latest .
echo $DOCKER_PASSWORD | docker login -u $DOCKER_USER --password-stdin
docker push jaisenbe58r/iaapi:latest
```
A continuación os dejo un [enlace](https://avbravo-2.gitbook.io/docker/chapter1/registrarse-en-docker-hub) de cómo crear vuestra propia cuenta en ```Docker Hub```

## Despliegue de servicios con Docker swarm

Docker Swarm es una herramienta integrada en el ecosistema de Docker que permite la gestión de un cluster de servidores. Pone a nuestra disposición una API con la que podemos administrar las  tareas y asignación de recursos de cada contenedor dentro de cada una de las máquinas. Dicha API nos permite gestionar el cluster como si se tratase de una sola máquina Docker.

Para nuestro proyecto, se genera un clúster con docker swarm con 1 replica del microservicio de FastAPI ```jaisenbe58r/iaapi:latest``` creado en el apartado anterior, 1 base de datos postgress ``postgres:11.5-alpine``, 1 administrador de base de datos pgadmin ``dpage/pgadmin4``, 1 visualizador de contenedores docker en el clúster (```visualizer```), 1 microservicio de monitoreo de servicios (```prometheus```) y 1 microservicio de consulta y visualización (```grafana```):


```yml
version: '3'

services:
  app:
    image: jaisenbe58r/iaapi:latest
    ports:
      - 8000:8000
    env_file:
      - .env
    depends_on:
      - db
    command: ['./docker/entrypoint.sh', "db:5432"]
    deploy:
      replicas: 1

  db:
    image: postgres:11.5-alpine
    ports:
      - 5432:5432
    env_file:
      - .env
    volumes:
      - ./dbdata:/var/lib/postgresql/data:cached

  pgadmin:
    image: dpage/pgadmin4
    links:
      - db
    depends_on:
      - db
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: pwdpwd
    # volumes:
    #   - pgadmin:/root/.pgadmin
    ports:
      - 5050:80

  visualizer:
    image: dockersamples/visualizer
    ports:
      - 9001:8080
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    deploy:
      placement:
        constraints: [node.role == manager]

  prometheus:
    image: prom/prometheus
    ports:
      - 9002:9090

  grafana:
    image: grafana/grafana:latest
    ports:
      - 9003:3000
    links:
      - prometheus:prometheus
    environment:
      - GF_USERS_ALLOW_SIGN_UP=false
```

Para poder desplegar el clúster de docker swarm vamos a ejecutar las siguientes lineas de comandos:

```
#Start docker service
sudo systemctl start docker

#Start Docker Swarm
docker swarm init

docker stack deploy -c compose-config-PROD.yml PROD-STACK

# Check services/containers
docker stack ls
docker service ls
docker container ls

#Visualize servicew on web browser (don't forget open port 9001)
http://<public IP>:8000/docs
```

## Chequear servivios activos

Una vez desplejado el clúster con todos los microservicios vamos a chequear que dichos servicios estén activos. Para ello vamos a ejecutar lo siguiente:

```cmd
docker stack ls
docker service ls
docker container ls
```

Para acceder al visualizador del clúster, basta con introducir en su navegador predeterminado la siguiente ruta:

http://```<public IP>```:9001/


Para eliminar el clúster de docker swarm ejecuta lo siguiente:

```cmd
# Remove stack
docker stack rm PROD-STACK

# Leave docker swarm
docker swarm leave --force

# Stop docker
sudo systemctl stop docker
```


## Monitorización

Como hemos comentado anteriormente, hemos desplegado el microservicio de grafana y prometheus que nos permiten almacenar y visualizar las metricas del cluster en funcionamiento que previamente configuremos.

Para acceder 

```
#### Monitoring ####

# Prometheus: IP:9002

# Grafana: IP:9003
# >> admin/admin

# Grafana Datasource:
# >> public-ip:9002
# >> server

# Grafana dashboards to import
https://grafana.com/grafana/dashboards?dataSource=prometheus

```

# Documentación de la API

La rutas de acceso a la documentación de la API son ``/docs`` or ``/redoc``, generadas con Swagger o ReDoc.


# Estructura del Proyecto


Los archivos relacionados con la aplicación estan en los directorios ``app`` o ``tests``:

```
app
├── api              - web related stuff.
│   ├── dependencies - dependencies for routes definition.
│   ├── errors       - definition of error handlers.
│   └── routes       - web routes.
├── core             - application configuration, startup events, logging.
├── db               - db related stuff.
│   ├── migrations   - manually written alembic migrations.
│   └── repositories - all crud stuff.
├── models           - pydantic models for this application.
│   ├── domain       - main models that are used almost everywhere.
│   └── schemas      - schemas for using in web routes.
├── resources        - strings that are used in web responses.
├── services         - logic that is not just crud related.
└── main.py          - FastAPI application creation and configuration.
```


# Tests

Los Test para este proyecto estan definidos en la carpeta ``test/``.

Este proyecto usa [pytest](https://docs.pytest.org/) para definir las pruebas puesto que nos permite usar el ``asset`` para las excepciones.

Para ejecutar todos los tests del proyecto, se ejecuta el comando ``pytest``:

```
$ pytest
================================================= test session starts ==================================================
platform linux -- Python 3.8.3, pytest-5.4.2, py-1.8.1, pluggy-0.13.1
rootdir: /home/some-user/user-projects/fastapi-realworld-example-app, inifile: setup.cfg, testpaths: tests
plugins: env-0.6.2, cov-2.9.0, asyncio-0.12.0
collected 90 items

tests/test_api/test_errors/test_422_error.py .                                                                   [  1%]
tests/test_api/test_errors/test_error.py .                                                                       [  2%]
tests/test_api/test_routes/test_articles.py .................................                                    [ 38%]
tests/test_api/test_routes/test_authentication.py ..                                                             [ 41%]
tests/test_api/test_routes/test_comments.py ....                                                                 [ 45%]
tests/test_api/test_routes/test_login.py ...                                                                     [ 48%]
tests/test_api/test_routes/test_profiles.py ............                                                         [ 62%]
tests/test_api/test_routes/test_registration.py ...                                                              [ 65%]
tests/test_api/test_routes/test_tags.py ..                                                                       [ 67%]
tests/test_api/test_routes/test_users.py ....................                                                    [ 90%]
tests/test_db/test_queries/test_tables.py ...                                                                    [ 93%]
tests/test_schemas/test_rw_model.py .                                                                            [ 94%]
tests/test_services/test_jwt.py .....                                                                            [100%]

============================================ 90 passed in 70.50s (0:01:10) =============================================
$
```

This project does not use your local ``PostgreSQL`` by default, but creates it in ``docker`` as a container (you can see it if you type ``docker ps`` when the tests are executed, the docker container for ``PostgreSQL`` should be launched with with a name like ``test-postgres-725b4bd4-04f5-4c59-9870-af747d3b182f``). But there are cases when you don't want to use ``docker`` for tests as a database provider (which takes an additional +- 5-10 seconds for its bootstrap before executing the tests), for example, in CI, or if you have problems with the ``docker`` driver or for any other reason. In this case, you can run the tests using your already running database with the following command:

Este proyecto no usa su `` PostgreSQL`` local por defecto, sino que lo crea en ``docker`` como un contenedor (puede verlo si escribe ``docker ps`` cuando se ejecutan las pruebas). Pero hay casos en los que no desea utilizar ``docker`` para las pruebas como proveedor de base de datos (lo que tarda +/- 5-10 segundos adicionales para su arranque antes de ejecutar las pruebas), puede ejecutar las pruebas utilizando su base de datos en local con el siguiente comando:
```
$USE_LOCAL_DB_FOR_TEST=True pytest
```

# CI/CD

En nuestro caso, hemos generado 
