## Start by creating a new directory to build a dockerfile for your django app
```
cd ~/stud3-kubernetes
```
```
mkdir django-dockerfile
```
create the docker file using the following command
```
vim dockerfile
```
put the following into that file

```
# base image  
FROM --platform=linux/amd64 python:3.9

# setup environment variable  
ENV DockerHOME=/home/django

# set work directory  
RUN mkdir -p $DockerHOME  

# where your code lives  
WORKDIR $DockerHOME  

# set environment variables  
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1  

# copy whole project to your docker home directory. 
COPY ./studybud/ $DockerHOME 
COPY ./runserver.sh $DockerHOME
# install dependencies  

RUN chmod +x ./runserver.sh
RUN pip install --upgrade pip  

# run this command to install all python dependencies for the django app
RUN pip install -r requirements.txt  
# port where the Django app runs
EXPOSE 80/tcp
EXPOSE 80/udp
EXPOSE 5432

# start server  

CMD [ "/bin/bash", "-c", "./runserver.sh" ]
```
# Create a directory structure
``` 
mkdir studybud
```
Now move all the files from your django app into this directory so it looks like the following
```
~/django-dockerfile/
        Dockerfile
        runserver.sh
        studybud/
            asgi.py
            settings.py
            urls.py
            wsgi.py
            ...etc
```
# Change django app to work with external postgres
- now add the following to your settings.py in your django app to reach out to the external postgres

- add import os to the top of the settings.py look below for examples of both
```
import os
```
also make sure to add the following to the top of your settings.py 
```
ALLOWED_HOSTS = ["*"]
```

Replace the database settings with the following
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'django_db',
        'USER': 'django',
        'PASSWORD': os.environ.get('POSTGRES_PASSWORD'),
        'HOST': os.environ.get('DB_HOSTNAME'),
        'PORT': '5432',
    }
}
```
test you django app in a virtual environment

# Create the requirements.txt
```
pip freeze > requirements.txt
```
your requirements.txt should look like the example below
```
asgiref==3.4.1
autopep8==1.5.7
Django==3.2.7
django-cors-headers==3.8.0
djangorestframework==3.12.4
Pillow==8.3.2
psycopg2==2.9.6
pycodestyle==2.7.0
pytz==2021.1
sqlparse==0.4.2
toml==0.10.2
```
# Next we will push that to harbor using docker
first login to harbor using the following example
```
docker login <harbor-hostname>.<domain>
```
Now that you are logged in tag the image using the following format
```
docker tag <current-image-name-and-tag> <new-image-name-and-tag>
```
once you have docker tag completed make sure you docker push it by using the following command
```
docker push <harbor-hostname>.<domain>/stud<student-number>/<image-name>:v1
```
# Verify the image in harbor

1. Navigate to Harbor again in your browser
1. Navigate to the project with your student number
1. Select the image you uploaded and verify the image is in your student repository
1. Select the image and the select the SHA265 link
   * here you can see information about the image including the labels we added to the dockerfile when we built it