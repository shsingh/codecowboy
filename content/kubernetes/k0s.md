+++
title = "k0s - small kubernetes - part 1"
date = "2022-02-10"
aliases = ["k0s"]
tags = ["kubernetes", "containers", "micro"]
categories = ["kubernetes", "software", "dev"]
[ author ]
  name = "codecowboy.io"
+++

## Intro
I recently sat down and thought to myself "I wonder how the current market of mini or micro kubernetes distributions is going". I've been using minikube for a bit and thought to myself it was about time to start using a few other distributions.

So I have been experimenting with different micro or mini distributions. 

This post is about **k0s**.

## k0s
**k0s** is an open source project that you can find here: [https://k0sproject.io/](https://k0sproject.io/)

**k0s** looks like it's sponsored by Mirantis. They offer commercial support and so on if you want it too. In and of itself, **k0s** is a free and open source project. You can use it and it works without paying anyone :)

## Install
Installation was relatively easy for the most part. It's possible to install both a single node cluster and a multi node cluster. I'll step through both of these.

### Single Node Cluster
A single node cluster is perhaps the easiest to install and the fastest way to get up and running.

```Bash
curl -sSLf https://get.k0s.sh | sudo sh
```
This will install the k0s binary in **/usr/local/bin/**.

```Bash
sudo k0s install controller --single
```
Next install the cluster as a single node cluster.

You can check the status of your freshly installed cluster using the k0s command.

```Bash
[root@kube ~]# k0s status
Version: v1.23.3+k0s.0
Process ID: 1313
Role: controller
Workloads: true
SingleNode: true
```

It's that simple - from this point on, you have a single node cluster that can be used as normal.



### Multi Node Cluster
To install a multi node cluster, it's best to use a second binary called **k0sctl**. This can be downloaded and installed in the following way.

```Bash
curl -L https://github.com/k0sproject/k0sctl/releases/download/v0.13.0-beta.1/k0sctl-linux-x64 -o k0sctl
```

Once you have the **k0sctl** binary installed, you can then use it to create a default cluster bootstrap yaml file.

```Bash
k0sctl init > k0sctl.yaml
```

This will create a file called k0sctl.yaml that has the following contents. The file below has IP addresses that are relevant to my setup. Each node must be reachable via passwordless ssh from the node where **k0sctl** is installed. See the note below for more on this.

```Yaml
apiVersion: k0sctl.k0sproject.io/v1beta1
kind: Cluster
metadata:
  name: k0s-cluster
spec:
  hosts:
  - ssh:
      address: 10.1.1.160
      user: root
      port: 22
      keyPath: /root/.ssh/id_rsa
    role: controller
  - ssh:
      address: 10.1.1.170
      user: root
      port: 22
      keyPath: /root/.ssh/id_rsa
    role: worker
  k0s:
    version: 1.23.3+k0s.0
    dynamicConfig: false
```

If your SSH is configured correctly to all of your nodes, you should be able to run the following command:

```Bash
k0sctl apply --config ./k0sctl.yaml
```

This will perform a multi cluster installation on the two nodes specified. The output will look something like this:

```Bash

⠀⣿⣿⡇⠀⠀⢀⣴⣾⣿⠟⠁⢸⣿⣿⣿⣿⣿⣿⣿⡿⠛⠁⠀⢸⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠀█████████ █████████ ███
⠀⣿⣿⡇⣠⣶⣿⡿⠋⠀⠀⠀⢸⣿⡇⠀⠀⠀⣠⠀⠀⢀⣠⡆⢸⣿⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀███          ███    ███
⠀⣿⣿⣿⣿⣟⠋⠀⠀⠀⠀⠀⢸⣿⡇⠀⢰⣾⣿⠀⠀⣿⣿⡇⢸⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⠀███          ███    ███
⠀⣿⣿⡏⠻⣿⣷⣤⡀⠀⠀⠀⠸⠛⠁⠀⠸⠋⠁⠀⠀⣿⣿⡇⠈⠉⠉⠉⠉⠉⠉⠉⠉⢹⣿⣿⠀███          ███    ███
⠀⣿⣿⡇⠀⠀⠙⢿⣿⣦⣀⠀⠀⠀⣠⣶⣶⣶⣶⣶⣶⣿⣿⡇⢰⣶⣶⣶⣶⣶⣶⣶⣶⣾⣿⣿⠀█████████    ███    ██████████

k0sctl v0.13.0-beta.1 Copyright 2021, k0sctl authors.
Anonymized telemetry of usage will be sent to the authors.
By continuing to use k0sctl you agree to these terms:
https://k0sproject.io/licenses/eula
INFO ==> Running phase: Connect to hosts
INFO [ssh] 10.1.1.160:22: connected
INFO [ssh] 10.1.1.170:22: connected
INFO ==> Running phase: Detect host operating systems
INFO [ssh] 10.1.1.160:22: is running Fedora Linux 35 (Server Edition)
INFO [ssh] 10.1.1.170:22: is running Fedora Linux 35 (Server Edition)
INFO ==> Running phase: Prepare hosts
INFO ==> Running phase: Gather host facts
INFO [ssh] 10.1.1.160:22: using kube as hostname
INFO [ssh] 10.1.1.170:22: using kube-worker as hostname
INFO [ssh] 10.1.1.160:22: discovered ens224 as private interface
INFO [ssh] 10.1.1.170:22: discovered ens224 as private interface
INFO [ssh] 10.1.1.160:22: discovered 192.168.141.131 as private address
INFO [ssh] 10.1.1.170:22: discovered 192.168.141.132 as private address
INFO ==> Running phase: Validate hosts
INFO ==> Running phase: Gather k0s facts
INFO ==> Running phase: Validate facts
INFO ==> Running phase: Download k0s on hosts
INFO [ssh] 10.1.1.170:22: downloading k0s 1.23.3+k0s.0
INFO ==> Running phase: Configure k0s
WARN [ssh] 10.1.1.160:22: generating default configuration
INFO [ssh] 10.1.1.160:22: validating configuration
INFO [ssh] 10.1.1.160:22: configuration was changed
INFO ==> Running phase: Initialize the k0s cluster
INFO [ssh] 10.1.1.160:22: installing k0s controller
INFO [ssh] 10.1.1.160:22: waiting for the k0s service to start
INFO [ssh] 10.1.1.160:22: waiting for kubernetes api to respond
INFO ==> Running phase: Install workers
INFO [ssh] 10.1.1.170:22: validating api connection to https://192.168.141.131:6443
INFO [ssh] 10.1.1.160:22: generating token
INFO [ssh] 10.1.1.170:22: writing join token
INFO [ssh] 10.1.1.170:22: installing k0s worker
INFO [ssh] 10.1.1.170:22: starting service
INFO [ssh] 10.1.1.170:22: waiting for node to become ready
INFO ==> Running phase: Disconnect from hosts
INFO ==> Finished in 2m9s
INFO k0s cluster version 1.23.3+k0s.0 is now installed
INFO Tip: To access the cluster you can now fetch the admin kubeconfig using:
INFO      k0sctl kubeconfig
```

#### A Note on SSH

## Deployment

## Ingress

## Conclusion
I was very pleasantly surprised by **k0s** it was easy to deploy and get up and running in either a single node cluster or a multi node cluster. The documentation was very good, and **it just worked**. This is perhaps the most important part for me. As a developer, I want to have the ability to get up and running super fast, without a lot of fuss to be able to try out different configurations or deploy applications.