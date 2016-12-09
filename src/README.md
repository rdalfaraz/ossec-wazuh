# Jenkins configuration

## Introduction:
This is a copy of the configuration files used in production. Details about how create and configure this system can be found below.

## How run a Jenkins container:

- Install docker:
```sudo apt-get install docker.io```

- Add the user to docker group:
```sudo usermod -a -G docker myuser```

- Download the image of Jenkins using:
```docker pull jenkins```

- Make a new folder where the configuration of Jenkins will be stored

- Change the root repository folder group to docker group:
```sudo chgrp -R docker /new_jenkins_configuration_path```

- Run the container with the following command:
```docker run -d -p 8080:8080 -p 50000:50000 --cpuset-cpus 0 -v /new_jenkins_configuration_path:/var/jenkins_home jenkins```

Now, we should have a Jenkins container running and it could be accessed on the port: 8080 of the IP address of the host where Jenkins is running.

## Starting the Docker API:

You need to enable Docker API in order to have operating systems working as slaves in Jenkins:

- Open the following file:
```sudo vim /etc/default/docker```

- Add this line to the end of file:
```DOCKER_OPTS="-H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock"```

- Restart docker service
```service docker restart```

- Create a firewall rule in order to secure it (only allow connections from loopback or docker interface):
  + ```sudo iptables -A INPUT -p tcp --dport 4243 -i docker0 -j ACCEPT```
  + ```sudo iptables -A INPUT -p tcp --dport 4243 -i lo -j ACCEPT```
  + ```sudo iptables -A INPUT -p tcp --dport 4243 -j DROP```

##How create a docker image by a dockerfile:
In order to create a docker image using a dockerfile, follow the next steps:
- First install docker:
```sudo apt-get install docker.io```
- Clone this repository in the host
- Execute ```docker build -t [Image name decided] /chosen_dockerfile_path/.```
- Example:
+ ```docker build -t debian-jessie-slave Dockerfiles/Debian/Jessie/.```

The dockerfiles stored in this repository build Docker images of operating systems with the compilers and configuration needed to be used by Jenkins as slaves.
There are more information about the Docker images, used in production, in the part: Dockerfiles, in this repository.

##How use Docker containers as slaves in Jenkins:
First of all, we need to install some additional plugins. Select "Manage Jenkins" on the main page of Jenkins and go to "Manage Plugins" section, in the Available tab we have to download and install: Docker and Github plugins.
After of restart Jenkins, we need to create a credential to enable SSH connections between Jenkins and Docker containers.
So, in "Credentials" we have to create a new one with:
- Username: jenkins
- Password: MYPASSWORD
This credential is used for the Docker images that are created. To use other different it is need to rebuild the Docker iamges with the new credential in the dockerfile.
Once it is done, we are able to configure Jenkins, to use Docker containers as slaves.
In the bottom of the form of "Configure system" of "Manage Jenkins", we can create a new cloud. We have to select Docker and fill the main fields: "Docker URL" and "Container cap".
- Docker URL: In this field we must indicate the URL To access oru Docker server API (for example: tcp://172.16.42.43:4243)
- Container cap: It is the maximum number of containers that are allowed to run, at the same time.
It very useful to select "Test Connection" to know if Jenkins it is well connected to Docker.
Now that we have conecction with Docker, we can indicate the Docker images that Jenkins will use. After of select "Add Docker Template" we should fill the next fields:
- Docker Image: It is the name of the Docker image that we want use as slave. It must be exactly the same name that was given when it was built.
- Instance Capacity: It is the maximum number of container of this image that can be run.
- Remote Filing System Root: For our images it is /home/jenkins
- Labels: It is the name which is used to refer the Docker images in the Jenkinsfiles used.
- Usage: It must be selected "Only build jobs with label expressions matching this node"
- Launch method: Selected "Docker SSH computer launcher" we have to indicate the credential: jenkins, that previously we have created.
-	Remote FS Root Mapping: It is /var/jenkins_home
We are able to add as many Docker images as we need.

##Configuration of the pipelines used in production
The pipelines used in production have differents purposes. All of them work on the Wazuh repository on the master and the development branches.
There are 3 differents types of configurations.
- Daily pipelines: Execute the processes indicated on Jenkinsfile-daily on all the configured Docker images of Jenkins. Once a day, these pipelines are executed, first the pipeline which works on master branch and after the pipeline which works on the development branch.
- Instant pipelines: Execute the processes indicated on Jenkinsfile-instant on 2  Docker images of Jenkins (latest version of Debian an CentOS). These pipelines are configured to be triggered each time a change is pushed to Wazuh repository using GitHub Webhooks.
- Coverity pipelines: There are 4 pipelines, 2 works on master branch compiling the source with server and agent modes respectively; and the other 2 do the same but in the development branch. These pipelines are schedules to be launched once a day (from monday to thursday), because the results of the analysis of Coverity are overwritten which each execution.
