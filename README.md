# loomio-deploy
## Use this to setup a Loomio instance

insert, use/buy domain name, setup vps

## Login as root
To login to the server, open a terminal window and type:

```sh
ssh root@loomio.dinotech.co.nz
```

## Clone loomio-deploy

```sh
git clone https://github.com/loomio/loomio-deploy.git
cd loomio-deploy
./scripts/create_swapfile
./scripts/install_docker
service docker start
docker run hello-world
```

If all that went correctly, your terminal should look like this:

![docker hello world](docker_hello_world.png)

Create a config file:

```sh
./scripts/create_env
```

Edit your env

```sh
nano config/env
```

Set your hostname and tld_length. We'll cover smtp settings later

Issue an ssl certificate for your hostname:
docker run -it --rm -p 443:443 -p 80:80 --name letsencrypt \
            -v "/loomio/config/letsencrypt:/etc/letsencrypt" \
            -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
            quay.io/letsencrypt/letsencrypt:latest auth

/usr/local/bin/docker-compose run app rake db:setup
/usr/local/bin/docker-compose up -d

cat crontab >> /etc/crontab

docker-compose run mailin
docker-compose run loomio-pubsub

```sh
docker logs loomio
```

Other need to know docker commands include:
* `docker ps` lists running containers.
* `docker ps -a` lists all containers.
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


So you need an SMTP server, here are some options:

- If you already have a mail server, that's great, you know what to do.

- If you less than 99 emails a day [use your gmail account and smtp.google.com](https://www.digitalocean.com/community/tutorials/how-to-use-google-s-smtp-server) for free.

- Look at the services offered by [SendGrid](https://sendgrid.com/), [SparkPost](https://www.sparkpost.com/), [Mailgun](http://www.mailgun.com/), [Mailjet](https://www.mailjet.com/pricing).

- Very shortly we'll publish a guide to setting up your own secure SMTP server.
