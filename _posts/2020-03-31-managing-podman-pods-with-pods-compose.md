---
title: "Managing Podman pods with pods-compose"
date: 2020-03-31 22:08:15 +0000
categories: [script, tutorials]
tags: [linux, podman, python]
header:
  teaser: /assets/uploads/2020/03/piano-1655558_1920-624x416.jpg
  image: /assets/uploads/2020/03/piano-1655558_1920-1536x1024.jpg
---

Managing Podman pods with pods-compose makes your move to Podman easier. I already [converted my docker-compose services to pods with Podman](/2020/02/10/convert-docker-compose-services-to-pods/), however I really missed some features, up until now. Let's meet [pods-compose](https://github.com/abalage/pods-compose).

## Missing features of Podman

As soon as I managed to [convert docker-compose services to pods](/2020/02/10/convert-docker-compose-services-to-pods/) I realized that managing them will be not as easy as it was with docker-compose. I faced with the following problems.

1. **There is no autostart for pods and containers.** Of course you can generate systemd service units for all components but managing them is not easy without an automation tool. There are separate service files for each pod and container.

1. **For updates you might have to stop, remove, recreate everything from scratch, unless you script it.** There is no '*up*', '*down*' or '*build*' features like you have with docker-compose.

1. **There is no single point of configuration which I could use to describe all the pods and containers.** I could write and maintain Kubernetes YAML files, but that's even harder than using the CLI syntax I am already familiar with.

I needed a tool which makes managing Podman pods easier. [podman-compose](https://github.com/containers/podman-compose) looked promising but it did not really work for me and I also did not like its CLI interface. So I decided to write [my own tool](https://github.com/abalage/pods-compose).

## Design goals of pods-compose

I did not want to put a lot of effort into this. I only wanted the following additional abilities.

- Be able to automatically start and stop all pods and containers upon reboot.

- Tear down existing pods at once.

- Create pods and containers from a description at once.

- Build all the images I define with a single command.

I did not want to rely on docker-compose's YAML format. Intentionally there is no support for using an existing compose configuration. Although I was already familiar with that format, I wanted a complete migration not just a partial one.

## Managing Podman pods with pods-compose

As a kickstart, let's get a glimpse into the similarities between pods-compose and docker-compose.

|  | **pods-compose** | **docker-compose** |
| --- | --- | --- |
| Deploy pod(s) | --up \[POD\] | up \[SERVICE\] |
| Tear down pod(s) | --down \[POD\] | down \[SERVICE\] |
| Start pod(s) | --start \[POD\] | start \[SERVICE\] |
| Stop pod(s) | --stop \[POD\] | stop \[SERVICE\] |
| Restart pod(s) | --restart \[POD\] | restart \[SERVICE\] |
| Build all container images | --build | build |
| Status of pods and containers | --ps | ps |
| Generate Kubernetes Pod YAML(s) | --generate |  |

## Autostart pods and containers

Podman can generate systemd units for pods and containers. However there will be many of them, making it hard to overview and maintain it. Because pods-compose takes care of starting and stopping pods with a single command line option, I could create a single systemd service file instead of many.

The install script will deploy that systemd service file for pods-compose. Enabling it makes your pods and containers to start automatically upon reboot. And of course gracefully stop before the system halts.

Further details can be found in the [readme of pods-compose](https://github.com/abalage/pods-compose/blob/master/README.md).

## Things you still have to do manually

Creating containers at least first time is a manual procedure. People usually start with 'docker run' commands then once the result looks okay then will create a docker-compose YAML.

This will not change with pods-compose. You still have to create your pods and containers with 'podman run'. **However you do not have to create any YAML files.** The tool will create them for you.   
Luckily [podman CLI syntax is almost the same as docker's](https://podman.io/whatis.html), so it is easy to make progress fast.

The other part is defining which image should be built by pods-compose. Because this information cannot be set in Kubernetes YAML files, you can use [pods-compose](https://github.com/abalage/pods-compose)'s INI formatted configuration file to define the TAG and the CONTEXT of images. As a result, pods-compose will build all the images for you.

## Final words

Let me know if you are still missing some features you would love to see implemented in pods-compose. Also please share if you liked it.
