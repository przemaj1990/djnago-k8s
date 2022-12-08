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

## First Step:


## First Step:


## First Step:

## Examples:
```
python3 create_ci.py
    sekiiatr00121
python3 clone_vm.py -s 'sekiius00202.exilis.seki.gic.ericsson.se' -u 'username_here' -p 'password_here!' --disable-ssl-verification -v 'sekiiatr00121' --template 'sekiiatr' --datacenter-name 'VxRail-Datacenter' --vm-folder 'Exilis ISP' --datastore-name 'VxRail-Virtual-SAN-Datastore-4e7f19e1-4b96-4933-bd2c-082b768b57f4' --cluster-name 'VxRail-Virtual-SAN-Cluster-4e7f19e1-4b96-4933-bd2c-082b768b57f4'
    cloning VM...
    VM cloned.
python3 get_mac.py -v 'sekiiatr00121'
    00:50:56:a5:7a:73
    00:50:56:a5:5f:b8
python3 update_hydra_mac.py sekiiatr00121 "mgmt" 00:50:56:a5:7a:73 "iac" 00:50:56:a5:5f:b8
python3 pxe_lxgen.py sekiiatr00121 00:50:56:a5:7a:73
    Added server  sekiiatr00121  with mac address 00:50:56:a5:7a:73 {'mac': '00:50:56:a5:7a:73', 'data': 'profile_home=lxgen profile=eis-eis-ubuntu-18.04 ip=dhcp ksdevice=bootif e_networking=dhcp hostname=sekiiatr00121 e_fact_team=estestteamcloudrancicd e_fact_profile=isp-server-hall-d e_fact_site=SERO e_fact_groups=base e_software=medium e_puppet_server=puppet.sero.gic.gricsson.se e_puppet_caserver=puppetca.sero.gic.ericsson.se e_rootpassword=$1$bc4RC4Z3$xTJ3EOyqzUQp7sRGdhxds/'}
    {"message":"Mac created!"}
    200
```

## Authentication:

We used few way to use credentials inside pipline as well as scripts, all of them could be used seperatly or together and in effect give u possibility to achive similar effects:

- Code from Andriej to get token for hydra:
'''
def get_hydra_config():
    config_file_pathname = os.path.expanduser('~') + "/config.ini"
    if not os.path.isfile(config_file_pathname):
        sys.exit('Could not find config.ini with hydra_api_url and hydra_token') 
    config = configparser.ConfigParser()
    config.read(config_file_pathname)
    if not config.has_section('hydra'):
        sys.exit('could not find hydra section in config.ini')
    return config
def instantiate_hydra():
        config = get_hydra_config()
        hydra_url = config.get(section='hydra', option='hydra_api_url')
        token = config.get(section='hydra', option='hydra_token')
        return HydraLibrary(hydra_url=hydra_url, token=token)
'''
- in Jenkins pipline we used "VCENTER_CREDENTIALS = credentials('vcenter')" to access credential stored in secret key inside Jenkins, further in pipline we can access them using: '${env.VCENTER_CREDENTIALS_USR}' and '${env.VCENTER_CREDENTIALS_PSW}'.
- we can also access the same credential from inside python scrpt using 'vcenter_user = os.environ['VCENTER_CREDENTIALS_USR']'

## Faced problems:

- We need to remember that we need to run pipline in production enviroment. In other case created VM will not receive ip address and in effect it will also not receive OS via PXE boot on the first boot. 
- Way of using and update enviroment variable in each stage of pipline: When you issue sh directive, a new instance of shell (most likely bash) is created. As usual Unix processes, it inherits the environment variables of the parent. Your bash instance is then running your script. When your script sets an environment variable, the environment of bash is updated. Once your script ends, the bash process that ran the script is destroyed, and all its environment is destroyed with it. Example:
'''
script{
  def script_output = sh(returnSTdout: true, script: """
source /dev/bin/activate
python3 /dev/script.py
""")
script_output = script_output.trim()
env.VALUUE = script_output
echo $env.VALUE
'''


- For future work it will be good to have one standard repo of passwords.
