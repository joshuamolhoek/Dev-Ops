create your directory in your git repository for everything we do with docker

start by changing directory

cd \~/stud-kubernetes

mkdir docker

cd docker/

now that your in your docker directory create your directory to stage all the files for creating your application

mkdir django-postgress-dockerfile

then change into that directory

cd django-postgres-dockerfile

next create a file named Dockerfile in your django-postgres-dockerfile

touch Dockerfile

first tell docker what the base image will be using the FROM instruction (edit the Dockerfile you just created)

FROM postgres:14.5

then add labels to our image with the LABEL instruction

LABEL version="1.0" student-number=''

Now we can add the instructions to define environment variables inside the image using the commands

ENV POSTGRES_DB=django_db ENV POSTGRES_USER=django

Now we can tell the container which ports it is allowed to open up to the rest of the world with the EXPOSE instruction

EXPOSE 5432

your completed Dockerfile should look like:

notes

FROM postgres:14.5

notes

LABEL version="1.0" student-number='3'

notes

ENV POSTGRES_DB=django_db

ENV POSTGRES_USER=django

notes

EXPOSE 5432

now that the prep work is complete, you can build the container using Docker

run this command to build the container

docker build -t django-postgres:v1 .

Now that you have created an image. You will create a container to run the postgres container

docker images | grep django-postgres

Now you can also view the labels you added to your image in your Dockerfile with the following command

docker image inspect django-postgres:v1

use the following command to allow access to port 5432 from your laptop to the container

docker run -d --name postgres -e POSTGRES_PASSWORD=password -p 5432:5432 django-postgres:v1

To make sure we configured the database correctly we can log into the container and check the database with psql.

how to log into the container

`docker exec -it <container-id> bash`

Once inside the container run the following command to log into postgres as the django user we created

psql -U django -d django_db

use the following information to make sure the django_db database exists

\\l (lowercase L)

Your output should look like this

root@a6fb1a768f63:/# psql -U django -d django_db psql (14.5 (Debian 14.5-1.pgdg110+1)) Type "help" for help.

django_db=# \\l (lowercase L) List of databases Name | Owner | Encoding | Collate | Ctype | Access privileges -----------+--------+----------+------------+------------+------------------- django_db | django | UTF8 | en_US.utf8 | en_US.utf8 | postgres | django | UTF8 | en_US.utf8 | en_US.utf8 | template0 | django | UTF8 | en_US.utf8 | en_US.utf8 | =c/django + | | | | | django=CTc/django template1 | django | UTF8 | en_US.utf8 | en_US.utf8 | =c/django + | | | | | django=CTc/django (4 rows)

Lab complete

dont forget to create and close an issue on gitlab!!
Test