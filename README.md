# dokku pushpin [![Build Status](https://img.shields.io/circleci/project/github/dokku/dokku-pushpin.svg?branch=master&style=flat-square "Build Status")](https://circleci.com/gh/dokku/dokku-pushpin/tree/master) [![IRC Network](https://img.shields.io/badge/irc-freenode-blue.svg?style=flat-square "IRC Freenode")](https://webchat.freenode.net/?channels=dokku)

Official pushpin plugin for dokku. Currently defaults to installing [pushpin 1.28.0](https://hub.docker.com/_/pushpin/).

## Requirements

- dokku 0.12.x+
- docker 1.8.x

## Installation

```shell
# on 0.12.x+
sudo dokku plugin:install https://github.com/dokku/dokku-pushpin.git pushpin
```

## Commands

```
pushpin:app-links <app>                        # list all pushpin service links for a given app
pushpin:create <service> [--create-flags...]   # create a pushpin service
pushpin:destroy <service> [-f|--force]         # delete the pushpin service/data/container if there are no links left
pushpin:enter <service>                        # enter or run a command in a running pushpin service container
pushpin:exists <service>                       # check if the pushpin service exists
pushpin:expose <service> <ports...>            # expose a pushpin service on custom port if provided (random port otherwise)
pushpin:info <service> [--single-info-flag]    # print the service information
pushpin:link <service> <app> [--link-flags...] # link the pushpin service to the app
pushpin:linked <service> <app>                 # check if the pushpin service is linked to an app
pushpin:links <service>                        # list all apps linked to the pushpin service
pushpin:list                                   # list all pushpin services
pushpin:logs <service> [-t|--tail]             # print the most recent log(s) for this service
pushpin:promote <service> <app>                # promote service <service> as WEBSOCKET_URL in <app>
pushpin:restart <service>                      # graceful shutdown and restart of the pushpin service container
pushpin:start <service>                        # start a previously stopped pushpin service
pushpin:stop <service>                         # stop a running pushpin service
pushpin:unexpose <service>                     # unexpose a previously exposed pushpin service
pushpin:unlink <service> <app>                 # unlink the pushpin service from the app
pushpin:upgrade <service> [--upgrade-flags...] # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to pushpin:help. Please consult the `pushpin:help` command for any undocumented commands.

### Basic Usage

### create a pushpin service

```shell
# usage
dokku pushpin:create <service> [--create-flags...]
```

flags:

- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-p|--password PASSWORD`: override the user-level service password
- `-r|--root-password PASSWORD`: override the root-level service password

Create a pushpin service named lolipop:

```shell
dokku pushpin:create lolipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the fanout/pushpin image. 

```shell
export WEBSOCKET_IMAGE="fanout/pushpin"
export WEBSOCKET_IMAGE_VERSION="${PLUGIN_IMAGE_VERSION}"
dokku pushpin:create lolipop
```

You can also specify custom environment variables to start the pushpin service in semi-colon separated form. 

```shell
export WEBSOCKET_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku pushpin:create lolipop
```

### print the service information

```shell
# usage
dokku pushpin:info <service> [--single-info-flag]
```

flags:

- `--config-dir`: show the service configuration directory
- `--data-dir`: show the service data directory
- `--dsn`: show the service DSN
- `--exposed-ports`: show service exposed ports
- `--id`: show the service container id
- `--internal-ip`: show the service internal ip
- `--links`: show the service app links
- `--service-root`: show the service root directory
- `--status`: show the service running status
- `--version`: show the service image version

Get connection information as follows:

```shell
dokku pushpin:info lolipop
```

You can also retrieve a specific piece of service info via flags:

```shell
dokku pushpin:info lolipop --config-dir
dokku pushpin:info lolipop --data-dir
dokku pushpin:info lolipop --dsn
dokku pushpin:info lolipop --exposed-ports
dokku pushpin:info lolipop --id
dokku pushpin:info lolipop --internal-ip
dokku pushpin:info lolipop --links
dokku pushpin:info lolipop --service-root
dokku pushpin:info lolipop --status
dokku pushpin:info lolipop --version
```

### list all pushpin services

```shell
# usage
dokku pushpin:list 
```

List all services:

```shell
dokku pushpin:list
```

### print the most recent log(s) for this service

```shell
# usage
dokku pushpin:logs <service> [-t|--tail]
```

flags:

- `-t|--tail`: do not stop when end of the logs are reached and wait for additional output

You can tail logs for a particular service:

```shell
dokku pushpin:logs lolipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku pushpin:logs lolipop --tail
```

### link the pushpin service to the app

```shell
# usage
dokku pushpin:link <service> <app> [--link-flags...]
```

flags:

- `-a|--alias "BLUE_DATABASE"`: an alternative alias to use for linking to an app via environment variable
- `-q|--querystring "pool=5"`: ampersand delimited querystring arguments to append to the service link

A pushpin service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our 'playground' app. 

> NOTE: this will restart your app

```shell
dokku pushpin:link lolipop playground
```

The following environment variables will be set automatically by docker (not on the app itself, so they won’t be listed when calling dokku config):

```
DOKKU_WEBSOCKET_LOLIPOP_NAME=/lolipop/DATABASE
DOKKU_WEBSOCKET_LOLIPOP_PORT=tcp://172.17.0.1:5561
DOKKU_WEBSOCKET_LOLIPOP_PORT_5561_TCP=tcp://172.17.0.1:5561
DOKKU_WEBSOCKET_LOLIPOP_PORT_5561_TCP_PROTO=tcp
DOKKU_WEBSOCKET_LOLIPOP_PORT_5561_TCP_PORT=5561
DOKKU_WEBSOCKET_LOLIPOP_PORT_5561_TCP_ADDR=172.17.0.1
```

The following will be set on the linked application by default:

```
WEBSOCKET_URL=websocket://lolipop:SOME_PASSWORD@dokku-pushpin-lolipop:5561/lolipop
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the 'expose' subcommand. Another service can be linked to your app:

```shell
dokku pushpin:link other_service playground
```

It is possible to change the protocol for `WEBSOCKET_URL` by setting the environment variable `PUSHPIN_DATABASE_SCHEME` on the app. Doing so will after linking will cause the plugin to think the service is not linked, and we advise you to unlink before proceeding. 

```shell
dokku config:set playground PUSHPIN_DATABASE_SCHEME=websocket2
dokku pushpin:link lolipop playground
```

This will cause `WEBSOCKET_URL` to be set as:

```
websocket2://lolipop:SOME_PASSWORD@dokku-pushpin-lolipop:5561/lolipop
```

### unlink the pushpin service from the app

```shell
# usage
dokku pushpin:unlink <service> <app>
```

You can unlink a pushpin service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku pushpin:unlink lolipop playground
```

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### enter or run a command in a running pushpin service container

```shell
# usage
dokku pushpin:enter <service>
```

A bash prompt can be opened against a running service. Filesystem changes will not be saved to disk. 

```shell
dokku pushpin:enter lolipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk. 

```shell
dokku pushpin:enter lolipop touch /tmp/test
```

### expose a pushpin service on custom port if provided (random port otherwise)

```shell
# usage
dokku pushpin:expose <service> <ports...>
```

Expose the service on the service's normal ports, allowing access to it from the public interface (`0.0.0.0`):

```shell
dokku pushpin:expose lolipop 5561 7999 5560 5562 5563
```

### unexpose a previously exposed pushpin service

```shell
# usage
dokku pushpin:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (`0.0.0.0`):

```shell
dokku pushpin:unexpose lolipop
```

### promote service <service> as WEBSOCKET_URL in <app>

```shell
# usage
dokku pushpin:promote <service> <app>
```

If you have a pushpin service linked to an app and try to link another pushpin service another link environment variable will be generated automatically:

```
DOKKU_WEBSOCKET_BLUE_URL=websocket://other_service:ANOTHER_PASSWORD@dokku-pushpin-other-service:5561/other_service
```

You can promote the new service to be the primary one:

> NOTE: this will restart your app

```shell
dokku pushpin:promote other_service playground
```

This will replace `WEBSOCKET_URL` with the url from other_service and generate another environment variable to hold the previous value if necessary. You could end up with the following for example:

```
WEBSOCKET_URL=websocket://other_service:ANOTHER_PASSWORD@dokku-pushpin-other-service:5561/other_service
DOKKU_WEBSOCKET_BLUE_URL=websocket://other_service:ANOTHER_PASSWORD@dokku-pushpin-other-service:5561/other_service
DOKKU_WEBSOCKET_SILVER_URL=websocket://lolipop:SOME_PASSWORD@dokku-pushpin-lolipop:5561/lolipop
```

### start a previously stopped pushpin service

```shell
# usage
dokku pushpin:start <service>
```

Start the service:

```shell
dokku pushpin:start lolipop
```

### stop a running pushpin service

```shell
# usage
dokku pushpin:stop <service>
```

Stop the service and the running container:

```shell
dokku pushpin:stop lolipop
```

### graceful shutdown and restart of the pushpin service container

```shell
# usage
dokku pushpin:restart <service>
```

Restart the service:

```shell
dokku pushpin:restart lolipop
```

### upgrade service <service> to the specified versions

```shell
# usage
dokku pushpin:upgrade <service> [--upgrade-flags...]
```

flags:

- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-R|--restart-apps "true"`: whether to force an app restart

You can upgrade an existing service to a new image or image-version:

```shell
dokku pushpin:upgrade lolipop
```

### Service Automation

Service scripting can be executed using the following commands:

### list all pushpin service links for a given app

```shell
# usage
dokku pushpin:app-links <app>
```

List all pushpin services that are linked to the 'playground' app. 

```shell
dokku pushpin:app-links playground
```

### check if the pushpin service exists

```shell
# usage
dokku pushpin:exists <service>
```

Here we check if the lolipop pushpin service exists. 

```shell
dokku pushpin:exists lolipop
```

### check if the pushpin service is linked to an app

```shell
# usage
dokku pushpin:linked <service> <app>
```

Here we check if the lolipop pushpin service is linked to the 'playground' app. 

```shell
dokku pushpin:linked lolipop playground
```

### list all apps linked to the pushpin service

```shell
# usage
dokku pushpin:links <service>
```

List all apps linked to the 'lolipop' pushpin service. 

```shell
dokku pushpin:links lolipop
```

### Disabling `docker pull` calls

If you wish to disable the `docker pull` calls that the plugin triggers, you may set the `PUSHPIN_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker pull` is disabled.