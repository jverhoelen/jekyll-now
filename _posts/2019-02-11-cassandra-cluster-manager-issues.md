---
layout: post
title: "Problem Solutions and Tricks for Cassandra Cluster Manager"
category: "Java"
---

[Cassandra Cluster Manager (CCM)](https://github.com/riptano/ccm) is a helpful tool for running local Cassandra clusters. Especially if you need some exotic configuration, DataStax Enterprise clusters or simply don't want to use Docker. CCM is a command line based Python application that lets you download, maintain and operate such clusters. Unfortunately it is not very robust and prone to errors on different environments. In addition in inherits problems from Cassandra itself since it's just functioning as a wrapper around Cassandra source code.

## CCM and Cassandra Can Not Handle New Java Versions

This issue is a similar one to the issues people are facing with DataStax DevCenter and new Java versions. A solution to that was [described in an earlier blog post](https://jonas.verhoelen.de/java/datastax-devcenter-freezes-wrong-java-version/) of me. The problem starts to occurr when updating your Java 8 installation. For me the earliest occurrence of this issue appeared in `8.0.172`.

The solution is easy again. An older version of Java needs to be provided. To handle different Java versions on your computer, the SDKMAN! version manager for a bunch of SDKs is recommended. To keep DevCenter and CCM alive on my computer, I found cure in Java version `7.0.181`. Using SDKMAN! it can be installed with `sdk install java 7.0.181-zulu`.

## Multi-Node-Clusters and Loopback Addresses

When working with multi-node-clusters, each Cassandra node needs a separate local IP address. By default, a three nodes cluster needs the loopback addresses `127.0.0.{1,2,3}`. All addresses except `127.0.0.1` need to be configured after every computer reboot:

```bash
sudo ifconfig lo0 alias 127.0.0.2  
sudo ifconfig lo0 alias 127.0.0.3
```

To avoid forgetting this it is helpful to put everything into a function of whatever Shell is used. This is how it could look for Zshell:

```bash
function ccmstart() {
  sudo ifconfig lo0 alias 127.0.0.2
  sudo ifconfig lo0 alias 127.0.0.3
  ccm start
  ccm status
}
```

Now you just need to remember using `ccmstart` when starting your cluster after a reboot.

## Corrupted Nodes Due to Sudden Shutdown

In case of a sudden computer shutdown, Cassandra can get consistency problems with its current state of the cluster and its commit logs. Commit logs are basically one point to which Cassandra writes changes. So the commit logs can be corrupted. In this situation it helps to delete the commit logs of all nodes of the cluster.

```bash
rm .ccm/my-cluster/*/commitlogs/**
```

## CCM Never Tells What's Wrong

No matter which problem CCM or the Cassandra nodes have, they do most probably not tell you directly what it is. Therefor it is essential to know the directory structure CCM is using to dig into it.

* `~/.ccm/my-cluster/` contains everything for one cluster
* `~/.ccm/my-cluster/node1/logs/` contains all kinds of logs for a node. `system.log` and `debug.log` are always worth a try.
* `~/.ccm/my-cluster/node1/commitlogs/` contains the commit logs for a node (see previous section)
* `~/.ccm/repository/` caches all downloaded Cassandra versions

This will make it possible to quickly spot all funny combinations of problems that Cassandra and CCM are having sometimes. Good luck!
