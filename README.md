# DivSeek Canada Portal

The [DivSeek Canada](http://www.divseekcanada.ca)  **Portal** is a web-based platform to implement association genetics 
workflows supporting plant breeding and crop research focusing on large scale plant genetic resources / crop 
genotype-phenotype data sets whose access is brokered / managed by the project.

# Genome Canada Pilot Project

The first iteration of the platform is funded under a 
[Genome Canada Project](https://www.genomecanada.ca/en/divseek-canada-harnessing-genomics-accelerate-crop-improvement-canada) with co-funding from other partners.

# Documentation

Some technical notes about the portal system will be compiled on the 
[Divseek Portal Wiki](https://github.com/DivSeek-Canada/divseek-canada-portal/wiki).

# Docker Deployment of the DivSeek Canada Portal

The DivSeek Canada Portal is being designed to run within a **Docker Compose** deployed system of **Docker** containers when the application is run on a Linux server or virtual machine. Thus, some system preparation to run Docker (and Compose) is required.

## Configuration for Cloud Deployment

When hosting Docker and the DivSeek Canada Portal in a cloud environment, such as the OpenStack cloud at Compute Canada, 
some special configuration is likely needed.

### Create the Cloud Instance

We start by creating a persistent _p4-6gb_ (4 core, 6 GB RAM)  flavour of compute instance. The security group should open up 
the TCP/IP ports exposed by the various docker instances, as specified in the project's docker-compose.yml file.

### Docker Image and Volume Storage

By default, the Docker image/volume cache (and other metadata) resides under **/var/lib/docker** which will end up being hosted
on the root volume of a cloud image, which may be relatively modest in size. To avoid "out of file storage" messages, 
which related to limits in inode and actual byte storage, it is advised that you remap (and copy the default contents
of) the **/var/lib/docker** directory onto an extra mounted storage volume (which should be configured to be 
automounted by _fstab_ configuration).

In effect, it is generally useful to host the entire portal and its associated docker storage volumes on such an extra mounted volume.
We generally use the **/opt** subdirectory as the target of the mount, then directly install various code and related subdirectories
there, including the physical target of a symbolic link to the **/var/lib/docker** subdirectory. You will generally wish to set 
this latter symbolic link first before installing Docker itself (here we assume that docker has _not_ yet been installed (let alone
running).

In Compute Canada, using the OpenStack dashboard, a cloud "Volume" can be created and attached to a running DivSeek Canada Portal
cloud server instance. We suggest creating a volume at least 200 GB in size (to allow for significant genomic data storage). After attaching the volume to the instance, the volume is initialized and mounted from within an 
SSH terminal session, as follows (where '$' is the Linux Bash CLI terminal prompt):

    # Before starting, make sure that the new volume (here, 'vdb') is visible (should be!)
    lsblk
    NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    vda     254:0    0  2.2G  0 disk
    ├─vda1  254:1    0  2.1G  0 part /
    ├─vda14 254:14   0    4M  0 part
    └─vda15 254:15   0  106M  0 part /boot/efi
    vdb     254:16   0  200G  0 disk

    # First, initialize the filing system on the new, empty, raw volume (assumed here to be on /dev/vdb)
    sudo mkfs -t ext4 /dev/vdb 
   
    # Mount the new volume in its place (we assume that the folder '/opt' already exists)
    sudo mount /dev/vdb /opt

    # Provide a symbolic link to the future home of the docker storage subdirectories
    sudo mkdir /opt/docker
    sudo chmod go-r /opt/docker
    
    # It is assumed that /var/lib/docker doesn't already exist. 
    # Otherwise, you'll need to delete it first, then create the symlink
    sudo ln -s /opt/docker /var/lib  
    
It is also recommended that a separate additional volume for each crop dataset deployed be created, following the above instructions.  A 500 GB volume is likely to be needed for this given the genomic large data sets involved. Also, you can attach volumes containing the raw input data (e.g. VCF files) as you need (Note: you should obviously not run the ```mkfs``` command afresh on any volumes which already have data!)  It is recommended first that you create a subdirectory ```/opt/divseekcanada``` then mount these additional volumes in that subdirectory, namely something like the following:

```
# Create a master folder for the DivSeek Canada code
sudo mkdir -p /opt/divseekcanada
# ensuring easy $USER access to these resources
sudo chown ubuntu:ubuntu /opt/divseekcanada
# Add the data volumes
sudo mkdir -p /opt/divseekcanada/data/downy-mildew
sudo mount /dev/vdc /opt/divseekcanada/data/downy-mildew
sudo mkdir -p /opt/divseekcanada/Sunflower
sudo mount /dev/vdd /opt/divseekcanada/Sunflower

# test the fstab mount with a 'fake' mounting
sudo mount -vf
```

After completing the above steps, you should configure ```/etc/fstab``` file for system boot up mounting of the new volumes:
    
    # These volumes need to be auto mounted upon each reboot of the system
    # so you should (carefully) add them to the Linux /etc/fstab file 
    # of the server, something like the following text entries (customize for your crop):
    /dev/vdb        /opt    ext4    rw,relatime     0       0
    /dev/vdc        /opt/divseekcanada/data/downy-mildew    ext4    rw,relatime     0       0
    /dev/vdd        /opt/divseekcanada/Sunflower    ext4    rw,relatime     0       0

Now, you can proceed to install Docker and Docker Compose.

## Installation of Docker

To run Docker, you'll obviously need to [install Docker first](https://docs.docker.com/engine/installation/) in your target Linux operating environment (bare metal server or virtual machine running Linux).

For our installations, we typically use Ubuntu Linux, for which there is an [Ubuntu-specific docker installation using the repository](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-using-the-repository).
Note that you should have 'curl' installed first before installing Docker:

```
sudo apt-get install curl
```

For other installations, please find instructions specific to your choice of Linux variant, on the Docker site.

## Testing Docker

In order to ensure that Docker is working correctly, run the following command:

```
sudo docker run hello-world
```

This should result in something akin to the following output:

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
ca4f61b1923c: Pull complete
Digest: sha256:be0cd392e45be79ffeffa6b05338b98ebb16c87b255f48e297ec7f98e123905c
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

## Installing Docker Compose

You will then also need to [install Docker Compose](https://docs.docker.com/compose/install/) alongside Docker on your target Linux operating environment.

### Docker under Linux

Note that under Ubuntu, you likely need to do a bit more preparation to avoid having to run docker (and docker-compose) 
as 'sudo'. See [here](https://docs.docker.com/install/linux/linux-postinstall/) for details on how to fix this.

## Testing Docker Compose

In order to ensure Docker Compose is working correctly, issue the following command:
```
docker-compose --version
docker-compose version 1.22.0, build f46880f
```
Note that your particular version and build number may be different than what is shown here. We don't currently expect that docker-compose version differences should have a significant impact on the build, but if in doubt, refer to the release notes of the docker-compose site for advice.

# ElasticSearch

During the creation of the ElasticSearch indexing container in the Docker Tripal system, one may run up against another
resource limit, reported by the following error message:

    max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

This solution to this is to add this line:

    vm.max_map_count=262144

in your **/etc/sysctl.conf** file on the host system and run

    sudo sysctl -p

to reload configuration with new value.

# Installing the DivSeek Canada Portal codebase

This project resides in [this Github project repository](https://github.com/DivSeek-Canada/divseek-canada-portal).

First, ensure that you have the git client installed (here again, we assume Ubuntu; '$' is the bash CLI prompt):

    sudo apt update
    sudo apt install git

Next, you should configure git with your Git repository metadata and, perhaps, activate credential management (we use 'cache' mode here to avoid storing credentials in plain text on disk)

    git config --global user.name "your-git-account"
    git config --global user.email "your-email"
    git config --global credential.helper cache

Then, you can clone the project. A convenient location for the code is in a folder under **/opt/divseekcanada**:

    cd /opt/divseekcanada
    git clone https://github.com/DivSeek-Canada/divseek-canada-portal 

# Deployment of the Portal System

As of March 2019, the DivSeek Canada Portal customizes a git fork of the 
[Galaxy Genome Annotation "Dockerized GMOD" code base](https://github.com/galaxy-genome-annotation/dockerized-gmod-deployment). 

The core of the customization is in the Docker Compose build file (**docker-compose.yml**) on the **divseek-canada-build** branch.  This file is customized for (crop) site specific needs using environment variables defined in a **.env** file. That is, to customize a given crop-specific site, need to copy the **template.env** into **.env** then customize the contents to point to your actual portal hostname.

The NGINX proxy is also configured during the docker comopose build using a **default.conf** file in the **nginx**
subdirectory. The GMOD deployment default is to show 'galaxy' on the hostname resolution but there is an alternate 
template for a  'galaxy-tripal' swapped proxy. One or the other template (under the **nginx** subdirectory)
should be copied into **nginx/default.conf**.

The general project launch steps noted in the  [GMOD stack README](./docker-gmod/README.md) are 
otherwise followed with the revised YML file specified:

```console
docker-compose pull # Pull all images
docker-compose up -d apollo_db chado # Launch the DBs
```

In a new terminal, in the same folder, run `docker-compose logs -f` in order to
watch what is going on.

```
docker-compose up -d --build tripal # Wait for tripal to come up and install Chado.
# It takes a few minutes. I believe you'll see an apache error when ready.
docker-compose up -d --build # This will bring up the rest of the services.
```

# Deeper Details about the GMOD Deployment (copied over from the original GGA repository)

This docker-compose.yml file specifies all of the infrastructure needed to run
the current iteration of GMOD products.

We have attempted to include as many "bells and whistles" as possible, which
means this approaches the level of "tech-demo" and away from something you
might wish to deploy as-is. That said, the deployment is very easy to customise
and adapt to your particular organisation's needs.

![](./media/network.png)

## Screenshots

**Galaxy** ...

![](./media/galaxy.png)

... allows you to load data in **Apollo**,

![](./media/apollo.png)

which in turn can be exported to **Chado** and **Tripal**.

![](./media/tripal.png)

But wait, there's more! Data in Chado is made available in **JBrowse**

![](./media/jbrowse.png)


## Running

```console
$ docker-compose pull # Pull all images
$ docker-compose up -d apollo_db tripal_db # Launch the DBs
```

In a new terminal, in the same folder, run `docker-compose logs -f` in order to
watch what is going on.

```
$ docker-compose up -d tripal # Wait for tripal to come up and install Chado.
$ # It takes a few minutes. I believe you'll see an apache error when ready.
$ docker-compose up -d # This will bring up the rest of the services.
```

## Services:

Service                          | Address
-------------------------------- | ----
Galaxy                           | :8200
Tripal (/tripal)                 | :8200/tripal
Apollo (through Galaxy, /apollo) | :8200/apollo
Chado JBrowse Connector          | :8200/jbrowse/
PostgREST                        | :8200/postgrest
PostGraphQL Graph*i*QL interface | :8200/graphiql
PgAdmin4                         | :8201/
Chado PostGraphQL Connector      | :8200/jbrowse_graphql/

## Credentials

Service | Username         | Password
------- | ---------------- | ---------
Galaxy  | admin@galaxy.org | admin
Tripal  | admin            | changeme
Apollo  | admin@local.host | password

The Apollo account is only used internally by the Galaxy tools. You should be automatically logged in with your Galaxy account when connecting to Apollo (thanks to [gx-cookie-proxy](https://github.com/erasche/gx-cookie-proxy)).

## Dependencies

This `docker-compose.yml` depends on a large number of containers. We link some
build status images here for developer convenience

Image               | Status
-----               | ------
galaxy              | ![](https://quay.io/repository/galaxy-genome-annotation/docker-galaxy-annotation/status)
postgrest           | ![](https://quay.io/repository/erasche/postgrest/status)
postgraphql         | [![Docker Automated build](https://img.shields.io/docker/automated/erasche/postgraphql.svg?style=flat-square)](https://hub.docker.com/erasche/postgraphql)
chado-jb-connector  | ![](https://quay.io/repository/erasche/chado-jbrowse-connector/status)
chado               | ![](https://quay.io/repository/galaxy-genome-annotation/chado/status)
tripal              | ![](https://quay.io/repository/galaxy-genome-annotation/tripal/status)
gx-cookie-proxy     | ![](https://quay.io/repository/erasche/gx-cookie-proxy/status)

## Configuring

Each Docker image can be configured to fit your needs. You should consult the documentation of each image, in particular:

[Galaxy Docker image documentation](https://github.com/bgruening/docker-galaxy-stable)

[Tripal Docker image documentation](https://github.com/galaxy-genome-annotation/docker-tripal)

[Chado Docker image documentation](https://github.com/galaxy-genome-annotation/docker-chado)

[Apollo Docker image documentation](https://github.com/galaxy-genome-annotation/docker-apollo)

## LICENSE

GPLv3

## Support

This material is based upon work supported by the National Science Foundation under Grant Number (Award 1565146)
