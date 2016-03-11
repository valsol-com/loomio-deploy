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
wget -qO- https://get.docker.com/ | sh
wget -O /usr/local/bin/docker-compose https://github.com/docker/compose/releases/download/1.6.2/docker-compose-`uname -s`-`uname -m`
chmod +x /usr/local/bin/docker-compose

git clone https://github.com/loomio/loomio-deploy.git
./scripts/install_docker
cd loomio-deploy
./scripts/create_swapfile
replacce with:
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
edit the defaults to be the example hostname.

Set your hostname and tld_length.

You will need an SMTP server, here are some options:

- If you already have a mail server, that's great, you know what to do.

- For setups that will send less than 99 emails a day [use smtp.google.com](https://www.digitalocean.com/community/tutorials/how-to-use-google-s-smtp-server) for free.

- Look at the services offered by [SendGrid](https://sendgrid.com/), [SparkPost](https://www.sparkpost.com/), [Mailgun](http://www.mailgun.com/), [Mailjet](https://www.mailjet.com/pricing).

- Very shortly we'll publish a guide to setting up your own secure SMTP server.

Issue an ssl certificate for your hostname:

```sh
docker run -it --rm -p 443:443 -p 80:80 --name letsencrypt \
            -v "/root/loomio-deploy/certificates/:/etc/letsencrypt" \
            -v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
            quay.io/letsencrypt/letsencrypt:latest auth
```

``` Setup the database
/usr/local/bin/docker-compose run app rake db:setup
```

``` start the system
/usr/local/bin/docker-compose up -d
```

install crontab
note: change it to use docker-compose run
```
cat crontab >> /etc/crontab
```

# these will be run by docker-compose automatically
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

To login to your running rails app

```sh
docker exec -t -i loomiodeploy_app_1 bundle exec rails console
```
