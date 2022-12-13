## General Description:

I would like to deploy Django & Postgresql on k8s in automatic way with DevOps methodology.

# Links to external sources:

Deploy Django into Production with Kubernetes, Docker, & Github Actions. Complete Tutorial Series:
- https://www.youtube.com/watch?v=NAOsLaB6Lfc&t=14254s&ab_channel=CodingEntrepreneurs

How To Deploy a Scalable and Secure Django Application with Kubernetes:
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-scalable-and-secure-django-application-with-kubernetes 
- https://github.com/do-community/django-polls

Deploy a Django app with Kubernetes in 20 minutes:
- https://www.architect.io/blog/2022-08-04/deploy-python-django-kubernetes/ 

How to Deploy a Django Application With Kubernetes:
- https://betterprogramming.pub/how-to-deploy-a-django-application-with-kubernetes-f5814b0688bf 

How To Deploy A Django App Over A Kubernetes Cluster (With Video):
- https://medium.com/@tech_with_mike/how-to-deploy-a-django-app-over-a-kubernetes-cluster-with-video-bc5c807d80e2

How to orchestrate your Django application with Kubernetes:
- https://mattermost.com/blog/orchestrate-django-application-with-kubernetes/

Deploy a Scalable and Secure Django Application with Kubernetes:
- https://bobcares.com/blog/deploy-a-scalable-and-secure-django-application-with-kubernetes/

Django With CI/CD (Docker Container & Kubernetes):
- https://uzzal2k5.github.io/django-docker/

Some tips to deploy Django in kubernetes:
- https://www.jujens.eu/posts/en/2021/Mar/29/deploy-django-kubernetes/

Kubernetes Guide - Deploying a machine learning app built with Django, React and PostgreSQL using Kubernetes:
- https://datagraphi.com/blog/post/2021/2/10/kubernetes-guide-deploying-a-machine-learning-app-built-with-django-react-and-postgresql-using-kubernetes



## First Step: Initial Configuration & Fight with GitLab:
Start Based on: based on: https://www.youtube.com/watch?v=SlHBNXW1rTk&amp;list=PLEsfXFp6DpzRMby_cSoWTFw8zaMdTEXgL

```
alias k='kubectl'
sudo apt install python3-pip
touch requirements.txt
-create branch remotly on gitlab
export GIT_SSH_COMMAND="ssh -i ~/.ssh/id_rsa1"
git clone git@gitlab.internal.ericsson.com:emajprz/backup_and_draft.git
git add .
git commit -m "first commit"
git push -u origin main
git add .
git commit -m "initial setup"
git branch -M Django_on_k8s
git push -uf origin Django_on_k8s
touch .gitignore
```

## Create Python VIrtal Env:

```
python3.8 -V
kubectl version --client
python3.8 -m venv venv
source venv/bin/activate
pip freeze
pip install pip --upgrade
touch requirements.txt
pip install -r requirements.txt 

```

## DJango - Start Projects & enviroment variable handling:

```
cd web
django-admin startproject django_k8s .
touch .env 
- add code
python3.8 manage.py shell
import os
print(os.environ.get("REGION"))
```

```git add -all && git commit -m 'something' ```

## Start with Docker:

We need to have docker file to build application correctly from dir:
```
FROM python:3.8.10-slim

COPY . /app
WORKDIR /app

RUN python3 -m venv /opt/venv

RUN /opt/venv/bin/pip install pip --upgrade && \
    /opt/venv/bin/pip install -r requirements.txt && \
    chmod +x entrypoint.sh

CMD ["/app/entrypoint.sh"]
```
Then we need to have a way to run app (using sh script):
```
#!/bin/bash
APP_PORT=${PORT:-8000}
cd /app/
/opt/venv/bin/gunicorn --worker-tmp-dir /dev/shm django_k8s.wsgi:application --bind "0.0.0.0:${APP_PORT}"
```

#to run the same as endpoints.sh
`/home/terra/emajprz/backup_and_draft/dev/django-k8s/venv django_k8s.wsgi:application`

## Migration Script:
```
#!/bin/bash

SUPERUSER_EMAIL=${DJANGO_SUPERUSER_EMAIL:-"przemyslaw.majdanski@ericsson.com"}
cd /app/

/opt/venv/bin/python manage.py migrate --noinput
/opt/venv/bin/python manage.py createsuperuser --email $SUPERUSER_EMAIL --noinput || true
```
- we igonore: python manage.py collectstatics as this can be done later (and many times for many app)
- we cannot run migration in docker file as it need to have access to database ;) to work

## Docker Compose part I:

- check used ports: sudo lsof -i -P -n | grep 8000
- docker-compose up & down
- docker-compose up --build
- I faced problem with pull images: docker login && docker pull <image> & docker image 
- generaly creating docker-compose file
- problem with connection between django & posgres solved: When we create docker-compose, name of app: postgres_db must be the same as POSTGRES_HOST from .env. As Django need to have a way to communicate, and docket-compose used this name of app ;) 

## Docker Compose part II:

- `expose` allow connection between pods.
- `port` handle connection with localhost
- to check somethin in shell django:
```
python3 manage.py shell
from django.conf import settings
print(settings.DB_IS_AVAIL)
```

## Differences & additional notes:

CoddingEntrepreneurs use DigitalOceans to privision k8s and posgresql. In our case we will prepare everything in k8s.
Usefull tips:
`export KUBECONFIG=/path/to/kubeconfig.yaml` this will allow to manage k8s
- we can add to workspace confgi file necessary entry so it will automatically load correct kubeconfig file and allow use manage correct k8s, in prod is good to have /.kube and there stored kubeconfig.yaml
- we can find config file in: `less ~/.kube/config`

## Kubernetes:

`kubectl apply -f dev/django-k8s/k8s/deployment.yaml`
`k run nginx-deplo --image=armdocker.rnd.ericsson.se/dockerhub-ericsson-remote/library/nginx:1.23.1-alpine --restart=Never`
`k get deployments nginx -o yaml`
`k get pods`
`k describe deployment <name>`
`k logs <name>`
end on: https://www.youtube.com/watch?v=NAOsLaB6Lfc&t=14254s&ab_channel=CodingEntrepreneurs

## New try on Kubernetes, resetup of this project on new enviroment:

/home/cloud_user/Projects/Djnago-on-k8s/djnago-k8s/dev/django-k8s <- create here venv using:
`python3.8 -m venv venv`
`sudo apt install python3.8-venv` <- solved problem with virtualenv
`source venv/bin/activate `
`pip install pip --upgrade`
`pip install -r requirements.txt`
- I had some problem with installin vnev but finally after reinstall it start to work
`source venv/bin/activate`
`sudo docker-compose up --build`
- there was issue to bring down docker compose and I solved this using: sudo aa-remove-unknown (base on: https://stackoverflow.com/questions/47223280/docker-containers-can-not-be-stopped-or-removed-permission-denied-error)
`sudo docker-compose up --build`











## First Step:


## First Step:

## Examples:

