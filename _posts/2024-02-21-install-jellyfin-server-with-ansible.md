---
title: "Install Jellyfin server with Ansible"
date: 2024-02-21 21:27:11 +0000
categories: [automation, entertainment]
tags: [ansible]
header:
  teaser: /assets/uploads/2024/02/cute-jellyfin-sleeps-on-a-computer-keyboard-feature-image-768x568.jpeg
  image: /assets/uploads/2024/02/cute-jellyfin-sleeps-on-a-computer-keyboard-feature-image.jpeg
---

Install Jellyfin server with Ansible to set up your own media streaming service in an automated manner. Download the role to install your own media server. This post does not explain Jellyfin itself, it is only about to install it with Ansible.

## Why install Jellyfin server with Ansible?

I have a plan to rebuild all my self hosted services in an automated way by using Ansible. I looked for existing roles to install Jellyfin server with Ansible but most of them were not updated for a while or clearly relied on APT package manager. There is nothing wrong in using a Debian package repository, but I use OpenSUSE and at the time of writing this, there is no package repository for OpenSUSE. So I sticked to the [generic amd64 installer.](https://jellyfin.org/docs/general/installation/linux#linux-generic-amd64)

The role installs and starts the generic amd64 package on Linux. Sets up a systemd service which starts jellyfin automatically on boot.

## Why a generic package and not Docker or Kubernetes?

That server I use is still [running Home Assistant on Kubernetes](/2023/06/15/running-home-assistant-on-kubernetes/). But passing tebibytes of data through some additional level of storage layer just to be able to connect it to a Jellyfin pod does not worth the effort at least performance wise.

## Automate the automation

I wanted to publish the role on [Ansible Galaxy](https://galaxy.ansible.com/ui/standalone/roles/abalage/jellyfin/) but also in an automated way by using GitHub Actions. I've been using GitLab for a while so I already familiar with the concept of CI/CD pipelines. I only needed to get around the syntax GitHub uses.

### Tasks I automated by using GitHub Actions

1. First of all there are lint tests by using [yamllint](https://pypi.org/project/yamllint/) and [ansible-lint](https://pypi.org/project/ansible-lint/) to look for possible issues introduced by typos or bad usage patterns. That's really basic but are very helpful.

1. I utilize [Molecule](https://pypi.org/project/molecule/) framework to test the Ansible role itself. Molecule itself is not a static code analysis tool like yamllint. It tests all tasks by actually executing them in a sandbox environment. You can even spot idempotent issues for example.
It is always a best practice to make sure you write idempotent tasks.
However I admit that sometimes fixing those are either impossible or simply does not worth the effort.

1. The third chunk of the workflow is to upload a release of the role once a commit is tagged in git. This makes releasing easy and less prone to errors.

I remember I used to release manually when I developed an Ansible collection to [deploy Elasticsearch stack with podman and ansible](/2021/12/06/deploy-elasticsearch-stack-with-podman-and-ansible/). But that's the past.

## How to use the Ansible role

I assume you already have an [Ansible inventory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html) file.
You need a playbook to invoke the role. The sample [playbook.yml](https://github.com/abalage/ansible-role-jellyfin?tab=readme-ov-file#example-playbook) I provided next to the [abalage.jellyfin](https://github.com/abalage/ansible-role-jellyfin) role works just fine.

You also need a **roles/requirements.yaml** file to contain the source of the role. Get the latest and greatest version directly from the git repo like this.

```
roles:
  - name: abalage.jellyfin
    src: https://github.com/abalage/ansible-role-jellyfin.git
    version: main
```

Or fetch the versioned role from Ansible Galaxy.

```
roles:
  - name: abalage.jellyfin
    version: 1.0.1
```

Once that is ready you can get it by using ansible-galaxy and install it by using ansible-playbook.

```
$ansible-galaxy role install --force -r roles/requirements.yml
$ ansible-playbook -i inventory.ini playbook.yml
```

The role is open source. Feel free to use it.
Should you any feedback please leave in the comments below.

Disclaimer: the featured image in this post was created by AI.
