---
title: "Miniflux on Raspberry Pi"
date: 2023-01-16T21:03:12+02:00

draft: false
---

## Rationale

In context of the ongoing purge of services one likes to use, 
and a link about RSS readers that you can self-host [RSS readers that you can self-host](https://rohanrd.xyz/posts/rss-readers-that-you-can-self-host/), I decided to give [Miniflux](https://miniflux.app/) a try.

This post is mostly reference for a future self but explains what to do to have a working 
Miniflux installation on your Raspberry Pi on your local network. 

## Pre-requisites

I used a Raspberry Pi 4 (4GB) that is doing some file-serving and managing TV recordings [TVHeadend](https://tvheadend.org/) on my network. Standard Raspbian lite, nothing fancy. 

## PostgreSQL installation and setup

You need a working installation of PostgreSQL to use Miniflux. 
When installing, we can directly setup the postgres user and the miniflux user and database. 

```shell
$ apt update
...
$ apt install postgresql

```

Set a password for the postgres user:

```shell
$ passwd postgres
```

Follow the instructions for setting up the miniflux database user and the miniflux database:

```shell
# Switch to the postgres user 
# (this is what you need an active postgres password for)
$ su - postgres

# Create a database user for Miniflux
$ createuser -P miniflux
Enter password for new role: ******
Enter it again: ******

# Create a database for miniflux that belongs to our user 
# (call it miniflux2 because this is somehow predefined in the configs.)
$ createdb -O miniflux miniflux2 

# Create the extension hstore as superuser
$ psql miniflux -c 'create extension hstore'
CREATE EXTENSION
```

## Miniflux installation and setup

I set up the miniflux installation via its APT repository to enable global updates via apt (stolen from the miniflux website): 

```shell
$ echo "deb [trusted=yes] https://repo.miniflux.app/apt/ /" | sudo tee /etc/apt/sources.list.d/miniflux.list > /dev/null
$ apt update
# finally:
$ apt install miniflux
```

I checked the installation and connection via systemctl:

```shell
$ systemctl status miniflux.service
● miniflux.service - Miniflux
   Loaded: loaded (/lib/systemd/system/miniflux.service; enabled; vendor preset: en
   Active: active (running) since Sun 2023-01-15 22:06:56 CET; 2s ago
     Docs: man:miniflux(1)
           https://miniflux.app/docs/index.html
 Main PID: 10322 (miniflux)
    Tasks: 10 (limit: 4915)
   CGroup: /system.slice/miniflux.service
           └─10322 /usr/bin/miniflux

Jan 15 22:06:55 lithium systemd[1]: Starting Miniflux...
Jan 15 22:06:55 lithium miniflux[10322]: -> Current schema version: 62
Jan 15 22:06:55 lithium miniflux[10322]: -> Latest schema version: 62
Jan 15 22:06:55 lithium miniflux[10322]: [INFO] Starting Miniflux...
Jan 15 22:06:55 lithium miniflux[10322]: [INFO] Starting scheduler...
Jan 15 22:06:56 lithium miniflux[10322]: [INFO] Sending readiness notification to S
Jan 15 22:06:56 lithium miniflux[10322]: [INFO] Listening on "192.168.178.96:8472" 
Jan 15 22:06:56 lithium miniflux[10322]: [INFO] Activating Systemd watchdog
Jan 15 22:06:56 lithium systemd[1]: Started Miniflux.
``` 
I made some updates to the config file, to enable local network access and connect to the correct database with the correct user. The most important bit here was, to restart everything after the update to the 
configuration to actually enable it. 

```bash
$ cat /etc/miniflux.cfg
# See https://miniflux.app/docs/configuration.html
DATABASE_URL=user=miniflux password=****** dbname=miniflux2 sslmode=disable
LISTEN_ADDR=192.168.178.96:8472
RUN_MIGRATIONS=1
# Set up an admin user to get into the web-application
$ miniflux -create-admin
```

## Additional niceties

If you want to try out an updated configuration, a good approach is to disable the service and run it in debug mode:

```bash
$ systemctl stop miniflux.service
# start miniflux with a specific config-file:
$ miniflux -debug -c /etc/miniflux.cfg
```

While the webapp is great especially on a desktop browser (and fast), I prefer native apps on my phone. 
There are two services you can activate in settings to enable some third party apps: Google Reader API and Fever API. This allows you to use apps like Reeder or Unread in iOS. Other options are also [available](https://miniflux.app/docs/services.html).
I personally also like the option to directly save an article to Instapaper.  