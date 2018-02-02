# [wslph/mule](https://hub.docker.com/r/wslph/mule)

- [Introduction](#introduction)
    - [Changelog](https://git.whiteskylabs.com/mulesoft/docker-mule/wikis/changelog)
- [Contributing](#contributing)
- [Issues](#issues)
- [Announcements](https://git.whiteskylabs.com/mulesoft/docker-mule/issues)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
    - [Installing Mule Enterprise License](#installing-mule-enterprise-license)
- [Configuration](#configuration)
    - [Available Configuration Parameters](#available-configuration-parameters)
- [References](#references)
- [About](#about)

---
# Introduction

Docker [Mule ESB Runtime](https://www.mulesoft.com/ty/dl/mule) container image for Community and Enterprise Edition.

---
# Contributing

If you find this image useful here's how you can help:

- Send a Merge Request with your awesome new features and bug fixes at https://git.whiteskylabs.com/mulesoft/docker-mule
- Be a part of the community and help resolve [Issues](https://git.whiteskylabs.com/mulesoft/docker-mule/issues)

---
# Issues

Docker is a relatively new project and is active being developed and tested by a thriving community of developers and testers, and every release includes many enhancements and bugfixes.

Given the nature of the development and release cycle it is very important that you have the latest version of Docker installed because any issue that you encounter might have already been fixed with a newer Docker release.

Install the most recent version of the Docker Engine for your platform using the [official Docker releases](http://docs.docker.com/engine/installation/), which can also be installed using:

```bash
wget -qO- https://get.docker.com/ | sh
```

Fedora and RHEL/CentOS users should try disabling selinux with `setenforce 0` and check if resolves the issue. If it does than there is not much that I can help you with. You can either stick with selinux disabled (not recommended by redhat) or switch to using ubuntu.

If you want to try this on your local development machine, you can try [Docker for Mac](https://docs.docker.com/docker-for-mac/), and [Docker for Windows](https://docs.docker.com/docker-for-windows/).

If you find any issues with this release, please file an issue request on the [issues](https://git.whiteskylabs.com/mulesoft/docker-mule/issues) page.

In your issue report please make sure you provide the following information:

- The host distribution and release version.
- Output of the `docker version` command
- Output of the `docker info` command
- The `docker run` command you used to run the image (mask out the sensitive bits).

---
# Prerequisites

Your docker host needs to have at least 2GB (or more) of available RAM to run Mule ESB Server Runtime. Please refer to the Mule ESB [hardware requirements](https://docs.mulesoft.com/mule-user-guide/v/3.9/hardware-and-software-requirements) documentation for additional information.

---
# Installation

Automated builds of the image are available on [Dockerhub](https://hub.docker.com/r/wslph/mule) and is the recommended method of installation.

---
To pull the latest build for 3.9.0-ee

```bash
docker pull wslph/mule:3.9.0-ee
```
---
To pull the container built on 2018-02-02 (YYYY-MM-DD).

```bash
docker pull wslph/mule:3.9.0-ee-2018-02-02
```
---
Example 1 : Launching  Mule ESB Server runtime container for Community Edition

```bash
docker run --restart=always --name mule-ce -d \
    --publish 7777:7777 \
    --env 'MULE_ENV=dev' \
    wslph/mule:3.9.0-ce && docker logs -f mule-ce
```
---
Example 2 : Launching  Mule ESB Server runtime container for Enterprise Edition

```bash
docker run --restart=always --name mule-ee -d \
    --publish 7777:7777 \
    --env 'MULE_ENV=dev' \
    wslph/mule:3.9.0-ee && docker logs -f mule-ee
```
---
Example 3 : Attaching container to host network

```bash
docker run --restart=always --name mule-ee -d \
    --net=host \
    --env 'MULE_ENV=dev' \
    wslph/mule:3.9.0-ee && docker logs -f mule-ee
```

---

*Please refer to [Available Configuration Parameters](#available-configuration-parameters) to understand `MULE_ENV` and other configuration options*

__NOTE__: Please allow a couple of minutes for the Mule ESB Server runtime to start.

In any of these examples, you should now have Mule ESB Server up and ready for testing. If you want to use this image in production then please read on.

---
# Installing Mule Enterprise License

By default, enterprise edition of Mule ESB Server will run in trial mode. If you want to [install an enterprise license](#installing-mule-enterprise-license) which is either named as  `license.lic`  or `mule-ee-license.lic`, just drop it in your `<mule-home>/conf`, and then restart your mule container, it will detect and install it. Just do the same process, whenever you need to renew your mule license.

---
# Configuration

**Data Store**

Mule ESB Server is an application hosting software and as such you don't want to lose your code when the docker container is stopped/deleted. To avoid losing any data, you should mount a volume at,

* `/opt/mule/apps`
* `/opt/mule/domains`
* `/opt/mule/conf`
* `/opt/mule/logs`
* `/opt/mule/patches`
* `/opt/mule/.mule`

Volumes can be mounted in docker by specifying the `--volume` option in the docker run command.

```bash
docker run --restart=always --name mule-ee -d \
    --publish 7777:7777 --publish 1098:1098 --publish 25000:5000 --publish 8081:8081\
    --env 'MULE_ENV=dev' \
    --env 'MULE_ESB_NAME=My-ESB-Server-01' \
    --env 'MULE_MMC_AGENT_PORT=7777' \
    --env 'MULE_MMC_URL=http://my-server.example.com/mmc-console' \
    --env 'MULE_MMC_USERNAME=admin' \
    --env 'MULE_MMC_PASSWORD=admin' \
    --env 'MULE_MMC_AGENT_HOST=123.455.234.110' \
    --volume ~/mule/apps:/opt/mule/apps \
    --volume ~/mule/domains:/opt/mule/domains \
    --volume ~/mule/conf:/opt/mule/conf \
    --volume ~/mule/logs:/opt/mule/logs \
    --volume ~/mule/patches:/opt/mule/patches \
    --volume ~/mule/.mule:/opt/mule/.mule \
    wslph/mule:3.9.0-ee && docker logs -f mule-ee
```

---
**Applying runtime patch**

The `/opt/mule/patches` volume sole purpose is to copy (before the mule runtime is started) its content to `/opt/mule/lib/user` and applies to specific version of mule runtime (e.g., `3.6.2-ee` requires applying `SE-3341-3.6.2.jar` that fixes the mule application filename issue). This allows maintenance of patch to an existing Mule runtime, requiring only a `docker restart <mule-container>`

---
**Registering to Anypoint Runtime Manager**

If you want to register your Mule ESB container to Anypoint Platform / Anypoint Runtime Manager, here's an example:

```bash
docker run --restart=always --name mule-ee -d \
    --publish 1098:1098 --publish 25000:5000 --publish 8081:8081\
    --env 'MULE_ENV=dev' \
    --env 'MULE_ESB_NAME=My-ESB-Server-01' \
    --env 'MULE_ARM_TOKEN=aaabbbcccdddeeefff...' \
    --volume ~/mule/apps:/opt/mule/apps \
    --volume ~/mule/domains:/opt/mule/domains \
    --volume ~/mule/conf:/opt/mule/conf \
    --volume ~/mule/logs:/opt/mule/logs \
    --volume ~/mule/patches:/opt/mule/patches \
    --volume ~/mule/.mule:/opt/mule/.mule \
    wslph/mule:3.9.0-ee && docker logs -f mule-ee
```

---
**Registering to Mule Management Console**

If you want to register your Mule ESB container to a MMC server.

You need to specify the following environment variables:
- `MULE_ESB_NAME`
- `MULE_MMC_AGENT_HOST`
- `MULE_MMC_AGENT_PORT`
- `MULE_MMC_URL`
- `MULE_MMC_USERNAME` and `MULE_MMC_PASSWORD` OR `MULE_MMC_HEADER_AUTH`

For further explanation of each environment variable, please take a look at *Available Configuration Parameters* section.

Here's an example:

```bash
docker run --restart=always --name mule-ee -d \
    --publish 7777:7777 --publish 1098:1098 --publish 25000:5000 --publish 8081:8081\
    --env 'MULE_ENV=dev' \
    --env 'MULE_ESB_NAME=My-ESB-Server-01' \
    --env 'MULE_MMC_AGENT_HOST=123.455.234.110' \
    --env 'MULE_MMC_AGENT_PORT=7777' \
    --env 'MULE_MMC_URL=http://my-server.example.com/mmc-console' \
    --env 'MULE_MMC_USERNAME=admin' \
    --env 'MULE_MMC_PASSWORD=admin' \
    --volume ~/mule/apps:/opt/mule/apps \
    --volume ~/mule/domains:/opt/mule/domains \
    --volume ~/mule/conf:/opt/mule/conf \
    --volume ~/mule/logs:/opt/mule/logs \
    --volume ~/mule/patches:/opt/mule/patches \
    --volume ~/mule/.mule:/opt/mule/.mule \
    wslph/mule:3.9.0-ee && docker logs -f mule-ee
```

OR

```bash
docker run --restart=always --name mule-ee -d \
    --publish 7777:7777 --publish 1098:1098 --publish 25000:5000 --publish 8081:8081\
    --env 'MULE_ENV=dev' \
    --env 'MULE_ESB_NAME=My-ESB-Server-01' \
    --env 'MULE_MMC_AGENT_HOST=123.455.234.110' \
    --env 'MULE_MMC_AGENT_PORT=7777' \
    --env 'MULE_MMC_URL=http://my-server.example.com/mmc-console' \
    --env 'MULE_MMC_HEADER_AUTH=Authorization:Basic YWRtaW46YWRtaW4=' \
    --volume ~/mule/apps:/opt/mule/apps \
    --volume ~/mule/domains:/opt/mule/domains \
    --volume ~/mule/conf:/opt/mule/conf \
    --volume ~/mule/logs:/opt/mule/logs \
    --volume ~/mule/patches:/opt/mule/patches \
    --volume ~/mule/.mule:/opt/mule/.mule \
    wslph/mule:3.9.0-ee && docker logs -f mule-ee
```
---
**IMPORTANT:**

In order to execute the registration script to MMC server, you need to do the following:
```
docker exec <container-name> mmc_setup
```

and it will provide a JSON response, which indicates wether registration is successful or not.

**Developer's Note:**
- *At the moment, this is the only way to register to MMC server. If you have a better way of doing this inside the container, I'd be happy to implement it and credit it to you. Thanks!*


**Available Configuration Parameters**

*Please refer the docker run command options for the `--env-file` flag where you can specify all required environment variables in a single file. This will save you from writing a potentially long docker run command.*

Below is the complete list of available options that can be used to customize your Mule ESB Server container installation.

| Parameter | Description |
|-----------|-------------|
| `MULE_ENV` | by default, its value is `dev`, it can be anything such as `sit`, `stg`, `prd`, or anything. This is accessible within Mule environment and mule application via `${mule.env}` |
| `APP_CONFIG_PATH` | By default its value is empty, but it can be overwritten. However anything set here is relative to `/opt/mule/conf/` . This is accessible within Mule environment and mule application via `${mule.config.path}` |
| `MULE_STARTUP_ENV_CONFIG` | Useful, when you need to specify some environment variable to mule upon start. For example you can pass a `-M-Dmule.vault.key=mulesoft` parameter. However, that this does not override the other environment variable in this table. |
| `MULE_ENV_OVERRIDE_DEFAULT` | 0 by default. If set to 1, it will load the existing host resources (e.g., `apps`, `domains`, and `conf`) of the host specified path at `--volume`. |
| `MULE_ARM_TOKEN` | If you want to register this container to Anypoint Runtime Manager, pass the token here. |
| `MULE_ESB_NAME` | Specify the name you want this container to use at Anypoint Management Center, or Mule Management Console. By default, it will generate its own name with a prefix of `MULE-ESB-` and appended with timestamp to register. |
| `MULE_MMC_URL` | Specify the URL where your MMC (Mule Management Console) is hosted (`https://<mmc-host>:<mmc-port>`). For example `https://mydomain.example.com/mmc` |
| `MULE_MMC_AGENT_HOST` | Specify the hostname and/or IP address where this server is hosted. For example `my-esb-server.example.com`, or `123.455.234.110` to complete the full url `http://<mmc-host>:MULE_MMC_AGENT_PORT/mmc-support`|
| `MULE_MMC_AGENT_PORT` | This is the port for when you need to register this mule esb container to an MMC server. By default its value is `7777`, but you can change this to use other port (e.g., when you want to have multiple Mule esb runtime container on the same host. |
| `MULE_MMC_USERNAME` | Specify the username to be used to authenticate to MMC, with premission to create, update, and delete server groups, as well as register new server. Note: If `MULE_MMC_HEADER_AUTH` is specified. This will be ignored. |
| `MULE_MMC_PASSWORD` | Specify the password for to be used to authentication. Note: If `MULE_MMC_HEADER_AUTH` is specified. This will be ignored. |
| `MULE_MMC_HEADER_AUTH` | Specify the default `header` for authorization to your MMC. Note: When specified, this will disable `MULE_MMC_USERNAME` and `MULE_MMC_PASSWORD`. For example, `Authorization:Basic YWRtaW46YWRtaW4=`|
| `MULE_MMC_SERVER_GROUP` | If specified, server will be registered and added into the specified server group. |

# Maintenance

**Shell Access**

For debugging and maintenance purposes you may want access the container's shell. If you are using docker version `1.3.0` or higher you can access a running container's shell using `docker exec` command.

```bash
docker exec -it mule-ee bash
```
---
# References

* For available server runtime version please see [tags](https://hub.docker.com/r/wslph/mule/tags/)
* [Mule ESB User Guide](https://docs.mulesoft.com/mule-user-guide/v/3.9/)
* Container is based from [wslph/ubuntu:xenial-latest](https://hub.docker.com/r/wslph/ubuntu)
* Mule ESB 3.6.x containers are installed with Oracle JDK7, while Mule ESB version 3.7.x and up are installed with the Oracle JDK8.
* Mule Agent version is updated to **1.9.3** for Mule ESB Enterprise Edition v3.6.x to 3.9.x. Mule runtime v4.0.0 have v2.0 of this agent, therefore its not updated.
* [Mule ESB Runtime v4 documentation](https://mule4-docs.mulesoft.com/)
* MMC and AMC setup are not supported for Mule ESB Community Edition.

---
# About
Developed and maintained by [Reynold Lariza](https://www.linkedin.com/in/relariza/) of [WhiteSky Labs](https://www.whiteskylabs.com), a Premier Partner of [MuleSoft®](https://www.mulesoft.com/) with the mission to be the #1 Certified Partner to deliver specialized API integration services.
