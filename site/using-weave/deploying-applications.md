---
title: Deploying Applications To Weave Net
layout: default
---

This section contains the following topics: 

 * [Launching Weave Net](#launching)
 * [Creating Peer Connections Between Hosts](#peer-connections)
 * [Testing Container Communications](#testing)
 * [Starting the Netcat Service](#start-netcat)


###<a name="launching"></a>Launching Weave Net

Before launching Weave and deploying your apps, ensure that Docker is [installed]( https://docs.docker.com/engine/installation/) on both hosts. 

On `$HOST1` run:

    host1$ weave launch
    host1$ eval $(weave env)
    host1$ docker run --name a1 -ti ubuntu

Where, 

 * The first line runs Weave. 
 * The second line configures the Weave environment, so that containers launched via the Docker command line are automatically attached to the Weave network, and, 
 * The third line runs the application container using [Docker commands]( https://docs.Docker.com/engine/reference/commandline/daemon/). 

>>**Note:** If the first command results in an error like
 `http:///var/run/Docker.sock/v1.19/containers/create: dial unix
 /var/run/Docker.sock: permission denied. Are you trying to connect to a TLS-enabled daemon without TLS?` then you likely need to be 'root' in order to connect to the Docker daemon. If so, run the above and all subsequent commands in a *single* root shell (e.g. one created with `sudo -s`). Do *not* prefix individual commands with `sudo`, since some commands modify environment entries and hence they all need to be executed from the same shell.

>>**Important!** if you are running the Weave Docker Network Plugin do not run `eval $(weave env)`. See [Using the Weave Net Docker Network Plugin](/site/pluing/weave-plugin-how-to.md) for more information.

Weave must be launched once per host. The relevant container images will be pulled down from Docker Hub on demand during `weave launch`. 

You can also preload the images by running `weave setup`. Preloaded images are useful for automated deployments, and ensure there are no delays during later operations.

If you are deploying an application that consists of more than one container to the same host, launch them one after another using `docker run`, as appropriate.  


###<a name="peer-connections"></a>Creating Peer Connections Between Hosts

To launch Weave on an additional host and create a peer connection, run the following:

    host2$ weave launch $HOST1
    host2$ eval $(weave env)
    host2$ docker run --name a2 -ti ubuntu

As noted above, the same steps are repeated for `$HOST2`. The only difference, besides the application container’s name, is that `$HOST2` is told to peer with Weave on `$HOST1` during launch. 

You can also peer with other hosts by specifying the IP address, and a `:port` by which `$HOST2` can reach `$HOST1`. 

>>**Note:** If there is a firewall between `$HOST1` and `$HOST2`,  you must permit traffic to flow through TCP 6783 and UDP 6783/6798, which are Weave’s control nd data ports.

There are a number of different ways that you can specify peers on a Weave network. You can launch Weave on `$HOST1` and then peer with `$HOST2`, or you can launch on `$HOST2` and peer with `$HOST1` or you can tell both hosts about each other at launch. The order in which peers are specified is not important. Weave automatically (re)connects to peers when they become available. 

####Specifying Multiple Peers at Launch

To specify multiple peers, supply a list of addresses to which you want to connect, all separated by spaces. 

For example: 
~~~bash
    `host2$:weave launch` <ip address> <ip address> 
~~~

Peers can also be dynamically added. See [Adding Hosts Dynamically](/site/using-weave/finding-adding-hosts-dynamically.md) for more information.


###<a name="testing"></a>Testing Container Communications

With two containers running on separate hosts, test that both containers are able to find and communicate with one another using ping. 

From the container started on `$HOST1`...

    root@a1:/# ping -c 1 -q a2
    PING a2.weave.local (10.40.0.2) 56(84) bytes of data.
    --- a2.weave.local ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.341/0.341/0.341/0.000 ms

Similarly, in the container started on `$HOST2`...

    root@a2:/# ping -c 1 -q a1
    PING a1.weave.local (10.32.0.2) 56(84) bytes of data.
    --- a1.weave.local ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 0.366/0.366/0.366/0.000 ms

###<a name="start-netcat"></a>Starting the Netcat Service

The `netcat` service can be started using the following commands:  

~~~bash
    root@a1:/# nc -lk -p 4422
~~~

and then connected to from the another container on `$HOST2` using:

~~~bash
    root@a2:/# echo 'Hello, world.' | nc a1 4422
~~~

Weave supports *any* protocol, and it doesn't have to be over TCP/IP. For example, a netcat UDP service can also be run by using the following:

~~~bash
    root@a1:/# nc -lu -p 5533
    root@a2:/# echo 'Hello, world.' | nc -u a1 5533
~~~


**See Also** 

 * [Installing Weave Net](/site/installing-weave.md)
 * [Using Fastdp With Weave](/site/fastdp/using-fastdp.md)
 * [Using the Weave Net Docker Network Plugin](/site/plugin/weave-plugin-how-to.md)