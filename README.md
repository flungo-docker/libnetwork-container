libnetwork-container
====================

Docker network driver for routing through another container.

**NOTE:** This driver is currently in development and the functionality documented in this `README.md` is the goal of what this driver will provide and may not reflect the current functionality of the driver.

# How it works?

This driver allows networks to be created where a container is used as the default gateway for that network. When a network is created a name for the router should be provided otherwise it will default to the network name suffixed with `-router`. When a container with that name is added to the network it will be configurd as the gateway for the network. If any other container is added it will be added as it normally would when using a [bridge](https://github.com/docker/libnetwork/blob/master/docs/bridge.md) network.

# Usage

## Starting the Driver

**NOTE:** Make sure you are using Docker 1.9 or later

To start the driver and make it available for use with `docker network commands` the following command should be used:

```console
$ docker run -d \
    --net host \
    --cap-add NET_ADMIN \
    --name libnetwork-container \
    -v /run/docker/plugins:/run/docker/plugins \
    -v /var/run/docker.sock:/var/run/docker.sock \
    flungo/libnetwork-container
```

## Create a router

To be able to create a network which routes through a container, a container that does the routing needs to be set up. A list of available routers that are designed for use with this driver [is available](#routers) in this `README.md`.

This example creates a router for the Tor network with the name `tor-router`:

```console
$ docker run -d \
    --name tor-router \
    --cap-add NET_ADMIN \
    flungo/tor-router

# follow the logs to make sure it is bootstrapped successfully
$ docker logs -f tor-router
```

## Create a network

The network is created as you would create a network with any other driver. By default, the name required for the routing container will be the name of the network suffixed by `-router` so if a network is created named `tor`, the router should be named `tor-router`. To use a name other than this default, the `me.flungo.network.container.router` option can be set.

The following example creates a network named `vidalia` which will use a router named `tor-router`:

```console
$ docker network create -d container \
    -o me.flungo.network.container.router=tor-router \
    vidalia
```

Once the network is created, you will need to complete this step by adding your router to the network. For the Tor example, this is:

```console
$ docker network connect vidalia tor-router
```

## Run a container

The last step is to connect your containers to the new network. Again this should follow the standard practice you would use to run your container using a specific network.

With the Tor example, the following can be used to test that the request is router through the Tor network.

```console
$ docker run --rm -it --net vidalia jess/httpie \
    -v --json https://check.torproject.org/api/ip
```

# Routers

Below is a list of router images which are designed for use with this driver:

- [Tor Router](https://hub.docker.com/r/flungo/tor-router/)
- [NAT Router](https://hub.docker.com/r/flungo/nat-router/)

# Development

## Running the tests

Unit tests:

```console
$ make test
```

Integration tests:

```console
$ make dtest
```

# Acknowledgements

Thanks to Jess Frazelle for the [onion](https://github.com/jessfraz/onion) driver which this driver is based on and in turn the libnetwork team for writing [the networking go plugin](https://github.com/docker/go-plugins-helpers/tree/master/network) and of course the networking itself.
