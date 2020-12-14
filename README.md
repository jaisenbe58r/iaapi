![Portada](./.github/assets/Portada111.png)



[![forthebadge made-with-python](http://ForTheBadge.com/images/badges/made-with-python.svg)](https://www.python.org/)

[![made-for-VSCode](https://img.shields.io/badge/Made%20for-VSCode-1f425f.svg)](https://code.visualstudio.com/)
![Python](https://img.shields.io/badge/python-3.6%20%7C%203.7%20%7C%203.8-blue)
[![GitHub release](https://img.shields.io/github/release/jaisenbe58r/iaapi.svg)](https://GitHub.com/jaisenbe58r/iaapi/releases/)
[![GitHub commits](https://img.shields.io/github/commits-since/jaisenbe58r/iaapi/v0.0..svg)](https://GitHub.com/jaisenbe58r/iaapi/commit/)
[![PyPI Latest Release](https://badge.fury.io/py/iaapi.svg)](https://pypi.org/project/iaapi/)
[![Build status](https://ci.appveyor.com/api/projects/status/7vx20e0h5dxcyla2/branch/master?svg=true)](https://ci.appveyor.com/project/jaisenbe58r/iaapi/branch/master)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/2f5b0302acc04a3dac74d6815fdf66e5)](https://www.codacy.com/manual/jaisenbe58r/iaapi?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=jaisenbe58r/iaapi&amp;utm_campaign=Badge_Grade)
[![Build Status](https://travis-ci.org/jaisenbe58r/iaapi.svg?branch=master)](https://travis-ci.org/jaisenbe58r/iaapi)
[![codecov](https://codecov.io/gh/jaisenbe58r/iaapi/branch/master/graph/badge.svg)](https://codecov.io/gh/jaisenbe58r/iaapi)
[![License](https://img.shields.io/badge/license-MIT-ORANGE.svg)](https://github.com/jaisenbe58r/iaapi/blob/master/LICENSE)
[![Documentation Status](https://readthedocs.org/projects/ansicolortags/badge/?version=latest)](http://ansicolortags.readthedocs.io/?badge=latest)
[![DOI](https://zenodo.org/badge/256283484.svg)](https://zenodo.org/badge/latestdoi/256283484)
[![GitHub contributors](https://img.shields.io/github/contributors/jaisenbe58r/iaapi.svg)](https://GitHub.com/jaisenbe58r/iaapi/graphs/contributors/)
[![GitHub issues](https://img.shields.io/github/issues/jaisenbe58r/iaapi.svg)](https://GitHub.com/jaisenbe58r/iaapi/issues/)
[![Discuss](https://img.shields.io/badge/discuss-DISCORD-PURPLE.svg)](https://discord.gg/HUxahg)
[![GitHubIssues](https://img.shields.io/badge/issue_tracking-github-violet.svg)](https://github.com/jaisenbe58r/iaapi/issues)
[![saythanks](https://img.shields.io/badge/say-thanks-ff69b4.svg)](https://saythanks.io/to/kennethreitz)





referencias = https://github.com/nsidnev/fastapi-realworld-example-app

# Quickstart

First, run ``PostgreSQL``, set environment variables and create database. For example using ``docker``:

```
export POSTGRES_DB=rwdb POSTGRES_PORT=5432 POSTGRES_USER=postgres POSTGRES_PASSWORD=postgres
docker run --name pgdb --rm -e POSTGRES_USER="$POSTGRES_USER" -e POSTGRES_PASSWORD="$POSTGRES_PASSWORD" -e POSTGRES_DB="$POSTGRES_DB" postgres
export POSTGRES_HOST=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' pgdb)
createdb --host=$POSTGRES_HOST --port=$POSTGRES_PORT --username=$POSTGRES_USER $POSTGRES_DB
```

Then run the following commands to bootstrap your environment with ``poetry``:

```
git clone https://github.com/jaisenbe58r/iaapi
cd iaapi/
poetry install
poetry shell
```

Then create ``.env`` file (or rename and modify ``.env.example``) in project root and set environment variables for application:

```
touch .env
echo DB_CONNECTION=postgresql://$POSTGRES_USER:$POSTGRES_PASSWORD@$POSTGRES_HOST:$POSTGRES_PORT/$POSTGRES_DB >> .env
echo SECRET_KEY=$(openssl rand -hex 32) >> .env
```

To run the web application in debug use:

```
alembic upgrade head
uvicorn app.main:app --reload
```

# Run tests


Tests for this project are defined in the ``tests/`` folder. 

This project uses [pytest](https://docs.pytest.org/) to define tests because it allows you to use the ``assert`` keyword with good formatting for failed assertations.


To run all the tests of a project, simply run the ``pytest`` command:
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

```
$ USE_LOCAL_DB_FOR_TEST=True pytest
```

Which will use your local database with DSN from the environment variable ``DB_CONNECTION``.


If you want to run a specific test, you can do this with [this](https://docs.pytest.org/en/latest/usage.html#specifying-tests-selecting-tests) pytest feature:

```
$ pytest tests/test_api/test_routes/test_users.py::test_user_can_not_take_already_used_credentials
```

# Deployment with Docker


You must have ``docker`` and ``docker-compose`` tools installed to work with material in this section.
First, create ``.env`` file like in `Quickstart` section or modify ``.env.example``.
``POSTGRES_HOST`` must be specified as `db` or modified in ``docker-compose.yml`` also.

Then just run:
```
docker-compose up -d db
docker-compose up -d app
```
Application will be available on ``localhost`` in your browser.

# Web routes

All routes are available on ``/docs`` or ``/redoc`` paths with Swagger or ReDoc.


# Project structure


Files related to application are in the ``app`` or ``tests`` directories.
Application parts are:

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