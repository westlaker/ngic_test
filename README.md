ARM64
=====
install docker 

`sudo apt-get install docker.io`

install docker server version 1.3.1

```shell
 Containers: 26
 Running: 1
 Paused: 0
 Stopped: 25
 Images: 308
 Server Version: 1.13.1
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins: 
  Volume: local
  Network: bridge host macvlan null overlay
 Swarm: inactive
 Runtimes: runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version:  (expected: aa8187dbd3b7ad67d8e5e3a15115d3eef43a7ed1)
 runc version: N/A (expected: 9df8b306d01f59d3a8029be411de015b7304dd8f)
 init version: N/A (expected: 949e6facb77383876aeff8a6944dde66b3089574)
 Security Options:
  seccomp
   Profile: default
 Kernel Version: 4.14.0-dirty
 Operating System: Ubuntu 16.04.3 LTS
 OSType: linux
 Architecture: aarch64
 CPUs: 4
 Total Memory: 15.65 GiB
 Name: localhost.localdomain
 ID: 4EWW:GPA7:BIZB:DLXJ:LW27:4F5R:F5MR:7ROJ:VPDU:6L2J:TOBI:KDL5
 Docker Root Dir: /var/lib/docker
 Debug Mode (client): false
 Debug Mode (server): false
 Registry: https://index.docker.io/v1/
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```
Install docker-compose

`sudo apt-get install python-setuptools python-dev` 

`sudo pip install docker-compose --force --upgrade`

`sudo docker-compose version`

```shell
Docker-compose version 1.20.1, build 5d8c71b
docker-py version: 3.2.1
CPython version: 2.7.12
OpenSSL version: OpenSSL 1.0.2g  1 Mar 2016
st version of docker-compose supported is 3.1
```


NOTES
=====

Tested on MacchiatoBin and ThunderxV1


Prerequisites
=============

This demo requires a Linux box with at least 4GB of free RAM.

Install [docker](https://docs.docker.com/engine/installation/) (at least version 1.13 to run only) and [docker-compose](https://github.com/docker/compose/releases).

After Docker is setup, clone the source code in this repo:

`git clone https://github.com/adivjoseph/ngic_test.git`

`git submodule update --recursive --remote`



Build Docker images
===================

Arm64 images must be in Docker cache, see README.md in dockerfiles directory on how to build them

Then open 2 additional terminals and change directory from the demo folder in all of them:

`cd ngic_test`

Link to build instructions [ARM64 ONLY](https://github.com/adivjoseph/ngic_test/blob/master/dockerfiles/README.md#ARM64-ONLY)




Starting the NGIC Demo
=======================

```

In terminal #1: Bring up DP

`docker-compose -p epc up dp`

Wait until you see messages about `Unable to resolve ...`, which means that it is waiting for the control plane.


Starting the Control Plane
==========================

In terminal #2: Bring up CP after DP is ready

`docker-compose -p epc up cp`

Wait until you see messages about `Unable to resolve mme.s11.ngic.`, which means that it is waiting for the traffic container.


Starting the Traffic 
====================

In terminal #3: Bring up Traffic container in daemon mode (-d) with:

`docker-compose -p epc up -d traffic`

Then enter the Traffic container with:

`docker exec -it epc_traffic_1 /bin/bash`

We need to rewrite the pre-recorded traffic to have the correct IP addresses. This will generate 3 modified pcap files. (This will take several seconds.)

`./rewrite_pcaps.py enb.s1u.ngic spgw.s11.ngic spgw.s1u.ngic spgw.sgi.ngic`


Now that the address fields are updated, it's time to start the traffic.
First, get the interface names by running the following commands:

```shell 
S11_IFACE=$(ip route get $(dig +short spgw.s11.ngic) | head -1 | awk '{print $3}')
S1U_IFACE=$(ip route get $(dig +short spgw.s1u.ngic) | head -1 | awk '{print $3}')
SGI_IFACE=$(ip route get $(dig +short spgw.sgi.ngic) | head -1 | awk '{print $3}')
```

Play the S11 (control plane) traffic to set up the flows

`tcpreplay --pps=200 -i $S11_IFACE tosend-s11.pcap`

Look at the Control Plane (Screen #2) and make sure that the packets appear. There should be 2000 packets sent/2000 received (from the 1000 CreateSession and 1000 ModifyBearer packets.)

Now start the S1U (Data Plane uplink) traffic

`tcpreplay --pps=2000 -i $S1U_IFACE tosend-s1u.pcap`

Check the Data Plane  (Screen #1).  You should see ~6500 packets received on the S1U and ~6500 packets transmitted on the SGi.

Now start the SGi (Data Plane downlink) traffic

`tcpreplay --pps=2000 -i $SGI_IFACE tosend-sgi.pcap`

Check the Data Plane  (Screen #1).  You should see ~6500 packets received on the SGi and ~6500 packets transmitted on the S1U.


Tear Down
=========
From the Traffic terminal, type:

`exit` 

and then type:

`docker-compose -p epc down`
