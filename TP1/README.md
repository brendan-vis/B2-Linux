# I. Init

## 3. sudo c pa bo

🌞 Ajouter votre utilisateur au groupe docker

```
[brendan@docker ~]$ sudo usermod -aG docker $USER

[brendan@docker ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

## 4. Un premier conteneur en vif

🌞 Lancer un conteneur NGINX

```
docker run -d -p 9999:80 nginx
```

🌞 Visitons

```
[brendan@docker ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS              PORTS                                     NAMES
a3e803d8d6f2   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:9999->80/tcp, [::]:9999->80/tcp   elegant_swartz


[brendan@docker ~]$ docker logs a3
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2024/12/11 09:49:27 [notice] 1#1: using the "epoll" event method
2024/12/11 09:49:27 [notice] 1#1: nginx/1.27.3
2024/12/11 09:49:27 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2024/12/11 09:49:27 [notice] 1#1: OS: Linux 5.14.0-284.11.1.el9_2.x86_64
2024/12/11 09:49:27 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1073741816:1073741816
2024/12/11 09:49:27 [notice] 1#1: start worker processes
2024/12/11 09:49:27 [notice] 1#1: start worker process 30


[brendan@docker ~]$ docker inspect --size a3
[
    {
        "Id": "a3e803d8d6f2860f250cc7ad2290b4461c87fa6fa1e850ec445567d4d608df99",
        "Created": "2024-12-11T09:49:25.501216408Z",
        "Path": "/docker-entrypoint.sh",
        "Args": [
            "nginx",
            "-g",
            "daemon off;"
        ],

#Trop long j'ai mis que le début


[brendan@docker ~]$ sudo ss -lnpt
[sudo] password for brendan:
State        Recv-Q       Send-Q             Local Address:Port             Peer Address:Port      Process
LISTEN       0            4096                     0.0.0.0:9999                  0.0.0.0:*          users:(("docker-proxy",pid=3462,fd=4))
LISTEN       0            128                      0.0.0.0:22                    0.0.0.0:*          users:(("sshd",pid=682,fd=3))
LISTEN       0            4096                        [::]:9999                     [::]:*          users:(("docker-proxy",pid=3467,fd=4))
LISTEN       0            128                         [::]:22                       [::]:*          users:(("sshd",pid=682,fd=4))


[brendan@docker ~]$ sudo firewall-cmd --add-port=9999/tcp --permanent
success
[brendan@docker ~]$ sudo firewall-cmd --reload


http://10.1.1.11:9999/
```

🌞 On va ajouter un site Web au conteneur NGINX

```
[brendan@docker nginx]$ docker rm -f  elegant_swartz

[brendan@docker nginx]$ docker run -d -p 9999:8080 -v /home/brendan/nginx/index.html:/var/www/html/index.html -v /home/brendan/nginx/site_nul.conf:/etc/nginx/conf.d/site_nul.conf nginx

[brendan@docker nginx]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED              STATUS
PORTS                                                 NAMES
92f99653b64d   nginx     "/docker-entrypoint.…"   About a minute ago   Up About a minute   80/tcp, 0.0.0.0:9999->8080/tcp, [::]:9999->8080/tcp   beautiful_goldwasser

http://10.1.1.11:9999/
# il y a écrit Meow
```

# 5. Un deuxième conteneur en vif

🌞 Lance un conteneur Python, avec un shell

```
docker run -it python bash
```

🌞 Installe des libs Python

```
pip install aiohttp
pip install aioconsole

root@7c05a58a147d:/# import aiohttp
import-im6.q16: unable to open X server `' @ error/import.c/ImportImageCommand/346.
```

# II. Images

## 1. Images publiques

🌞 Récupérez des images

```
[brendan@docker nginx]$ docker pull python:3.11
[brendan@docker nginx]$ docker pull mysql:5.7[brendan@docker nginx]$ docker pull wordpress
[brendan@docker nginx]$ docker pull linuxserver/wikijs

[brendan@docker nginx]$ docker image ls
REPOSITORY           TAG       IMAGE ID       CREATED         SIZE
linuxserver/wikijs   latest    863e49d2e56c   4 days ago      465MB
python               3.11      342f2c43d207   7 days ago      1.01GB
nginx                latest    66f8bdd3810c   2 weeks ago     192MB
wordpress            latest    c89b40a25cd1   2 weeks ago     700MB
mysql                5.7       5107333e08a8   12 months ago   501MB
```


🌞 Lancez un conteneur à partir de l'image Python

```
[brendan@docker nginx]$ docker run -it python:3.11 bash

[brendan@docker nginx]$ docker run -it python:3.11 bash
root@bb3784b03fbb:/# python
Python 3.11.11 (main, Dec  4 2024, 20:38:25) [GCC 12.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
```


# 2. Construire une image

🌞 Ecrire un Dockerfile pour une image qui héberge une application Python

```
[brendan@docker python_app_build]$ ls
app.py  Dockerfile
[brendan@docker python_app_build]$ cat Dockerfile
FROM debian

RUN apt update
RUN apt -y install python3
RUN apt-get -y install python3-pip
RUN python3 -m pip config set global.break-system-packages true
RUN pip install emoji
COPY app.py /app.py
ENTRYPOINT ["python3", "app.py"]
```

🌞 Build l'image

```
[brendan@docker python_app_build]$ docker build . -t python_app:version_de_ouf
```

🌞 Lancer l'image

```
[brendan@docker python_app_build]$ docker run python_app:version_de_ouf
Cet exemple d'application est vraiment naze 
```


# III. Docker compose

🌞 Créez un fichier docker-compose.yml

```
[brendan@docker ~]$ cat compose_test/docker-compose.yml
version: "3"

services:
  conteneur_nul:
    image: debian
    entrypoint: sleep 9999
  conteneur_flopesque:
    image: debian
    entrypoint: sleep 9999
```

🌞 Vérifier que les deux conteneurs tournent

```
[brendan@docker compose_test]$ docker compose ps
WARN[0000] /home/brendan/compose_test/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
NAME                                 IMAGE     COMMAND        SERVICE               CREATED              STATUS              PORTS
compose_test-conteneur_flopesque-1   debian    "sleep 9999"   conteneur_flopesque   About a minute ago   Up About a minute
compose_test-conteneur_nul-1         debian    "sleep 9999"   conteneur_nul         About a minute ago   Up About a minute


[brendan@docker compose_test]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED             STATUS             PORTS                                                 NAMES
eae6a18e1722   debian    "sleep 9999"             4 minutes ago       Up 4 minutes                                                             compose_test-conteneur_nul-1
84ca4d8ea401   debian    "sleep 9999"             4 minutes ago       Up 4 minutes                                                             compose_test-conteneur_flopesque-1
```


🌞 Pop un shell dans le conteneur conteneur_nul

```
[brendan@docker compose_test]$ docker exec -it eae6a18e1722 bash

root@eae6a18e1722:/# apt update
root@eae6a18e1722:/# apt install -y iputils-ping


root@eae6a18e1722:/# ping conteneur_flopesque
PING conteneur_flopesque (172.18.0.2) 56(84) bytes of data.
64 bytes from compose_test-conteneur_flopesque-1.compose_test_default (172.18.0.2): icmp_seq=1 ttl=64 time=1.09 ms
64 bytes from compose_test-conteneur_flopesque-1.compose_test_default (172.18.0.2): icmp_seq=2 ttl=64 time=0.623 ms
64 bytes from compose_test-conteneur_flopesque-1.compose_test_default (172.18.0.2): icmp_seq=3 ttl=64 time=0.163 ms
^C
--- conteneur_flopesque ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2005ms
rtt min/avg/max/mdev = 0.163/0.626/1.094/0.380 ms
```