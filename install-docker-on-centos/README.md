Docker on CentOs 7
===================

source: https://docs.docker.com/install/linux/docker-ce/centos/

###prepare yum
```
yum update ;\
yum upgrade ;\
yum install -y yum-utils device-mapper-persistent-data lvm2 ;\
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

###install docker from yum
```
yum update ;\
yum install -y docker-ce
```

###start docker and check if docker is running
```
systemctl start docker 
docker --version
docker info
```

###start docker on system start
```
systemctl enable docker
```

###prepare for docker stacks (replacing docker-compose in most parts)
```
docker swarm init
```

