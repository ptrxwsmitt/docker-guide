Install Ghost 2 Blog using Docker on CentOs 7
=============================================

### prepare docker folders for volumes and stack yml files
I tend to put my volumes separate from the stack configuration yml:
```
mkdir -p /opt/docker/stacks/
mkdir -p /opt/docker/volumes/
```

Install nginx as reverse proxy
------------------------------

Thank you Jason Wilder for providing a nginx based reverse proxy on github:
https://github.com/jwilder/nginx-proxy

Further reading and source:
https://blog.agchapman.com/ghost-blog-with-nginx-and-docker-1/.

### prepare folders for volumes and stack yml files
```
mkdir -p /opt/docker/stacks/
mkdir -p /opt/docker/volumes/
```

#### prepare volume folders for nginx
```
mkdir -p /opt/docker/volumes/nginx-proxy/vhost.d ;\
mkdir -p /opt/docker/volumes/nginx-proxy/html ;\
mkdir -p /opt/docker/volumes/nginx-proxy/certs
```

sudo rm -r /opt/docker/volumes/nginx-proxy/certs

#### request certificates
Make sure to replace email and domain names with the <>-brackets.
```
docker run -t -i -p 80:80 -p 443:443 -v /opt/docker/volumes/certbot/etc:/etc/letsencrypt -v /opt/docker/volumes/certbot/var:/var/lib/letsencrypt certbot/certbot certonly --standalone --email <your.email@email.com> -d <domain.com> -d <blog.domain.com>
```

#### copy certificates to gitlab volumes
Make sure to replace domain names with the <>-brackets.
Nginx requires certificate and key to be in the PEM format (as returned from letsencrypt).
```
sudo cp /opt/docker/volumes/certbot/etc/live/<domain.com>/fullchain.pem /opt/docker/volumes/nginx-proxy/certs/<domain.com>.crt ;\
sudo cp /opt/docker/volumes/certbot/etc/live/<domain.com>/privkey.pem /opt/docker/volumes/nginx-proxy/certs/<domain.com>.key
```

### create yml file content
See the file "nginx-proxy.yml" in this folder. It contains further comments.
```
vi /opt/docker/stacks/nginx-proxy.yml
```


### deploy proxy and check logs for startup difficulties
```
docker stack deploy -c /opt/docker/stacks/nginx-proxy.yml proxy
docker service logs proxy_proxy
```

### output configuration
```
docker exec -it `docker container ls | grep proxy_proxy | cut -c-12` cat /etc/nginx/conf.d/default.conf
```

install ghost with mysql
------------------------

### prepare ghost volume folders
```
mkdir -p /opt/docker/volumes/ghost/var/lib/ghost/content
mkdir -p /opt/docker/volumes/mysql/var/lib/mysql
```

### create file yml file content
See the file "ghost.yml" in this folder. It contains further comments.
```
vi /opt/docker/stacks/ghost.yml
```

### deploy container
```
docker stack deploy -c /opt/docker/stacks/ghost.yml ghost
```

### show logs and wait till container is up (usually few seconds)
```
docker service logs ghost_ghost
docker container ls
```

### show logs as readable formatted json
The lines in the log are valid json objects, but the complete log file is not valid json.
You may need to split the file by lines before piping it through the python formatter.
```
cat /opt/docker/volumes/ghost/var/lib/ghost/content/logs/https___*_production.log | python -m json.tool | less -S
```

### login as su if you want to check sth inside the container
```
docker exec -it `docker container ls | grep ghost_ghost | cut -c-12` su
docker exec -it `docker container ls | grep ghost_mysql | cut -c-12` su
```
