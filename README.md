# Gitlab-Docker-Demo
Demo about setting up gitlab on docker, include gitlab-ce , gitlab-runner, private docker repo , ci-di examples.

## 1. Install Docker
&nbsp;&nbsp;The docker community offers a full [guidance](https://docs.docker.com/install/) about installing docker on different os,you can follow these steps on the guidance.
Read carefully about the section **What to know before you install** if you are mac or windows user.
### 1.1 Your First Docker Application
After installing docker on your machine,you can try some docker command like `docker info` or run a app on docker like `docker run hello-world`.You can get a [*Get Started*](https://docs.docker.com/get-started/) tutorial on the official websites
## 2. Run the Gitlab Image in Docker Engine
&nbsp;&nbsp;Since this repo is a demo about setting up gitlab, we choose the free version of gitlab.Gitlab community provide us a official doc about how to run gitlab image in docker.
> sudo docker run --detach \
   	--hostname docker.for.mac.host.internal \
   	--publish 443:443 --publish 80:80 --publish 22:22 \
   	--name gitlab \
   	--restart always \
   	--volume /srv/gitlab/config:/etc/gitlab:Z \
   	--volume /srv/gitlab/logs:/var/log/gitlab:Z \
   	--volume /srv/gitlab/data:/var/opt/gitlab:Z \
   	gitlab/gitlab-ce:latest

&nbsp;&nbsp;Remember,local locations in volume option(*/srv/gitlab/config* and so on)
depends on your machine,if you don't have these directories,you may change the command above or create these directories manually. &nbsp;
<br/>
&nbsp;&nbsp;Now you can run `docker container ls` to see if gitlab runs and visit [gitlab homepage](http://localhost) to login to gitlab.

## 3.Config Gitlab Runner
### 3.1 Run Gitlab Runner in Docker Engine
if you want to set up ci pipeline,you should set up gitlab runner first.[this doc](https://docs.gitlab.com/runner/install/docker.html)
tells us how to run gitlab runner in docker engine.
> docker run -d --name gitlab-runner --restart always \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /srv/gitlab-runner/config:/etc/gitlab-runner:Z \
    gitlab/gitlab-runner:latest
    
Pay attention to the directory */srv/gitlab-runner/config*,create it if necessary.
### 3.2 Config Runner
&nbsp;&nbsp;Open [gitlab admin page](http://localhost/admin/runners) to get url and token,you need these two to config your gitlab runner.
<br />
&nbsp;&nbsp;Login to the gitlab runner docker, you can use `docker container ls` to get the container id of gitlab runner.
>  docker exec -it f28aaf921bf9 /bin/bash

`f28aaf921bf9` is the container id of gitlab runner on my laptop.

&nbsp;&nbsp;After you have logged in the container,you can config your gitlab runner now.
>gitlab-runner register -n   --url http://docker.for.mac.host.internal    --registration-token ${your token}   --executor docker   --description "My Docker Runner"   --docker-image "docker:latest"   --docker-volumes /var/run/docker.sock:/var/run/docker.sock

&nbsp;&nbsp;You want to access host machine's localhost in a docker container(gitlab runner),but `http://localhost` seems  not working on mac,so  use `docker.for.mac.host.internal` instead.You may find more info in [this question](https://stackoverflow.com/questions/31324981/how-to-access-host-port-from-docker-container/43541732).
<br />
&nbsp;&nbsp;Now you can go to [gitlab admin page](http://localhost/admin/runners) again to see if the runner has been registered or not.
### 3.3 Build Customer Gitlab Runner Image
&nbsp;&nbsp;If we use the official gitlab runner image,we need to login to the container and configure our gitlab runner.We can build our customer image to config gitlab runner when container start.Go to [Gitlab Runner Customer Image](https://github.com/wendy260310/Gitlab-Docker-Demo/tree/master/Gitlab-Runner-Customer-Image) and build our gitlab runner image.When you run our customer image,you will find that a new runner was added to our gitlab in the [admin page](http://localhost/admin/runners).
<br/>
&nbsp;&nbsp;Don't forget to change the token in `entry.sh`
## 4.Set Up Private Docker Repository
&nbsp;&nbsp;We need to set up our private docker repository if we want to build our app in one container and deploy our app in another container.[this tutorial](https://docs.docker.com/registry/deploying/) shows us the command to set up a local docker repository
> docker run -d -p 5000:5000 --restart=always --name registry registry:2

## 5.Your First Gitlab Application
Now,after steps above,we started to create a gitlab application,build and deploy it.
### 5.1 create application
&nbsp;&nbsp;Go to [new project](http://localhost/projects/new) and choose the `Create from template` tab,we choose spring app as our demo.