#build from specific file name (other that Dockerfile)
docker build -f /c/Users/x/Desktop/git/Dockerfile-uaa /c/Users/x/Desktop/git/y
docker tag 5e2f0a6bb6ed centos-x

[create Dockerfile]

echo "FROM alpine:3.5" >> DockerFile
echo CMD sh -c "echo 'Started...'; while true ; do sleep 10 ; done" >> Dockerfile

docker login
docker build -t looper
docker tag looper docmseco/looper
docker push docmseco/looper

use in image as: docmseco/looper

[install docker compose]
sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

[debug]
https://opencredo.com/debugging-java-applications-running-in-docker/

random generator problem:
-v /dev/urandom:/dev/random

[windows]

echo "dock-vs = Docker box status"
alias dock-vs="cd 'C:\Program Files\Docker Toolbox' && docker-machine.exe env box && docker-machine.exe stop box && docker-machine.exe start box"

echo "dock = Docker terminal"
alias dock="cd 'C:\Program Files\Docker Toolbox' && docker-machine.exe env box && docker-machine.exe ssh box"


docker-machine.exe create box --virtualbox-disk-size "40000" --virtualbox-cpu-count "4" --virtualbox-memory "8096"

grep -c ^processor /proc/cpuinfo     
awk '/MemTotal/ {print $2}' /proc/meminfo


C:\Users\~\.docker\machine\machines\default

conf.json

 "insecure-registries" : ["x.om:443"]


start.sh
winpty docker login  https://registry.

cert prob 
https://stackoverflow.com/questions/31205438/docker-on-windows-boot2docker-certificate-signed-by-unknown-authority-error

MINGW64:$ docker-machine ssh default

docker@default:~$ sudo -s
root@default:/home/docker# mkdir /var/lib/boot2docker/certs
root@default:/home/docker# cp /c/Users/my.username/certs/*.pem /var/lib/boot2docker/certs

touch /var/lib/boot2docker/bootlocal.sh && chmod +x /var/lib/boot2docker/bootlocal.sh
vi /var/lib/boot2docker/bootlocal.sh

#!/bin/sh

mkdir -p /etc/docker/certs.d && cp certs/certificate.pem /etc/docker/certs.d

docker-machine restart default

#need image
docker search ubuntu
docker pull ubuntu-blabla

#customize it
docker run -i -t 546dd3a716b2 /bin/bash

# -p 8080:8080 
# -v /c/Users/uaa/:/uaa-config 

CACHED VOLUMES:
-v /c/Users/uaa/:cached:/uaa-config

type exit to exit and save

#view last runs
docker ps -a

export CLOUD_FOUNDRY_CONFIG_PATH=

docker start (your container this time)


docker commit bd0fbf61d9f9 centos-uaa( as new image)


#clean up
# remove exited containers:
docker ps --filter status=dead --filter status=exited -aq | xargs -r docker rm -v
    
# remove unused images:
docker images --no-trunc | grep '<none>' | awk '{ print $3 }' | xargs -r docker rmi

# remove unused volumes:
find '/var/lib/docker/volumes/' -mindepth 1 -maxdepth 1 -type d | grep -vFf <(
  docker ps -aq | xargs docker inspect | jq -r '.[] | .Mounts | .[] | .Name | select(.)'
) | xargs -r rm -fr



docker volume ls -qf dangling=true | xargs -r docker volume rm

docker attach

#to get a new shell on a running container
docker exec -i -t 665b4a1e17b6 /bin/bash