<ins>Table of contents</ins>
- [Notes, goals, refernce](#notes-goals-reference)
- [install process](#installation)
- [Static IP setup](#set-static-IP)
- [setup docker swarm](#docker-swarm-setup)
- [Setup docker NFS volume](#setup-docker-nfs-volume)
- [Setup Portainer](#setup-portainer)
# Notes, goals, reference
Promox host

VM image of Ubuntu 20.04 server

portainer swam deployment

[How-To: Deploy Portainer on Docker Swarm](https://www.youtube.com/watch?v=L80QDuix5RE&ab_channel=PortainerIO)

[https://docs.portainer.io/v/ce-2.11/start/install/server/swarm/linux](https://docs.portainer.io/v/ce-2.11/start/install/server/swarm/linux)

(reference video)[https://youtu.be/L80QDuix5RE]

# Installation


1. Install pre requests
```
apt-get install \
ca-certificates \
curl \
gnupg \
lsb-release \
net-tools \
nfs-common \
docker-compose
```
2. Update server and install upgrades
```
apt update
```
```
apt upgrade
```
3. Download docker install file
```
curl -fsSL https://get.docker.com -o get-docker.sh
```
4. Run the installer
```
sh get-docker.sh
```
5. Check Docker installation
```
docker ps
```
```
systemctl status docker
```
6. Enable docker on system startup
```
systemctl enable docker
```
7. make a directory to connect the NFS shares to
```
cd
```
```
mkdir /mnt/docker
```
8. add the mount for NFS share
> the Folders must be created in the share before you can map them for docker
```
mount –t nfs 192.168.1.X:/<Path to NFS share> /mnt/docker
```
9. check the mount
```
mount -l | grep nfs
```
10. Add the mount information in the fstab so it remains mounted even after a reboot
```
nano /etc/fstab
```
- add (examples below):
```
192.168.1.X:<Path to NFS Share> /mnt/docker  nfs      defaults    0       0
```

11. Test the mount share
```
cd /mnt/docker
```
```
touch test.txt
```
```
nano test.txt
```
> add some text to the .txt file and save and close

Run ls command to see what is in the mounted directory
```
ls -l
```
```
cat test.txt
```

## set static IP

```
cd /etc/netplan
```
```
nano <filename>config.yaml
```
Add:
```
network:
  ethernets:
    ens18:
      dhcp4: false
      addresses: [192.168.1.X/24]
      gateway4: 192.168.1.X
     nameservers:
       addresses: [8.8.8.8 , 1.1.1.1]
  version: 2
```
```
netplan apply
```
Reboot the machine

Login then run command to see if your new IP is set
```
ifconfig
```

## Docker Swarm setup
```
docker swarm init
```
> If you get an error
```
docker swarm init --advertise-addr 192.168.1.X
```
Copy the resulting command to add any additional WORKER nodes

> To add a manager to the node you will need a token command like for the worker node

run:
```
docker swarm join-token manager
```

Check the node status:
```
docker node ls
```

## Setup Docker NFS Volume

Run: (examples below)
``` UNRAID docker command
docker volume create --driver local \

  --opt type=nfs \

  --opt o=addr=192.168.1.X,rw \

  --opt device=:/<Path to NFS Share> \

  <Name for NFS Volume>
```

check the docker volumes

List the volumes
```
docker volume ls
```

inspect details of a specific volume
```
docker volume inspect nfs-volume
```

## Setup Portainer

There are a number of ways to setup portainer. You can run as a single docker command. But recommended way is to use a yaml file to deploy portianer as a stack. Running portainer as a stack will allow you to utilize the full swarm.

```
docker run -d -it --name portianer --mount source=nfs-volume,target=/data portainer/portainer-ce:latest
```
> Instead of using latest specify the newest version
>> Update the source for the NFS with the correct volume name, must already be mapped as docker volume

Error from docker run command?
```
docker pull portainer/portainer-ce:latest
```
> instead of latest replace with the newest version number.
Verify everything is running
```
docker inspect [container name]
```
> replace [contianer name] with the --name specified in the command

Validate docker portainer service is running
```
docker service ls
```

### Docker compose deployment
This process requires docker-compose to be installed on the host

Create a docker-compose.yml file
```
touch docker-compose.yml
```
Edit the yml file
```
nano docker-compose.yml
```
Launch the deployment
```
docker-compose up
```
Take down deployment
```
docker-compose down
```
> Experienced issues deploying this way. *ERROR* unable to retrieve a list of IP associated to the host

### Deploy with Docker Stack
This method is *recommended*
download a yml template to edit
```
curl -L [https://downloads.portainer.io/portainer-agent-stack.yml](https://downloads.portainer.io/portainer-agent-stack.yml) -o portainer-agent-stack.yml
```
Edit the file just downloaded:
```
nano portainer-agent-stack.yml
```
Add the values for the NFS share which is required for the docker stack to be HA
```
driver: local  
    driver_opts:  
      type: nfs  
      o: addr=192.168.1.X,rw  
      device: ":<Path to NFS Share>"
```
> also update the version under image: value to the. DO NOT USE "latest"
Deploy the docker stack
```
docker stack deploy -c portainer-agent-stack.yml
```

Verify deployment
```
docker service ls
```
```
docker stack ls
```

Remove the stack deployment
```
docker stack rm [deployment name]
```
> update [deployment name] to the same name specified in the yml file IE "portainer"

COMPLETE!!!

login to the webUI

URL: https://[hostIP]:9443

### Updating the instance

```
docker pull portainer/portainer-ce:[version]
```
```
docker service update --image portainer/portainer-ce:[version] --publish-add 9443:9443 --force portainer_poratiner
```
```
docker pull portainer/agent:[version]
```
```
docer service update --image portainer/agent:[version] --force portainer_agent
```

Login default:
user: admin
PW: password
