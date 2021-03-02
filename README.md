![Logo](/img/logo.png)

# Introduction

Welcome to the DTech ServiceNow Docker Project.

This project creates the Docker containers required to run ServiceNow and gets them ready for the Docker Registry. It also contains deployment steps and prerequisites.

# Prerequisites
* DIA Requirements
  * AWS AMI spel-minimal-centos-7-hvm-2019.04.1.x86_64-gp2
    * Expand root device with additional storage capacity
* [Docker CE](https://docs.docker.com/engine/installation/#supported-platforms)
* [Docker Compose](https://github.com/docker/compose/releases)

# Deploy DARTS-DEV Environment

Install Docker (CentOS 7):
```
curl -fsSL https://get.docker.com/ | sh
sudo systemctl enable docker
sudo systemctl start docker
```

Install Docker-Compose (CentOS 7):
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose
sudo chmod +x /usr/bin/docker-compose
docker-compose --version
```

Install Git (CentOS 7):
```
yum install -y git
```
Pull Code:
```
cd /opt
git clone https://discovertech@dev.azure.com/discovertech/DGit/_git/servicenow-docker
```
_**Git Password:** 36hqyvb7nogvksxpwisgb2sydgjmdtni24ua74x345bvjjuluaja_

Deploy:
```
cd /opt/servicenow-docker/dtech-snow/docker-compose
docker login dtech-on.azurecr.io -u DTech -p /L/p=+YN+=MmE/=+=CEqfTZDeCbKAIh+
export POD_NAME=snow POD_PORT=16001 BUILD_ID=${BUILD_ID}; docker-compose -p $POD_NAME up -d
```
After sending this command it will take some time to download the images, start the containers, and deploy the SQL database. Wait at least one(1) hour.

# Build and Push Docker Images

* Login to the Docker Host (SSH)
* Clone the the SERVICENOW-DOCKER project

```
mkdir /snowdocker
cd /snowdocker
git clone https://discovertech@dev.azure.com/discovertech/DGit/_git/servicenow-docker
```
_**Git Password:** 36hqyvb7nogvksxpwisgb2sydgjmdtni24ua74x345bvjjuluaja_

* Build and tag the MySQL Container.

```
cd servicenow-docker/dtech-snow/docker-mysql/5.7/
docker build -t dtech-on.azurecr.io/servicenow/docker-mysql:<<BUILD ID>> --build-arg BUILD_ID=<<BUILD ID>> .

docker build -t dtech-on.azurecr.io/servicenow/docker-mysql:<<BUILD ID>> --build-arg account=<<AZURE STORAGE ACCOUNT>> --build-arg share=<<STORAGE SHARENAME>> --build-arg key=<<STORAGE ACCESS KEY>> --b

Working Example:
docker build -t dtech-on.azurecr.io/servicenow/docker-mysql:0.0.04 --build-arg account="dtechservicenow" --build-arg share="dtechsnow" --build-arg key="sajmM5mezZ3c2c36BKoS51uLCapTsX/aRfMbrIWzYNf3HxJDhc7B3Vz5V48U2hJtolmRS+4I8MMlmB9Nrhb/RQ==" --build-arg path="orlando/123/sn_16001-DTECH.sql.gz" .

docker tag dtech-on.azurecr.io/servicenow/docker-mysql:<<BUILD ID>> dtech-on.azurecr.io/servicenow/docker-mysql:latest
```

* Push the MySQL Container to the Private Registry

```
docker push dtech-on.azurecr.io/servicenow/docker-mysql:<<BUILD ID>>
docker push dtech-on.azurecr.io/servicenow/docker-mysql:latest
```

_DTech Docker Image Registry: Username: DTech Password: /L/p=+YN+=MmE/=+=CEqfTZDeCbKAIh+_

* Build and tag the ServiceNow Container

```
cd servicenow-docker/dtech-snow/docker-snow/
<<<<<<< HEAD
docker build -t dtech-on.azurecr.io/servicenow/docker-snow:<<DOCKER_IMAGE_BUILD>> --build-arg host=<<MYSQL IMAGE>> --build-arg username=<<MYSQL USERNAME>> --build-arg pass=<<MYSQL PASSWORD>> --build-arg account=<<AZURE_STORAGE_ACCOUNT_NAME>> --build-arg key=<<AZURE_STORAGE_ACCOUNT_KEY>> --build-arg share=<<SHARE_NAME>> --build-arg snowfile=<<SERVICENOW FILE>> .

Working Example:
docker build -t dtech-on.azurecr.io/servicenow/docker-snow:0.0.01 --build-arg host=docker-mysql --build-arg username=DTech --build-arg pass=DTPass.01 --build-arg account=dtechservicenow --build-arg key=sajmM5mezZ3c2c36BKoS51uLCapTsX/aRfMbrIWzYNf3HxJDhc7B3Vz5V48U2hJtolmRS+4I8MMlmB9Nrhb/RQ== --build-arg share=dtechsnow --build-arg snowfile=glide-orlando-12-11-2019__patch2-03-18-2020_03-31-2020_1417.zip .
=======
docker build -t dtech-on.azurecr.io/servicenow/docker-snow:<<DOCKER_IMAGE_BUILD>> --build-arg host=<<MYSQL IMAGE>> --build-arg username=<<MYSQL USERNAME>> --build-arg pass=<<MYSQL PASSWORD>> --build-ar

Working Example:
docker build -t dtech-on.azurecr.io/servicenow/docker-snow:0.0.01 --build-arg host=docker-mysql --build-arg username=DTech --build-arg pass=DTPass.01 --build-arg account=dtechservicenow --build-arg key
>>>>>>> eb34132c56cdf7a89e6bfdcbb2ad41f5e6234e38

docker tag dtech-on.azurecr.io/servicenow/docker-snow:<<BUILD ID>> dtech-on.azurecr.io/servicenow/docker-snow:latest
```

* Push the ServiceNow Container to the Private Registry

```
docker push dtech-on.azurecr.io/servicenow/docker-snow:<<BUILD ID>>
docker push dtech-on.azurecr.io/servicenow/docker-snow:latest
```

_DTech Docker Image Registry: Username: DTech Password: /L/p=+YN+=MmE/=+=CEqfTZDeCbKAIh+_

# References

* [Docker Compose Command Line Reference](https://docs.docker.com/compose)
* [Extend Storage in Linux LVM](https://www.tecmint.com/extend-and-reduce-lvms-in-linux/)

