# Deploy your own Loomio

This repo contains a docker-compose configuration for running Loomio on your own server and domain name.

It assumes you want to run everything on a single host and automatically issues an SSL certificate via the amazing [letsencrypt.org](https://letsencrypt.org/).

If you want a local install of Loomio to develop features with, please visit the [development handbook](https://help.loomio.org/en/dev_manual/setup_dev_environment/).


## What you'll need
* Root access to a server, on a public IP address, running a default configuration of Ubuntu 18.04.

* A domain name which you can create DNS records for.

* An SMTP server for sending email. More on that below.

## Network configuration
What hostname will you be using for your Loomio instance? What is the IP address of your server?

For the purposes of this example, the hostname will be loomio.example.com and the IP address is 123.123.123.123

### DNS Records

To allow people to access the site via your hostname you need an A record:

```
A loomio.example.com, 123.123.123.123
```

Loomio supports "Reply by email" and to enable this you need an MX record so mail servers know where to direct these emails.

```
MX loomio.example.com, loomio.example.com, priority 0
```


## Configure the server

### Login as root
To login to the server, open a terminal window and type:

```sh
ssh -A root@loomio.example.com
```

### Install docker and docker-compose

There are many ways to install docker. Here is one way to do so, as described in the [Docker Documentation](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-using-the-convenience-script).

```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

These commands fetch and install docker-compose

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### Clone the loomio-deploy repo
This where all the configuration for your Loomio services will live. In this step you make a copy of this repo, so that you can modify the settings to work for your particular setup.

As root on your server, clone this repo:

```sh
git clone https://github.com/loomio/loomio-deploy.git
cd loomio-deploy
```

The commands below assume your working directory is this repo, on your server.

### Setup a swapfile (optional)
There are some simple scripts within this repo to help you configure your server.

This script will create and mount a 4GB swapfile. If you have less than 2GB RAM on your server then this step is required.

```sh
./scripts/create_swapfile
```

### Create your ENV files
This script creates `env` files configured for your server. It also creates directories on the host to hold user data.

When you run this, remember to swap `loomio.example.com` for your hostname, and give your contact email address, so you can recover your SSL keys later if required.

```sh
./scripts/create_env loomio.example.com you@contact.email
```

Now have a look inside the files:

```sh
cat env
```

### Basic usage reporting

By default your Loomio instance will report back to www.loomio.org once a day with the number of discussions, comments, polls, stances, users and visits that your site has had.

We encourage you not to disable this, so that we can tell what impact our work is having, which in turn helps us to continue to develop Loomio.

If you wish to disable this reporting function, add the following line to your `env` file

```
DISABLE_USAGE_REPORTING=1
```

### Configure SMTP settings

Loomio requires an SMTP server to function. In this step you need to edit your `env` file and configure the SMTP settings for outbound emails.

If you already an SMTP server you can use, that's great, add the config to your env file (see the SMTP_ prefixed variables).

Those of you without an existing SMTP server, may wish to use a service such as: [SendGrid](https://sendgrid.com/), [SparkPost](https://www.sparkpost.com/),
[Mailgun](http://www.mailgun.com/),
[Mailjet](https://www.mailjet.com/pricing), or run your own SMTP with [Haraka](https://haraka.github.io/manual/tutorials/SettingUpOutbound.html)

You should add an SPF record to indicate that the SMTP can send mail for your loomio domain.

### Initialize the database
This command initializes a new database for your Loomio instance to use.

```
docker-compose run app rake db:setup
```

### Setup crontab

To edit your crontab use:
```
crontab -e
```

and paste the following lines in there

```
0 * * * *  docker exec loomio-worker bundle exec rake loomio:hourly_tasks
```

## Starting the services
This command starts the database, application, reply-by-email, and live-update services all at once.

```
docker-compose up -d
```


## Try it out

visit your hostname in your browser.

Once you have signed in (and confirmed your email), grant yourself admin rights

```
docker-compose run app rails c
User.last.update(is_admin: true)
```

you can now access the admin interface at https://loomio.example.com/admin


## If something goes wrong

Check the logs. These will almost always tell you what the problem is.

```
docker-compose logs -f
```

Confirm `env` settings are correct. After you change your `env` files you need to restart the system:

```sh
docker-compose down
docker-compose up -d
```

To update Loomio to the latest image you'll need to stop, rm, pull, apply potential changes to the database schema, and run again.

```sh
docker-compose down
docker-compose pull
docker-compose run app rake db:migrate
docker-compose up -d
```

From time to time, or if you are running out of disk space

```sh
docker system prune
```

To login to your running rails app console:

```sh
docker-compose run app rails console
```

A PostgreSQL shell to inspect the database:

```sh
docker exec -ti loomio-db su - postgres -c 'psql loomio_production'
```

### Sign in via third party

If you want to allow users to sign in via Google, Facebook, Twitter or Github, then you'll need to add APP_KEY and APP_SECRET env lines for each provider you wish to support.

You'll find env records for these providers commented out in your env file by default.

To register your app with Facebook, visit https://developers.facebook.com

For twitter, https://apps.twitter.com/
- Website: https://loomio.example.com
- callback url: https://loomio.example.com/twitter/authorize

For Google: https://console.developers.google.com/
- Authorized JavaScript origins: https://loomio.example.com
- Authorized redirect URIs: https://loomio.example.com/google/authorize
- domain verification: https://loomio.example.com


## Backups

There is a simple backup script, intended to be run daily.

Run it manually like so:

```
scripts/backup .
```

Or add a line to your crontab like so:

```
0 0 * * *  /root/loomio-deploy/scripts/create_backup /root/loomio-deploy > ~/loomio-deploy-backup.log 2>&1
```

It's left as an exercise for you to copy the backup tarball somewhere after it is created.

*Need some help?* Visit the [Installing Loomio group](https://www.loomio.org/g/C7I2YAPN/loomio-community-installing-loomio).
