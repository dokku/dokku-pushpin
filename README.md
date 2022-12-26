# dokku pushpin [![Build Status](https://img.shields.io/github/workflow/status/dokku/dokku-pushpin/CI/master?style=flat-square "Build Status")](https://github.com/dokku/dokku-pushpin/actions/workflows/ci.yml?query=branch%3Amaster) [![IRC Network](https://img.shields.io/badge/irc-libera-blue.svg?style=flat-square "IRC Libera")](https://webchat.libera.chat/?channels=dokku)

Official pushpin plugin for dokku. Currently defaults to installing [fanout/pushpin 1.35.0](https://hub.docker.com/r/fanout/pushpin/).

## Requirements

- dokku 0.19.x+
- docker 1.8.x

## Installation

```shell
# on 0.19.x+
sudo dokku plugin:install https://github.com/dokku/dokku-pushpin.git pushpin
```

## Commands

```
pushpin:app-links <app>                            # list all pushpin service links for a given app
pushpin:create <service> [--create-flags...]       # create a pushpin service
pushpin:destroy <service> [-f|--force]             # delete the pushpin service/data/container if there are no links left
pushpin:enter <service>                            # enter or run a command in a running pushpin service container
pushpin:exists <service>                           # check if the pushpin service exists
pushpin:expose <service> <ports...>                # expose a pushpin service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)
pushpin:info <service> [--single-info-flag]        # print the service information
pushpin:link <service> <app> [--link-flags...]     # link the pushpin service to the app
pushpin:linked <service> <app>                     # check if the pushpin service is linked to an app
pushpin:links <service>                            # list all apps linked to the pushpin service
pushpin:list                                       # list all pushpin services
pushpin:logs <service> [-t|--tail] <tail-num-optional> # print the most recent log(s) for this service
pushpin:pause <service>                            # pause a running pushpin service
pushpin:promote <service> <app>                    # promote service <service> as WEBSOCKET_URL in <app>
pushpin:restart <service>                          # graceful shutdown and restart of the pushpin service container
pushpin:start <service>                            # start a previously stopped pushpin service
pushpin:stop <service>                             # stop a running pushpin service
pushpin:unexpose <service>                         # unexpose a previously exposed pushpin service
pushpin:unlink <service> <app>                     # unlink the pushpin service from the app
pushpin:upgrade <service> [--upgrade-flags...]     # upgrade service <service> to the specified versions
```

## Usage

Help for any commands can be displayed by specifying the command as an argument to pushpin:help. Plugin help output in conjunction with any files in the `docs/` folder is used to generate the plugin documentation. Please consult the `pushpin:help` command for any undocumented commands.

### Basic Usage

### create a pushpin service

```shell
# usage
dokku pushpin:create <service> [--create-flags...]
```

flags:

- `-c|--config-options "--args --go=here"`: extra arguments to pass to the container create command (default: `None`)
- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-m|--memory MEMORY`: container memory limit in megabytes (default: unlimited)
- `-p|--password PASSWORD`: override the user-level service password
- `-r|--root-password PASSWORD`: override the root-level service password
- `-s|--shm-size SHM_SIZE`: override shared memory size for pushpin docker container

Create a pushpin service named lollipop:

```shell
dokku pushpin:create lollipop
```

You can also specify the image and image version to use for the service. It *must* be compatible with the fanout/pushpin image.

```shell
export PUSHPIN_IMAGE="fanout/pushpin"
export PUSHPIN_IMAGE_VERSION="${PLUGIN_IMAGE_VERSION}"
dokku pushpin:create lollipop
```

You can also specify custom environment variables to start the pushpin service in semi-colon separated form.

```shell
export PUSHPIN_CUSTOM_ENV="USER=alpha;HOST=beta"
dokku pushpin:create lollipop
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
dokku pushpin:info lollipop
```

You can also retrieve a specific piece of service info via flags:

```shell
dokku pushpin:info lollipop --config-dir
dokku pushpin:info lollipop --data-dir
dokku pushpin:info lollipop --dsn
dokku pushpin:info lollipop --exposed-ports
dokku pushpin:info lollipop --id
dokku pushpin:info lollipop --internal-ip
dokku pushpin:info lollipop --links
dokku pushpin:info lollipop --service-root
dokku pushpin:info lollipop --status
dokku pushpin:info lollipop --version
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
dokku pushpin:logs <service> [-t|--tail] <tail-num-optional>
```

flags:

- `-t|--tail [<tail-num>]`: do not stop when end of the logs are reached and wait for additional output

You can tail logs for a particular service:

```shell
dokku pushpin:logs lollipop
```

By default, logs will not be tailed, but you can do this with the --tail flag:

```shell
dokku pushpin:logs lollipop --tail
```

The default tail setting is to show all logs, but an initial count can also be specified:

```shell
dokku pushpin:logs lollipop --tail 5
```

### link the pushpin service to the app

```shell
# usage
dokku pushpin:link <service> <app> [--link-flags...]
```

flags:

- `-a|--alias "BLUE_DATABASE"`: an alternative alias to use for linking to an app via environment variable
- `-q|--querystring "pool=5"`: ampersand delimited querystring arguments to append to the service link

A pushpin service can be linked to a container. This will use native docker links via the docker-options plugin. Here we link it to our `playground` app.

> NOTE: this will restart your app

```shell
dokku pushpin:link lollipop playground
```

The following environment variables will be set automatically by docker (not on the app itself, so they won’t be listed when calling dokku config):

```
DOKKU_PUSHPIN_LOLLIPOP_NAME=/lollipop/DATABASE
DOKKU_PUSHPIN_LOLLIPOP_PORT=tcp://172.17.0.1:5561
DOKKU_PUSHPIN_LOLLIPOP_PORT_5561_TCP=tcp://172.17.0.1:5561
DOKKU_PUSHPIN_LOLLIPOP_PORT_5561_TCP_PROTO=tcp
DOKKU_PUSHPIN_LOLLIPOP_PORT_5561_TCP_PORT=5561
DOKKU_PUSHPIN_LOLLIPOP_PORT_5561_TCP_ADDR=172.17.0.1
```

The following will be set on the linked application by default:

```
WEBSOCKET_URL=websocket://dokku-pushpin-lollipop:5561
```

The host exposed here only works internally in docker containers. If you want your container to be reachable from outside, you should use the `expose` subcommand. Another service can be linked to your app:

```shell
dokku pushpin:link other_service playground
```

It is possible to change the protocol for `WEBSOCKET_URL` by setting the environment variable `PUSHPIN_DATABASE_SCHEME` on the app. Doing so will after linking will cause the plugin to think the service is not linked, and we advise you to unlink before proceeding.

```shell
dokku config:set playground PUSHPIN_DATABASE_SCHEME=websocket2
dokku pushpin:link lollipop playground
```

This will cause `WEBSOCKET_URL` to be set as:

```
websocket2://dokku-pushpin-lollipop:5561
```

### unlink the pushpin service from the app

```shell
# usage
dokku pushpin:unlink <service> <app>
```

You can unlink a pushpin service:

> NOTE: this will restart your app and unset related environment variables

```shell
dokku pushpin:unlink lollipop playground
```

### Service Lifecycle

The lifecycle of each service can be managed through the following commands:

### enter or run a command in a running pushpin service container

```shell
# usage
dokku pushpin:enter <service>
```

A bash prompt can be opened against a running service. Filesystem changes will not be saved to disk.

> NOTE: disconnecting from ssh while running this command may leave zombie processes due to moby/moby#9098

```shell
dokku pushpin:enter lollipop
```

You may also run a command directly against the service. Filesystem changes will not be saved to disk.

```shell
dokku pushpin:enter lollipop touch /tmp/test
```

### expose a pushpin service on custom host:port if provided (random port on the 0.0.0.0 interface if otherwise unspecified)

```shell
# usage
dokku pushpin:expose <service> <ports...>
```

Expose the service on the service's normal ports, allowing access to it from the public interface (`0.0.0.0`):

```shell
dokku pushpin:expose lollipop 5561 7999 5560 5562 5563
```

Expose the service on the service's normal ports, with the first on a specified ip adddress (127.0.0.1):

```shell
dokku pushpin:expose lollipop 127.0.0.1:5561 7999 5560 5562 5563
```

### unexpose a previously exposed pushpin service

```shell
# usage
dokku pushpin:unexpose <service>
```

Unexpose the service, removing access to it from the public interface (`0.0.0.0`):

```shell
dokku pushpin:unexpose lollipop
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
DOKKU_WEBSOCKET_SILVER_URL=websocket://lollipop:SOME_PASSWORD@dokku-pushpin-lollipop:5561/lollipop
```

### start a previously stopped pushpin service

```shell
# usage
dokku pushpin:start <service>
```

Start the service:

```shell
dokku pushpin:start lollipop
```

### stop a running pushpin service

```shell
# usage
dokku pushpin:stop <service>
```

Stop the service and removes the running container:

```shell
dokku pushpin:stop lollipop
```

### pause a running pushpin service

```shell
# usage
dokku pushpin:pause <service>
```

Pause the running container for the service:

```shell
dokku pushpin:pause lollipop
```

### graceful shutdown and restart of the pushpin service container

```shell
# usage
dokku pushpin:restart <service>
```

Restart the service:

```shell
dokku pushpin:restart lollipop
```

### upgrade service <service> to the specified versions

```shell
# usage
dokku pushpin:upgrade <service> [--upgrade-flags...]
```

flags:

- `-c|--config-options "--args --go=here"`: extra arguments to pass to the container create command (default: `None`)
- `-C|--custom-env "USER=alpha;HOST=beta"`: semi-colon delimited environment variables to start the service with
- `-i|--image IMAGE`: the image name to start the service with
- `-I|--image-version IMAGE_VERSION`: the image version to start the service with
- `-R|--restart-apps "true"`: whether to force an app restart
- `-s|--shm-size SHM_SIZE`: override shared memory size for pushpin docker container

You can upgrade an existing service to a new image or image-version:

```shell
dokku pushpin:upgrade lollipop
```

### Service Automation

Service scripting can be executed using the following commands:

### list all pushpin service links for a given app

```shell
# usage
dokku pushpin:app-links <app>
```

List all pushpin services that are linked to the `playground` app.

```shell
dokku pushpin:app-links playground
```

### check if the pushpin service exists

```shell
# usage
dokku pushpin:exists <service>
```

Here we check if the lollipop pushpin service exists.

```shell
dokku pushpin:exists lollipop
```

### check if the pushpin service is linked to an app

```shell
# usage
dokku pushpin:linked <service> <app>
```

Here we check if the lollipop pushpin service is linked to the `playground` app.

```shell
dokku pushpin:linked lollipop playground
```

### list all apps linked to the pushpin service

```shell
# usage
dokku pushpin:links <service>
```

List all apps linked to the `lollipop` pushpin service.

```shell
dokku pushpin:links lollipop
```

### Disabling `docker pull` calls

If you wish to disable the `docker pull` calls that the plugin triggers, you may set the `PUSHPIN_DISABLE_PULL` environment variable to `true`. Once disabled, you will need to pull the service image you wish to deploy as shown in the `stderr` output.

Please ensure the proper images are in place when `docker pull` is disabled.
