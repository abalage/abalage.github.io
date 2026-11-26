---
title: "Running Home Assistant on Kubernetes"
date: 2023-06-15 21:33:24 +0000
categories: [automation, kubernetes, security]
tags: [ansible, diy_home_server, kubernetes]
header:
  teaser: /assets/uploads/2023/06/home-assistant-on-kubernetes-624x390.jpg
  image: /assets/uploads/2023/06/home-assistant-on-kubernetes-1536x960.jpg
---

Running Home Assistant on Kubernetes. It is about time to put your IoT devices under control. Everything runs off the cloud on my on-premise Kubernetes. Clean and secure.

This blog will not cover what [Home Assistant](https://www.home-assistant.io) is or [how Kubernetes works](https://kubernetes.io/docs/concepts/). I assume you already have that covered.

## Separate network traffic for IoT devices

I created a separate VLAN just for IoT devices on my OpenWRT. Some things to notice about this VLAN.

- A separate AP exists for devices communicating over WiFi.

- This VLAN does have access neither to other networks nor to the Internet. So no IoT device will be able to [phone home](/2019/04/27/android-app-phones-home-china/). And most probably you cannot use the official Apps.

- Kubernets has a dedicated [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/) listening in this VLAN, opposite to the [Traefik Ingress Controller](https://docs.k3s.io/networking#traefik-ingress-controller) which listens in the default network.

- The host running Kubernetes does have a network connection in both VLANs (optional).

If you would like to create VLANs but don't know how to start I highly recommend watching Mark's great video about [VLANs in OpenWrt 21](https://www.youtube.com/watch?v=qeuZqRqH-ug). It will give you a head start.

## Use K3s lightweight Kubernetes distribution for IoT

Although there are many Kubernetes distributions to choose from, I have picked [K3s](https://k3s.io/).

It is a lightweight Kubernetes and it is proven to work on a bunch of ~~inexpensive~~ unavailable Raspberry Pi4s running in a basement just like [Jeff's Pi Dramble](https://www.jeffgeerling.com/project/raspberry-pi-dramble). Make sure to check him out, it is unbelievable. :)

## The Deployment problem of Home Assistant

There are a couple of supported ways of [installing Home Assistant](https://www.home-assistant.io/installation/). Unfortunately Kubernetes is not among them. I do not blame them. Probably it simply does not worth the effort to support it, because people using Home Assistant is already a niche market, even more people running Kubernetes at home.

There are 4 options to install it.

- Home Assistant Operating System - their own operating system based on Docker (no go)

- Home Assistant Container - A container image containing only the core of Home Assistant (we'll use this)

- Home Assistant Supervised - *"This way of running Home Assistant will require the most of you. It also has strict requirements you need to follow."* (still requires Docker's socket - no go)

- Home Assistant Core - Same as *Home Assistant Container* but without the container (no go)

The biggest drawback of using their container image is that there will be no "[Addons store](https://www.home-assistant.io/addons/)" feature. And you will need to find the necessary howtos in forums, github issues, etc.

So I will rely on their container image but what about deployment descriptions?

I cannot really use their docker-compose.yml in Kubernetes for other purposes than as a base for creating my own manifests.

There were Helm charts available for some time but at the time of writing this article, none of them are maintained anymore but at least they are better source for creating something new than using the docker-compose YMLs. I decided to roll my own version of manifest and keeping things as simple as possible.

## Installing Home Assistant with Kustomize

So I have the official container image but I will need services other than the plain Core of Home Assistant. I need the following functionalities.

- Home Assistant itself. You can grab the Kubernetes manifest from [abalage/home-assistant-k8s](https://github.com/abalage/home-assistant-k8s).

- MQTT support ([Mosquitto](https://mosquitto.org/)) which is a frequently required integration interface for lots of IoT devices. You can download the Kubernetes manifest from [abalage/mosquitto-mqtt-k8s](https://github.com/abalage/mosquitto-mqtt-k8s).

- [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/) to expose HTTP(s) and MQTT's TCP port 1883 (later one is not HTTP protocol)

- [MetalLB](https://github.com/metallb/metallb) for acquiring an externally addressable IP address for Home Assistant's service (LoadBalancer)

Like I mentioned before the Helm charts are outdated and I needed some customization anyway,  hence I use [Kustomize](https://kustomize.io/) to get all the dependencies properly configured and patched to fit my network environment.

**You do not need to individually download the manifests. Just check out the GitHub project [abalage/Home-Assistant-on-Kubernetes](https://github.com/abalage/Home-Assistant-on-Kubernetes) which contains everything you need to start.**

Make sure you edit \`kustomization.yaml\` to fit to your environment. Check out the README for details.

Now you can run your own Home Assistant on Kubernetes.

What do you think? Isn't it awesome? :)
