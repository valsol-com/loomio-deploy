# loomio-deploy

To setup your own instance of Loomio you'll need a domain name and access to modify records for that domain.
You'll also need a server to run Loomio on. This guide assumes you have root access to a newly installed Ubuntu Linux x64 server.

## Server and domain name records
What hostname will you be using for your Loomio instance? What is the IP address of your server?

I'm going to use loomio.example.com and 123.123.123.123 as my IP address.

Just 2 domain name records are needed to run Loomio.
The first one is an A record, so that when people enter your domain name in their browser it resolves to your server's IP address.
The second is an MX so that when people reply by email to discussions, the email is sent to your server.

```
A loomio.example.com 123.123.123.123
MX loomio.example.com, loomio.example.com, priority 0
```

## Login as root
To login to the server, open a terminal window and type:

```sh
ssh root@loomio.example.com
```

## install docker and docker-compose

```sh
wget -qO- https://get.docker.com/ | sh
wget -O /usr/local/bin/docker-compose https://github.com/docker/compose/releases/download/1.6.2/docker-compose-`uname -s`-`uname -m`
chmod +x /usr/local/bin/docker-compose
```

## Clone the loomio-deploy git repository

```sh
git clone https://github.com/loomio/loomio-deploy.git
cd loomio-deploy
```

## Setup a swapfile (optional)
If you have less than 2GB RAM on your server then this step is required. This script will create a 4GB swapfile on your host.

```sh
./scripts/create_swapfile
```

## Create your config file:
This step creates an `env` file configured for your hostname. It also creates directories on the host to hold user data.

```sh
./scripts/create_env loomio.example.com
```

## Setup SMTP

```sh
nano env
```

Loomio is broken if it cannot send email. In this step you need to edit your `env` file and configure the SMTP settings to get outbound email working.

So you'll need an SMTP server. If you already have one, that's great, you know what to do. For everyone else here are some options to consider:

- For setups that will send less than 99 emails a day [use smtp.google.com](https://www.digitalocean.com/community/tutorials/how-to-use-google-s-smtp-server) for free.

- Look at the (sometimes free) services offered by [SendGrid](https://sendgrid.com/), [SparkPost](https://www.sparkpost.com/), [Mailgun](http://www.mailgun.com/), [Mailjet](https://www.mailjet.com/pricing).

- Very shortly we'll publish a guide to setting up your own private and secure SMTP server.

## Issue an SSL certificate for your hostname:
It's easy to obtain an SSL certificate and encrypt all the traffic in and out of your Loomio instance. Just paste this command into your terminal and follow the onscreen instructions.

```sh
docker run -it --rm -p 443:443 -p 80:80 --name letsencrypt \
            -v "/root/loomio-deploy/certificates/:/etc/letsencrypt" \
            -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
            quay.io/letsencrypt/letsencrypt:latest auth
```

## Create the database
This command initializes a new database for your Loomio instance to use.

```
docker-compose run web rake db:setup
```

## start the system
This command starts the database, application, reply-by-email, and live-update services all at once.

```
docker-compose up -d
```

## install crontab
Tell the host what regular tasks it needs to run to keep loomio functioning properly.

```
cat crontab >> /etc/crontab
```

## confirm it works
visit your hostname in your browser and hopefully you'll see a login screen.

todo:
* confirm mailin, pubsub work
* force ssl

Other need to know docker commands include:
* `docker ps` lists running containers.
* `docker ps -a` lists all containers.
* `docker logs <container>` to find out what a container is doing.
* `docker stop <container_id or name>` will stop a container
* `docker start <container_id or name>` will start a container
* `docker restart <container_id or name>` will restart a container
* `docker rm <container_id or name>` will delete a container
* `docker pull loomio/loomio:latest` pulls the latest version of loomio
* `docker help <command>` will help you understand commands

To update Loomio to the latest image you'll need to stop, rm, pull, and run again.

```sh
docker stop loomio
docker rm loomio
docker pull loomio/loomio
docker run
```

To login to your running rails app

```sh
docker exec -t -i loomiodeploy_app_1 bundle exec rails console
```
