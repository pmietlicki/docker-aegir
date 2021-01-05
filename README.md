**A docker image for aegir hostmaster with apache / php version 7.3**

*Designed to run easily under Rancher, see [rancher website](https://rancher.com/)*

** WORKDIR and VOLUME under /var/aegir**. You could map this volume if you want to have a persistent volume thus keeping data between updates and rollbacks.

What is Aegir?
==============

Aegir is a free and open source hosting system for Drupal, CiviCRM, and Wordpress.

Designed for mass Drupal hosting, Aegir can launch new sites with a single form submission or API call (See [Hosting Services](http://drupal.org/project/hosting_services).

Aegir itself is built on Drupal and Drush, allowing you to tap into the large contributed module community.

For more information, visit aegirproject.org

How to use this image
=====================

This image is designed to create a new [aegir](https://www.aegirproject.org/) hostmaster server.

## Manual launch:

    $ docker run --name database -d -e MYSQL_ROOT_PASSWORD=12345 -e mariadb 
    $ docker run --name hostmaster --hostname aegir.local.computer -e MYSQL_ROOT_PASSWORD=12345 --link database:mysql -p 80:80 pmietlicki/aegir-web-server
    
## docker-compose launch:

  1. Create a docker-compose.yml file:

    ```yml
    version: '2'
    services:
    
      hostmaster:
        image: pmietlicki/aegir
        ports:
          - 80:80
        hostname: devaegir.local.computer
        links:
          - database
        depends_on:
          - database
        environment:
          MYSQL_ROOT_PASSWORD: strongpassword
          AEGIR_HOSTNAME: devaegir.local.computer
          
      webserver:
        image: pmietlicki/aegir-web-server
        ports:
          - 80:80
        hostname: devwebsrv.local.computer
      
      database:
        image: mariadb
        environment:
          MYSQL_ROOT_PASSWORD: strongpassword
    ```
  2. run `docker-compose up`.

# Environment Variables

## LANG 
*Default: fr_FR.UTF-8*

Default LANG for system, put en_US.UTF-8 for english.

## LANGUAGE
*Default: fr_FR:fr*

Default LANGUAGE for system, put en_US for english.

## AEGIR_CLIENT_NAME 

*Default: admin*

The username of UID1 and the client node.

## AEGIR_CLIENT_EMAIL 
*Default: aegir@aegir.local.computer*

The email for UID1. The welcome email for user 1 gets sent to this address.

## AEGIR_PROFILE 
*Default: hostmaster*

The install profile to run for the drupal front-end. Defaults to hostmaster.

## AEGIR_VERSION 
*Default: 7.x-3.192*

The aegir version to install.

## AEGIR_MAKEFILE
*Defalt: http://cgit.drupalcode.org/provision/plain/aegir-release.make?h=$AEGIR_VERSION*

The makefile to use for building the front-end drupal dashboard.  Defaults to hostmaster.

May use https://raw.githubusercontent.com/opendevshop/devshop/1.x/build-devmaster.make to build a devmaster instance.

## AEGIR_DATABASE_SERVER
*Default: database*

The MariaDB / MySQL database hostname to use.

## MYSQL_ROOT_PASSWORD
**Default: password**

The MariaDB / MySQL database root password, **it is mandatory**.

## AEGIR_HOSTNAME
*Default: devaegir*

The hostname of the server. **It is very important !**. Must be pingable and resolvable inside the container. You should set an etc/hosts entry with the final FQDN of the server, for example :
**127.0.0.1 devaegir.yourdomain.com**

## PROVISION_VERSION 
*Default: 7.x-3.190*

The provision version to install on top of drush.

## AEGIR_WORKING_COPY 
*Default: 0*

To create the hostmaster platform as working copies (Git checkouts for all modules) you can set the aegir/working-copy option to 1.

## AEGIR_HTTP_SERVICE_TYPE 
*Default: apache*

The web server aegir will be hosted by, you should let apache web server by default. The image has not be created for other type of web servers (nginx, etc.) for the moment.

## DRUSH_VERSION
*Default: 8.3.0*

The drush version to install inside the container.

## REGISTRY_REBUILD_VERSION
*Default: 7.x-2.5*

The aegir registry version to install on top of drush.

## AEGIR_UID
*Default: 1000*

UID of the aegir user, you can put 0 if you have sudo or rights problems.

## AEGIR_HOSTMASTER_ROOT
*Default: /var/aegir/hostmaster*

Must be fixed across versions so we can upgrade containers. Should not be changed.

## PATH
*Default: /var/aegir/.drush:${PATH}*

PATH added for /var/aegir (workdir and volume). Should not be changed except if you want to do specific things (install other tools for example).

# USAGE

# Aegir on Docker / Rancher

This project is aimed to get Aegir working *inside* Docker and/or Rancher.

An official Aegir docker image will make it really easy to fire up an aegir instance for production or just to try.

This image will also make contributing and testing much, much easier.

## Launching

### Requirements:

 - [Docker](https://docs.docker.com/engine/installation/) & [Docker Compose 2](https://docs.docker.com/compose/install/).
 - Optional : [Rancher](https://rancher.com/)

### Launching:

1. Clone this repo:

    git clone git@github.com:pmietlicki/aegir.git

2. Change directories into the codeabase.

    cd aegir

3. Edit your `/etc/hosts` file to add the container's hostname, which is set in `docker-compose.yml` (aegir.local.computer by default).  It must point at your docker host.
    127.0.0.1  aegir.local.computer  # If running native linux
    192.168.99.100  aegir.local.computer  # If running on OSx, or using default docker-machine

4. Build the image. (Optional)

  If you made your own changes to the dockerfile, you can build your own image:

    docker build -t pmietlicki/aegir -f Dockerfile .

4. Run docker compose up.

    docker-compose up

  If this is the first time, it will download the base images and hostmaster will install. If not, the database will be read from the volume as already having installed, so it will just run the drush commands to prepare the server.

  You will see the hostmaster install success output:

      hostmaster_1 | ==============================================================================
      hostmaster_1 |
      hostmaster_1 |
      hostmaster_1 | Congratulations, Aegir has now been installed.
      hostmaster_1 |
      hostmaster_1 | You should now log in to the Aegir frontend by opening the following link in your web browser:
      hostmaster_1 |
      hostmaster_1 | http://aegir.local.computer/user/reset/1/1468333477/iKwXpRJ7xhHeiPwhiE2oe5UcswlLeS_fZVALR9EvKZg/login
      hostmaster_1 |
      hostmaster_1 |
      hostmaster_1 | ==============================================================================

  Visit that link, but change the port if you had to change it in docker-compose.

  http://aegir.local.computer:12345/user/reset/1/abscdf....abcde/login

  That's it!

  You can access the container via terminal with docker exec:

        docker exec -ti aegirdocker_hostmaster_1 bash

  Since the user of the container is already set to `aegir`, you can just run "bash".

## Tech Notes

### Developing Dockerfiles

It can be confusing and monotonous to build the image, docker compose up, kill, remove, then rebuild, repeat...

 So I use the following command to ensure fully deleted containers and volumes, a rebuilt image, and a quick exit if things fail (--abort-on-container-exit)

    docker-compose kill ; docker-compose rm -vf ; docker build -t aegir/hostmaster ../ ; docker-compose up --abort-on-container-exit

You only need to run this full command if you change the Dockerfile or the docker-entrypoint-*.sh files.

If you are only changing the docker-compose.yml file, sometimes you can just run:

    docker-compose restart

Or you can kill then up the containers if you need the "CMD" or "command" to run again:

    docker-compose kill; docker-compose up

### Hostnames

The trickiest part of getting Aegir Hostmaster running in docker was the hostname situation. Aegir installs itself based on the servers hostname. This hostname is used for the server nodes and for the drupal front-end,

I've worked around that for now by using a docker-entrypoint.sh file that does the hostmaster installation at run time using AEGIR_HOSTNAME :
```
if [[ -z "${AEGIR_HOSTNAME}" ]]; then
  HOSTNAME=`hostname --fqdn`
else
  HOSTNAME="${AEGIR_HOSTNAME}"
fi
drush hostmaster-install -y --strict=0 $HOSTNAME \
```

### Services

A stock hostmaster requires mysql, apache and supervisor in order to have a responsive queue. The "docker way" is to separate services out.

I was able to use an external database container for MySQL.

Apache automatically gets started when Hostmaster is verified, so docker-entrypoint.sh doesn't have to do it.

Finally, instead of Supervisor, we run `drush @hostmaster hosting-queued`, so that is at the end of docker-entrypoint.sh.

**I really recommend to deploy it under Rancher or other type of docker/kubernetes orchestrator manager, it is much easier !**

### RANCHER

The parameters for a fully functional image under Rancher are :

#### Environment variables
- Set AEGIR_DATABASE_SERVER to the hostname (mariadb.dev for example) of your database 
- Set AEGIR_HOSTNAME to the hostname for your aegir server (devaegir for example)
- Set MYSQL_ROOT_PASSWORD to the root password of your MariaDB/MySQL DMBS
- Optionaly, set http_proxy, https_proxy and no_proxy variables

#### Health check
- Optionaly, set the liveness check part to "command run inside the containers exits with status 0" with :
/usr/bin/env php /usr/local/bin/drush @hostmaster hosting-dispatch
Checking after 300 seconds, interval 30 seconds, timeout 30 seconds, unhealthy after 10 attempts

#### Volumes
- You can set NFS volumes for /var/aegir/.ssh and for /var/aegir

#### Networking
- Set hostname as same as the AEGIR_HOSTNAME variable
- Add a host aliases entry (/etc/hosts) with :
127.0.0.1 FQDN_OF_AEGIR

#### Security and host config
In privilege escalation, set "Yes : container can gain more privileges than its parent process
Memory reservation : min 4096 
Cpu reservation : min 2000

  That's it!
