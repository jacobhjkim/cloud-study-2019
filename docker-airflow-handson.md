# Docker + Airflow

## What is Airflow?
https://airflow.apache.org

## Steps
### 1. Pull Docker image
```
docker pull puckel/docker-airflow
```

### 2. Install Extra Airflow 
```
docker build --rm --build-arg AIRFLOW_DEPS="datadog,dask" --build-arg PYTHON_DEPS="flask_oauthlib>=0.9" -t puckel/docker-airflow .
```

### 3. Run Docker
```
docker run -d -p 8080:8080 puckel/docker-airflow webserver
```

Check `http://localhost:8080/admin/`
![alt text](https://raw.githubusercontent.com/TheJacobKim/cloud-study-2019/master/photos/airflow_default.png)

**Use Cases**  
Slack Notification Image  
![alt text](https://raw.githubusercontent.com/TheJacobKim/cloud-study-2019/master/photos/airflow_slack.png)


### 4. Stop Docker 
```
docker stop $(docker ps -a -q)
```

### 5. Airflow CeleryWorker
```
version: '2.1'
services:
    redis:
        image: 'redis:5.0.5'
        # command: redis-server --requirepass redispass

    postgres:
        image: postgres:9.6
        environment:
            - POSTGRES_USER=airflow
            - POSTGRES_PASSWORD=airflow
            - POSTGRES_DB=airflow
        # Uncomment these lines to persist data on the local filesystem.
        #     - PGDATA=/var/lib/postgresql/data/pgdata
        # volumes:
        #     - ./pgdata:/var/lib/postgresql/data/pgdata

    webserver:
        image: puckel/docker-airflow:1.10.4
        restart: always
        depends_on:
            - postgres
            - redis
        environment:
            - LOAD_EX=n
            - FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
            - EXECUTOR=Celery
            # - POSTGRES_USER=airflow
            # - POSTGRES_PASSWORD=airflow
            # - POSTGRES_DB=airflow
            # - REDIS_PASSWORD=redispass
        volumes:
            - ./dags:/usr/local/airflow/dags
            # Uncomment to include custom plugins
            # - ./plugins:/usr/local/airflow/plugins
        ports:
            - "8080:8080"
        command: webserver
        healthcheck:
            test: ["CMD-SHELL", "[ -f /usr/local/airflow/airflow-webserver.pid ]"]
            interval: 30s
            timeout: 30s
            retries: 3

    flower:
        image: puckel/docker-airflow:1.10.4
        restart: always
        depends_on:
            - redis
        environment:
            - EXECUTOR=Celery
            # - REDIS_PASSWORD=redispass
        ports:
            - "5555:5555"
        command: flower

    scheduler:
        image: puckel/docker-airflow:1.10.4
        restart: always
        depends_on:
            - webserver
        volumes:
            - ./dags:/usr/local/airflow/dags
            # Uncomment to include custom plugins
            # - ./plugins:/usr/local/airflow/plugins
        environment:
            - LOAD_EX=n
            - FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
            - EXECUTOR=Celery
            # - POSTGRES_USER=airflow
            # - POSTGRES_PASSWORD=airflow
            # - POSTGRES_DB=airflow
            # - REDIS_PASSWORD=redispass
        command: scheduler

    worker:
        image: puckel/docker-airflow:1.10.4
        restart: always
        depends_on:
            - scheduler
        volumes:
            - ./dags:/usr/local/airflow/dags
            # Uncomment to include custom plugins
            # - ./plugins:/usr/local/airflow/plugins
        environment:
            - FERNET_KEY=46BKJoQYlPPOexq0OhDZnIlNepKFf87WFwLbfzqDDho=
            - EXECUTOR=Celery
            # - POSTGRES_USER=airflow
            # - POSTGRES_PASSWORD=airflow
            # - POSTGRES_DB=airflow
            # - REDIS_PASSWORD=redispass
        command: worker
```
http://localhost:8080/admin/


### 6. Docker compose
```
docker-compose -f docker-compose-CeleryExecutor.yml run --rm webserver airflow list_dags
```

