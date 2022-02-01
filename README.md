
 Conversation opened. 1 read message.

Skip to content
Using Think Future Technologies Pvt. Ltd. Mail with screen readers
21 of 4,479
readme
Inbox

Amandeep Singh
Attachments
Tue, Feb 1, 11:06 AM (13 hours ago)
to me




Amandeep Singh
 Think Future Technologies Pvt Ltd	
email: singh.amandeep@tftus.com
mobile: +91 9650854465
Attachments area
# CybintIfrastructure
## Overview
Cybint LTI Provider provides Desktop-as-a-Service(DaaS) to the Users(Learners) with custom images accessible over the browser.

The Scope for the Cybint’s VM Platform is to provide a LMS provider solution for providing required virtual machines
with a smooth interface via remote connection over web browser (no software installation required) for the courses a 
LMS learner has opted for with SSO (single sign on) support via LTI.

## Installation

We have two ways to install this project as described below:
- Using Docker
- Orthodox Development Base
### Docker Installation
#### Prerequisites:
- Docker

Git clone the project:
```
$ git clone https://github.com/CybintLabs/CybintIfrastructure.git
```
Rename the following files
```
$ rm docker-compose.yaml
$ mv env.stag.sample env.stag
$ mv env.web-app.sample env.web-app
$ mv env.worker.sample env.worker
$ mv env.beat.sample env.beat
$ mv docker-compose-dev-sample.yaml docker-compose.yaml
```



Now change the database configurations according to your system database or keep default as given below in all env prefixed files:
```
POSTGRES_DB="name of database"
POSTGRES_USER="username in postgres"
POSTGRES_PASSWORD="password of the above user"
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
```

Configure AWS Keys in env.web-app and env.worker:

Replace the 'xxxxxxxxxxx' with original keys
```
AWS_ACCESS_KEY_ID=xxxxxxxxxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxx
```

####Just run the command for Permission and docker and you are good to go.
```
sudo chmod 777 /var/run/docker.sock
```
```
docker-compose up -d --build
```
#### Images build by above docker command 
- python:3.6-alpine
- postgres:12.2-alpine
- redis:latest
- web(image of application)
- workers(celery worker and celery beat worker)

This will start up your containers with above images and you will be able to use the application on
 http://localhost:8000/
 
After this setup follow the process given after installation section.

### Development Installation 
#### Prerequisites:
1. Postgresql 12.2
2. Redis
3. Pyhton 3.6
4. Pipenv
5. AWS CLI

To configure AWS CLI you can visit https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html and follow the simple steps

Git clone the project:
```
$ git clone https://github.com/CybintLabs/CybintIfrastructure.git
```
Configure Environment file:
```
$ mv .env.dev.sample .env
```
Configure Database variables in .env file:
```
POSTGRES_DB="name of database"
POSTGRES_USER="username in postgres"
POSTGRES_PASSWORD="password of the above user"
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
```

You also need to change some varibles given below:
```
ENV_PREFIX=DEV
PEM_FILE_PATH=./key/cybint_staging.pem
PUBLIC_IP=True
```

In CybintIfrastructure/settings.py change Debug=True


Run the following commands in pip environment to create a development environment
  ```
$ pipenv shell

Customized Django beat is installed before any requirements.Please follow the commands given below:
$ tar -xf local_packages/django-celery-beat-2.0.0.tar.xz -C local_packages/
$ cd local_packages/django-celery-beat-2.0.0 && python setup.py build && python setup.py install
$ cd -

$ pip install -r requirements.txt
```
  Migrate the tables in your database
  ```
$ python manage.py migrate
```
  Load Fixtures(sample)
  ```
$ python manage.py loaddata user/fixtures/load_user_role_admin.json
$ python manage.py loaddata user/fixtures/load_celery_beat.json
$ python manage.py loaddata admin/region/fixtures/load_region_az.json
$ python manage.py loaddata admin/machine/fixtures/load_machine.json
$ python manage.py loaddata admin/settings/fixtures/load_global_settings.json
$ python manage.py loaddata admin/node_group/fixtures/load_node.json 
``` 
Run the server
```
$ python manage.py runserver 0.0.0.0:8000
```
Run the workers and beat workers
```
$ celery -A vm_tasks worker  --loglevel=info --autoscale=10,2 -n vm_worker.%h
$ celery -A vm_tasks beat -l info --scheduler django_celery_beat.schedulers:DatabaseScheduler
```
 Now check that application is running on http://localhost:8000/
 
 ## Configuration as LTI Provider
 The URL http://localhost:8000/ will not redirect you to landing page as this is a customized LTI provider we have URL as 
 http://localhost:8000/lti/auth/ which will be included in LMS consumer in LTI Tool link.
 The keys are also to be included as:
 ```
Key Name  | Value
--------- | -------------
Key       | Value in settings File 
Secret    | Value in settings file
```
Change/add the values of following in .env
- LTI_KEY
- LTI_SECRET

After adding the above keys you need to add custom parameter as lab_id="Value generated at the the time of lab creation from
admin"

To get lab_id create a lab in Admin panel by accessing  http://localhost:8000/cybint_admin/login/
- Email : admin@admin.com
- Password : 123456
- Now Create a Lab from Lab utility -> Lab management -> Create

The development Environment is now set. Access the LTI Provider through your LMS Consumer.

## Important Note
- You will be needing a public IP with SSL enables to trigger LTI request from LMS as it works on HTTPS
- Replace localhost or 0.0.0.0 and add your Public IP with SSL else only admin panel will work not LTI Tool

## Dependencies
All the dependencies with there versions can be found in requirements.txt
But here are some major libraires that are required in the same way as described below

AWS is used as cloud infrastructure with the help of boto3 api and following are major dependencies
```
awscli==1.18.41
boto==2.49.0
boto3==1.12.41
botocore==1.15.41
celery==4.4.2
```

Sockets also play a major role in the application for that we have ASGI server with redis channel and async
```
asgiref==3.2.7
async-timeout==3.0.1
channels==2.4.0
channels-redis==2.4.2
```

Django is our main frame work which run with the help of Uvicorn with following dependencies
```
Django==3.0.5
uvicorn==0.11.5
django-extensions==2.2.9
django-adminlte2==0.4.1(AdminLTE is library for template used in Admin)
```

LTI provider tool dependencies are activated in application through the following:
```
django-lti-auth==1.0.0
django-lti-provider==0.4.0
PyLTI==0.5.1
```
Guacamole is major part to play to show Virtual machine through various protocols(RDP.VNC,SSH)
```
guacapy==0.9.5
```

Authentication is major requirement of the LTI as well as Admin 
```
oauth2==1.9.0.post1
oauthlib==3.1.0
cryptography==2.9.2
```

Connection with postgresql to access ORM in Django the following library is required
```
psycopg2-binary==2.8.5
```

To use Celery for workers we need celery and redis

```
redis==3.4.1
celery==4.4.2
```

## Credits

Development Team 
- Amandeep Singh(@singh-amandeep)
- Rahul Verma(@rahulv3993)
- Shashank Rawat(@shashanktft)
- Aditya Kumar(@adityatft)
- Shubham Kumar Gupta
  
## Contribution
Have you spotted a typo, would you like to fix a link, or is there something you’d like to suggest? 
Browse the source repository of this article and open a pull request. I will do my best
to review your proposal in due time.
readme.md
Displaying readme.md.
