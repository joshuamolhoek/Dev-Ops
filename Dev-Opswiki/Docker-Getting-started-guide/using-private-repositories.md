# Add Your Image to a private Harbor Repository

In the previous lab, you created a Docker image which you will be using throughout this course. When you downloaded the base image in the Dockerfile, you were connected to the internet and had access to the docker.io repository. Now we want to stage our images for an air-gapped deployment by placing this image into a private repository.

## Step 1 - Get the harbor ca certificate

1. Set the DNS on your laptop to the upstream DNS server located in your student configuration file.

> **NOTE** Additionally, you can set the harbor hostname and IP in your `hosts` file. On Windows, this file is located in the `C:\Windows\System32\drivers\etc` directory.

1. Log into Harbor by going to the fqdn in your browser (You will not be able to connect to Harbor by IP address)
1. Use your student username and password to log in.
1. You will see a list of projects on the main screen after you log in.
   * Create a new project named `stud<student-number>`
   * Ensure the Public box is `unchecked`
   * Click on the project
1. Above the repositories select "REGISTRY CERTIFICATE" to download the ca cert.
1. On your laptop navigate to `C:\Users\<username>\.docker\certs.d\` and create a new directory named `<harbor-hostname>.<domain>`
1. Place the cert you downloaded in this new directory
1. Open the Docker Desktop UI and navigate to the debug menu. Restart Docker Desktop.
1. Docker Desktop will close and you may have to reopen it if it doesn't open back up automatically.

## Step 2 - Login in to Harbor with Docker

1. In a terminal type the following to log into Harbor

```shell
docker login <harbor-hostname>.<domain>
```

1. Enter your username and password

## Step 3 - Tag the image

Currently the image exists with a tag which will work for our local machine. This tag needs to be changed to match the repository we are pushing to. Essentially we are just renaming the image so we can push it to a new repository.

Image tagging is in the following format

```
docker tag <current-image-name-and-tag> <new-image-name-and-tag>
```

1. Tag the image we created in the previous lab

* If you were in the group which created the apache web server, use the following:

  ```shell
  docker tag apache-server:v1 <harbor-hostname>.<domain>/stud<student-number>/apache-server:v1
  ```
* If you were in the group which created the django postgres, use the following:

  ```shell
  docker tag django-postgres:v1 <harbor-hostname>.<domain>/stud<student-number>/django-postgres:v1
  ```

## Step 4 - Push the image

```shell
docker push <harbor-hostname>.<domain>/stud<student-number>/<image-name>:v1
```

## Step 5 - Verify the image in harbor

1. Navigate to Harbor again in your browser
1. Navigate to the project with your student number
1. Select the image you uploaded and verify the image is in your student repository
1. Select the image and the select the SHA265 link
   * here you can see information about the image including the labels we added to the dockerfile when we built it