basic commands
after installing docker make sure that it installed properly by typing 'docker version' in the command line

how to run a docker application docker run 'hello world'

how to start a container with python embedded in it docker run -it python:3.9.5-slim-buster after running that command you can exit the session with quit() or Ctrl-D

you can also run a similar command to open an interactive bash session to do whatever you want inside the container docker run -ti python:3.9.5-slim-buster bash you can exit by typing 'exit'

how to run a container using ubuntu version 21.04 docker run -ti ubuntu:22.04 dont forget to update using the command apt-get update then install figlet apt-get install figlet then test figlet functionality figlet <enter-any-text-here>

how to see a list of all containers docker ps -a

how to delete a container docker rm <container-id>

how to start a container which has been stopped and attach the input to your terminal docker start -ai <container_id>

running a container as its own daemon docker run -d jpetazzo/clock

how to check the logs docker logs <container id>

how to stop a container docker stop <container id>

you can also kill a container docker kill <container id>