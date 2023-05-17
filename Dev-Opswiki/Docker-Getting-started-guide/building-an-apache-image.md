# Step 1 - Create working directory

* Within stud10-kubernetes directory, `mkdir docker` `cd docker`
* Within docker directory, `mkdir apache-dockerfile` `cd apache-dockerfile`

# Step 2 - Import HTML files for Apache

* In GitLab, navigate to Student_Resources/Module 6 - Kubernetes/Labs/Docker, download the directory into a ZIP.
  * Navigate to the saved ZIP, extract the html directory then drag into your `apache-dockerfile` directory.
  * The apache-dockerfile directory should look like this when running the `tree` command:

```shell
├── apache-dockerfile
│   └── html
│       ├── Chmod_Calculator_files
│       │   ├── bootstrap.min.css
│       │   ├── bootstrap.min.js.download
│       │   ├── custom.css
│       │   ├── favicon-16x16.png
│       │   ├── favicon-32x32.png
│       │   └── jquery.min.js.download
│       └── index.html
```

# Step 3 - Create Dockerfile

* Create dockerfile then edit the file: `touch Dockerfile` `nano Dockerfile`
* The completed dockerfile should look like this:

```
# Select the base image
FROM ubuntu:22.04

# Assign labels to the image
LABEL version="1.0" student-number='<student-number>'

# update the ubuntu repositories and install apache2
RUN apt-get update
RUN apt-get install -y apache2
RUN apt-get install -y apache2-utils
RUN apt-get clean

# Copy the nessecary files for the 
COPY html/ /var/www/html/

# Expose ports used by apache
EXPOSE 80

# Command to run when the container runs with this image
CMD apache2ctl -D FOREGROUND
```

# Step 4 - Build image

* Run command `docker build -t apache-server:v1 .`: instructs Docker to build an image named apache-server with the tag v1, using the Dockerfile and any other necessary files in the current directory.
* Run command `docker images | grep apache-server`: filter the output of the docker images command to only show images that contain the string "apache-server" in their repository name.
* Run command `docker image inspect apache-server:v1`: view image in JSON format

# Step 5 - Run container

* Run command `docker run -d -p 8080:80 --name=apache apache-server:v1`: Docker will start a container named "apache" based on the image name/tag "apache-server:v1" and expose it to port 8080 of the host machine.
  * If you need to stop the container: `docker stop apache`
  * If you want to start the container again: `docker start apache`
* Open browser, navigate to http://127.0.0.1:8080.
  * You should see a CHMOD calculator.