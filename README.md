# Labrob Tiago Docker
guide to install the public workspace of tiago using docker

## Table of Contents
* [Installation](#installation)
    - [Docker-Engine](#docker-engine)
    - [Rocker](#rocker)
    - [Tiago Image](#tiago-image)
* [Creation of a Container](#creation-of-a-container)
* [Usefull Docker Commands](#usefull-docker-commands)



## Installation


### Docker-Engine
From the [official site](https://docs.docker.com/engine/install/), here will be reported the procedure to install it in ubuntu 20.04, 22.04, 24.04.

Run the following command to uninstall all conflicting packages.
```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
1. Set up docker `apt` repository.
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
2. Install the latest version of Docker packages.
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
3. Verify that the Docker Engine installation is successful by running the `hello-world` image.
```
sudo docker run hello-world
```

By default you’re required to use `sudo` to run Docker commands. Shortly, it is because you need to add Docker to the groups. More information can be found [here](https://docs.docker.com/engine/install/linux-postinstall/).
1. Create the `docker` group
```
sudo groupadd docker
```
2. Add your `user` to the docker group.
```
sudo usermod -aG docker $USER
```
3. Log-out and log-back to activate so that the group membership is re-evaluated (may be necessary to restart the PC). 
4. Verify that you can run `docker` commands without `sudo`.
```
docker run hello-world
```


### Rocker
[This repository](https://github.com/osrf/rocker) contains the tool to run docker images with customized local support injected for things like nvidia support. And user id specific files for cleaner mounting file permissions.

Rocker is available via pip you can install it via pip using
```
pip install rocker
```
After the installation you need to restart your PC


### TIAGo Image
The TIAGo image can be pulled from `docker-hub` with the command
```
docker pull francescod98/labrob_tiago:noetic
```

The base image is the `palroboticssl/tiago_tutorials:noetic` in which [blasfeo](https://github.com/giaf/blasfeo), [hpipm](https://github.com/giaf/hpipm), [acados](https://docs.acados.org/) and [pinocchio](https://stack-of-tasks.github.io/pinocchio/) have been already installed



## Creation of a Container

The goal is to create the controller starting from the downloaded image. To do it, the rocker command is used that has multiple options to use.
the most importants are the following
 - `--home` mount the user home diretcory in the container.
 - `--user` mount the user's id and run as that user.
 - `--network {bridge,none,host}` select which network to use. Must be set to `host` in case of experiments with the real robot.
 - `--nocleanup` don't remove the docker container when stopped.
 - `--nvidia` enables Nvidia docker support

In case of Nvidia graphic support mounted on the PC, you would need first to install the [Nvidia Docker Support](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) with the following commands (installation with apt)
```
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```
and then you need to configure docker running
```
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

In order to maintain the container you need to use the `--nocleanup` flag and you need to target a TMPDIR as an environmental variabale, otherwise some persistent files used to open the container will be put in the /tmp folder, and then deleted when the PC is turned off. For instance, a possible choise is run
```
mkdir ~/tmp_docker_files
export TMPDIR=~/tmp_docker_files
```

Finally the container is created using the command
```
rocker --home --user --x11 --privileged --nocleanup francescod98/labrob_tiago:noetic --devices /dev/dri/card0 
```
or, in case of Nvidia support, with
```
rocker --home --user --nvidia --x11 --privileged --nocleanup francescod98/labrob_tiago:noetic --devices /dev/dri/card0
```
If the procedure ends succesfully the terminal enters in the container just created.



## Usefull Docker Commands

Some usefull docker commands
 - `exit`: exits from the container in the current terminal. If the container is open in only one terinal, it is stopped
 - `docker ps -a`: lists all the container that runs in the PC
 - `docker help`: to see all the basic commands of docker such as rename, delete, or list containers or images
 - `docker start -ia <CONTAINER_NAME>`: to start the container and enters inside its terminal. <CONTAINER_ID> can be used also, that can be seen from the `docker ps -a` command
 - `docker exec -it <CONTAINER_NAME> bash`: to enter in the terminal of a running container, such as from another terminal. In any case is better to use the `terminator -u` inside a container to open a terminal that can be splitted at will to manage multiple terminals simultaneously.
