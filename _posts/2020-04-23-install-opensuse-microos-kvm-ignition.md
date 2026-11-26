---
title: "Install OpenSUSE MicroOS in KVM with Ignition"
date: 2020-04-23 21:44:07 +0000
categories: [automation, tutorials]
tags: [opensuse, podman]
header:
  teaser: /assets/uploads/2020/04/featured-image-624x351.png
  image: /assets/uploads/2020/04/featured-image-1536x864.png
---

Install [OpenSUSE MicroOS](https://en.opensuse.org/Kubic:Installation#Installation) in KVM with Ignition. A step-by-step guide to provision container specific OS instances really fast.

## About OpenSUSE MicroOS

I needed a container specific OS since I [converted my docker-compose services to pods with Podman](/2020/02/10/convert-docker-compose-services-to-pods/). Fedora CoreOS looks promising. However I have been using OpenSUSE for years, so it was convenient for me to try [MicroOS](https://en.opensuse.org/Kubic:MicroOS) which is derived from Tumbleweed.

These are the features which I like the most.

- Automatic transactional updates. If you do not like the result, then you can switch back to the previous snapshot. 

- Up to date software versions. Especially for software like [Podman](/2020/03/31/managing-podman-pods-with-pods-compose/) which improves rapidly.

- [A minimal environment specially designed for container workloads.](https://www.youtube.com/watch?v=nIwqzGbX-oc) It has greatly reduced attack surfaces and improved performance.

## Install OpenSUSE MicroOS in KVM with Ignition

There are documentations about MicroOS, but I could not find a complete guide about how to install OpenSUSE MicroOS in KVM. Also its Ignition guide directly redirects to CoreOS's git repo for documentation. The information is there but putting it together takes time.   
As I already managed to do it then why not to share it? :)

In my guide I will use virsh ([libvirt](https://libvirt.org/)) and virt-install ([virt-manager](https://virt-manager.org/)) to provision headless MicroOS VMs based on the downloadable KVM images they made available. Both tools are higher lever APIs to KVM / QEMU.

### Installing libvirt to make using QEMU easy

OpenSUSE has a package pattern for turning your OS into a virtualization host. Following the [Virtualization Guide](https://doc.opensuse.org/documentation/leap/virtualization/html/book.virt/cha-vt-installation.html#sec-vt-installation-kvm) will definitely help. But if you do not want to read it all then just run the following command.

```
zypper in -t pattern kvm_server kvm_tools
```

Should you want to read more about [managing virtual machines with libvirt](https://doc.opensuse.org/documentation/leap/virtualization/html/book.virt/part-virt-libvirt.html) then there is a documentation for that too. I will not go into details this time.

### Downloading and verify installation media of MicroOS

Once the tools are ready then download the installation media and verify the checksum and [signature](https://en.opensuse.org/SDB:Download_help).

```
# cd /var/lib/libvirt/images
# curl -LO http://download.opensuse.org/tumbleweed/appliances/openSUSE-MicroOS.x86_64-ContainerHost-kvm-and-xen.qcow2
# curl -LO http://download.opensuse.org/tumbleweed/appliances/openSUSE-MicroOS.x86_64-ContainerHost-kvm-and-xen.qcow2.sha256
# curl -LO http://download.opensuse.org/tumbleweed/appliances/openSUSE-MicroOS.x86_64-ContainerHost-kvm-and-xen.qcow2.sha256.asc
```

Note that sha256 checksum and signature was made for a snapshot whose name is different from the file we downloaded, though the content is the same. Probably the other files on their webserver are just symbolic links.

```
# sha256sum openSUSE-MicroOS.x86_64-ContainerHost-kvm-and-xen.qcow2
# gpg --search-keys B88B2FD43DBDC284
# gpg --recv-keys 0x22C07BA534178CD02EFE22AAB88B2FD43DBDC284
# gpg --verify openSUSE-MicroOS.x86_64-ContainerHost-kvm-and-xen.qcow2.sha256.asc
```

### Creating Ignition configuration for VM

Ignition expect its configuration to be in JSON. [However one just not create a JSON file by hand](https://docs.fedoraproject.org/en-US/fedora-coreos/producing-ign/). But creates a YML file and convert it with semantic checks (and some boilerplate) to JSON by using CoreOS's [fcct](https://docs.fedoraproject.org/en-US/fedora-coreos/fcct-config/). Here is my example. It is pretty straight forward.

```
variant: fcos
version: 1.0.0
passwd:
  users:
    - name: root
      ssh_authorized_keys:
        - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDHj2D8PAxz0MKV9UJ8dxavlVzdAvMBrfGp38sj4q/aRbkcuYVNHAQh+xXHI0VcPEtu9rqZbvqfmQt0DFhsdf938W6r3y6mLp4+6KIDgb4Jj2B3zlzBIF0haAFi/GZAp4dh4uuhHsVvZGqsdqCglxUnIPb+i8IyYA8GGU+3IOgRfjjtpMfDJcWZTzGm56yDsBYORX3EckkGcWTN4/oW0SKWoO9zf/887/CvVZF/0V7corEAdMyTCiSSqqUjIDLAZpCMU4czadZop7cvVjGT6WLmyGDuTBruvjsMwxYA/OMAZrUuOEoAW0bf/QZRZ4tO7ku+o0oqwca5uwVbuouAFovJ root@example
      password_hash: "$1$salt$qJH7.N4xYta3aEG/dfqo/0"
storage:
  files:
    - path: /etc/sysconfig/network/ifcfg-eth0
      mode: 0600
      overwrite: true
      contents:
        inline: |
          BOOTPROTO='static'
          STARTMODE='auto'
          BROADCAST=''
          ETHTOOL_OPTIONS=''
          IPADDR='192.168.0.10/24'
          MTU=''
          NAME=''
          NETWORK=''
          REMOTE_IPADDR=''
          ZONE=public
    - path: /etc/sysconfig/network/routes
      mode: 0644
      overwrite: true
      contents:
        inline: |
          default 192.168.0.254 - -
    - path: /etc/hostname
      mode: 0644
      overwrite: true
      contents:
        inline: |
          example.com
```

Please note that on OpenSUSE you cannot just provide static DNS information by overwriting /etc/resolv.conf. Because the [content of /etc/resolv.conf is managed by netconfig](https://www.suse.com/c/configure-netconfig-static-domain-search-lists/). And it configuration file bigger than optimal to include it in a YML file. Though you can configure it manually after the first run.

Once you are ready, put the contents into config.fcc and convert it to JSON by using fcct.

```
# podman run -i --rm quay.io/coreos/fcct:release --pretty --strict < config.fcc > config.ign
```

### Create a KVM host with virt-install

The Ignition file can be specified via QEMU command line. Adjust the specification of the VM according to your needs.

```
# virt-install --import --connect qemu:///system --name example \
    --ram 1024 --vcpus 1 \
    --disk size=20,backing_store=/var/lib/libvirt/images/openSUSE-MicroOS.x86_64-ContainerHost-kvm-and-xen.qcow2,bus=virtio \
    --os-variant=opensusetumbleweed \
    --network bridge=br0,model=virtio \
    --noautoconsole \
    --graphics spice,listen=127.0.0.1 \
    --qemu-commandline="-fw_cfg name=opt/com.coreos/config,file=/path/to/config.ign"
```

No you have just installed OpenSUSE<span class="has-inline-color has-blue-color"> MicroOS in KVM with Ignition. </span>Enjoy.

## Managing VM with Virsh

Virsh is another tool to manage your VMs. Here I provide some examples I frequently use. They could be useful.

### Console access

You can attach to the serial console of the VM with the following command.

```
# virsh console example
```

[Disconnecting from Virsh console](https://www.cyberciti.biz/faq/howto-exit-virsh-console-connection-on-centos-ubuntu-debian-linux/) is possible with SHIFT+5.

### Remote console access

Remote access to the console is possible with [Spice](https://www.spice-space.org/index.html). If you are not in production then you can easily [access the remote Spice port without TLS](https://unix.stackexchange.com/questions/148794/how-to-create-kvm-guest-with-spice-graphics-but-tls-disabled-using-virt-install) via SSH port forward. 

### Delete virtual machines

This can be useful when you are not satisfied with the result and want to start over from scratch. [Storage files are not automatically deleted](https://www.cyberciti.biz/faq/howto-linux-delete-a-running-vm-guest-on-kvm/).

```
# virsh dumpxml --domain example | grep 'source file'
      <source file='/var/lib/libvirt/images/example.qcow2'/>
        <source file='/var/lib/libvirt/images/openSUSE-MicroOS.x86_64-ContainerHost-kvm-and-xen.qcow2'/>
# virsh destroy example
# virsh undefine example
# rm -f /var/lib/libvirt/images/example.qcow2
```

I hope you will enjoy your shiny new container host. :)
