# Docker image for Headwind MDM

Headwind MDM is an open source mobile device management software for Android 
devices. It has been originally designed for Ubuntu Linux. This image helps
to run Headwind MDM on any Linux.

Headwind MDM project URL: https://h-mdm.com

## Summary

The image is based on Ubuntu 20.04 and Tomcat 9.

It doesn't include PostgreSQL and certbot, so they need to be started in
separate containers or on the host machine.

As an alternative, you can use docker-compose to run Headwind MDM and all 
required packages (certbot, PostgreSQL) on a fresh virtual machine with the 
most common options (see below).

## Building the image from the source code

Before building the image, review the default variables (in particular the 
Headwind MDM URL) in the Dockerfile and change them if required.

The build command is:

    docker build -t headwindmdm/hmdm:0.1.2 .

## Prerequisites

1. Create the PostgreSQL database for Headwind MDM, and use the environment
variables SQL_HOST, SQL_BASE, SQL_USER, SQL_PASS to define the database access
credentials.

Default values are: SQL_HOST=localhost, SQL_BASE=hmdm, SQL_USER=hmdm,
SQL_PASS=topsecret

2. If you want to use HTTPS, install certbot and generate the certificate for
the domain where Headwind MDM should be installed.

    certbot certonly --standalone --force-renewal -d your-mdm-domain.com 

## Running the Docker container

**Works with the external PostgreSQL database only. Default database installation on localhost DOES NOT WORK!**

**Please set up your domain name when running Headwind MDM!**

To create the container, use the command:

    docker run -d -p 443:8443 -p 31000:31000 -e SQL_HOST=database.host -e SQL_BASE=hmdm -e SQL_USER=hmdm -e SQL_PASS=password -e BASE_DOMAIN=build.h-mdm.com -v /etc/letsencrypt:/etc/letsencrypt -v $(pwd)/volumes/work:/usr/local/tomcat/work --name="hmdm" headwindmdm/hmdm:0.1.2

If everything is fine, Headwind MDM will become available via the url 
`https://your-mdm-domain.com` in a few seconds. 

To view logs, use the command:

    docker logs hmdm

Stop and start the container:

    docker stop hmdm
    docker start hmdm

Connect to the container for debugging:

    docker exec -it hmdm /bin/bash

## Configuration of Headwind MDM

The container is configured by the environment variables.

The full list of variables can be found in the Dockerfile.

## First start and subsequent starts

At first start, Headwind MDM performs the initialization:

  - Creates the config files using the environment
  - Initializes the database
  - Converts the LetsEncrypt's (or your own) SSL certificates to a JKS keystore

Subsequent starts of the container skip this step, but you can force the
configuration renewal by setting the following environment variable:

FORCE_RECONFIGURE=true

When this variable is set to true, the configuration is always re-created by the
Headwind MDM entry point script. 

## Running with the most common options by Docker Compose

For a simple start of Headwind MDM on a fresh virtual machine, run the 
following commands.

    apt install -y docker-compose
    cd hmdm-docker
    docker build -t headwindmdm/hmdm:0.1.2 .
    cp .env.example .env
    vim .env              # Replace ADMIN_EMAIL and BASE_DOMAIN to your values
    docker-compose up

The command `docker-compose up` will start Headwind MDM in the interactive 
mode where you can easily trace and fix errors.

Once Headwind MDM start is successful, you can start it in the background
(detached) mode by using the command:

    docker-compose up -d

To view logs, use the command:

    docker-compose logs hmdm -f

To stop (but not remove) the service, use the command:

    docker-compose stop


