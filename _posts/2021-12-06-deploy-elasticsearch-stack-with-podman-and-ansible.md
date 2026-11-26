---
title: "Deploy Elasticsearch stack with podman and Ansible"
date: 2021-12-06 22:03:43 +0000
categories: [automation, script]
tags: [ansible, elasticsearch, kibana, podman]
header:
  teaser: /assets/uploads/2021/12/ansible-elastic-podman-background-624x351.jpg
  image: /assets/uploads/2021/12/ansible-elastic-podman-background-1536x864.jpg
---

Deploy Elasticsearch stack with podman and Ansible. Halfway on the road towards complete automation. But without the necessity of a complex orchestration tool. Somewhere between [pets and cattles](https://www.hava.io/blog/cattle-vs-pets-devops-explained).

There is an existing Ansible collection [containers.podman](https://galaxy.ansible.com/containers/podman) to handle podman pods and containers. Although Elastic the company already maintains an [Ansible playbook for Elasticsearch](https://github.com/elastic/ansible-elasticsearch), it uses regular Linux packages and not container images. Meet [abalage.elasticstack\_podman](https://galaxy.ansible.com/abalage/elasticstack_podman) a collection of Ansible roles to deploy and handle an Elasticsearch cluster and its components like Kibana, Filebeat, Metricbeat and Logstash.

You can download it from [galaxy.ansible.com](https://galaxy.ansible.com/abalage/elasticstack_podman) or from [github](https://github.com/abalage/ansible_collection_elasticstack_podman).

## Requirements

Any operating system which supports a relatively recent version of podman (\>=3.0) is required. Beware that CentOS 7 is not among them. The playbooks were tested on AlmaLinux 8.4 and OpenSUSE Leap 15.3. However on OpenSUSE you need to use a third party repository ([Virtualization\_containers](https://build.opensuse.org/project/show/Virtualization:containers)).

The collection does not contain a reverse proxy for Kibana. You can use either Traefik of NGINX. The Kibana container is [already provides labels](https://github.com/abalage/ansible_collection_elasticstack_podman/blob/e9bedff52dd5c49c58d97859d2d3361297f2e034/roles/kibana/tasks/kibana_podman.yml#L43) for Traefik.

## Features

I implemented the following features.

- It deploys an Elasticsearch cluster. Works with single node deployments. However you can build a cluster of multiple nodes as well. You can even run multiple nodes on the same host OS.

- Use Kibana for visualization.

- Metricbeat automatically collects and stores all components metrics in the cluster. Use Kibana's *Stack Monitoring* app to access the metrics.

- Filebeat sends the components logs to Elasticsearch. Use Kibana's *Logs* app to access the logs.

- Optionally you can set up Logstash containers too. Although there are not many pipeline templates available.

- Automatically populates built-in and custom users, passwords and roles. It does not support AD integration yet.

- Pods and containers are automatically started upon reboot by using systemd units.

- Supports host firewalld. Disabled by default.

- Works best with [host networking](https://podman.io/getting-started/network.html). Support for bridge networking is best effort and has scalability limitations. It does not support rootless networking at the moment.

## Usage of the collection

I expect you already have an Ansible control node and several managed hosts. The collection was developed and tested with Ansible 2.9.

### Create your deployment playbook

A playbook defines which play, roles and tasks of the collection are executed on which hosts. There is an existing playbook called [elk-podman-deployment](https://github.com/abalage/elk-podman-deployment) you can use. For example there is an [example playbook](https://github.com/abalage/ansible_collection_elasticstack_podman/blob/main/example/playbook/playbook.yml) in the repository too.

### Create your deployment inventory

The deployment inventory describes how your cluster looks like. You can use the variables from the role's defaults to create an inventory form scratch. However I provide a [example inventory](https://github.com/abalage/ansible_collection_elasticstack_podman/blob/main/example/playbook/playbook.yml) that you can customize.

Do not forget to encrypt sensitive data with ansible-vault.

I highly recommend to create proper X.509 certificates for TLS for security reasons. Make sure to follow the [Securing Elasticsearch cluster](https://github.com/abalage/ansible_collection_elasticstack_podman#securing-elasticsearch-cluster) guide to create such certificates.

### Run the playbook

Once the inventory is complete, you can run the playbook like this tot deploy Elasticsearch stack with podman and Ansible.

```
$ ansible-playbook -i /path/to/production.ini playbook.yml --vault-password-file /path/to/vault-secret 
```

It is a good idea to run in check mode on the first run to see whether is there anything missing from the inventory.

### Reverse proxy

The collection itself does not provide any reverse proxy.

You can use any kind of reverse proxy to provide access to Kibana or any other components. I suggest to use Traefik for auto-discovery.

## Conclusion

Developing all these roles and task were fun. I could learn a lot about Ansible. Therefore I can recommend this collection to anyone who would need such a setup but without the requirement of having a complex orchestration platform. I am aware of production systems deployed by this playbook.

However. I think this approach on the long run is not feasible. The architecture can grow to became uncontrolled pretty easily, unless someone constantly maintains the collection and provides support.

I could think of better alternatives like incorporating the container parts into Elastic's official Ansible playbook. So the support would come from the vendor and not from the community. It might also worth to try some Edge/IoT oriented Kubernetes distribution like K3s which is lightweight but also supports [Helm charts or better Operators](https://cloud.redhat.com/blog/build-kubernetes-operators-from-helm-charts-in-5-steps).

What do you think?
