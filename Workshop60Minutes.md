# SCIONWorkshop - Path Aware Networking (PAN)

SCION is a next generation, clean slate technology to address some of the fundamental security issues with the Internet. Path Aware Networking (PAN), a feature of SCION, allows multiple paths to be available simultaneously for selection by the application or end user when a network connection is originated. One, or more paths, can be selected based upon a multitude of technical, business, and compliance/regulatory metrics or needs (i.e. bandwidth, latency, hops, MTU, price, must stay in EU, etc).

This workshop walks attaching a host to the SCIONLab network and examining Path Aware Networking in details. This includes examining available paths, the metrics of the paths, and then selecting the desired path for a connection. This workshop also walks through now to attach a legacy IPv4 application to SCION via the SCION Internet Gateway (SIG). The workshop wraps up with an example of connecting to the ETH "Hell" Attachment Point that introduces packet loss, latency, and bandwidth caps to better illustrate path selection.

## Prerequisites

As an workshop attendee, you'll need an Internet attached laptop with an SSH client.

## SCION Basics...

Just some initial information and nomenclature when it comes to SCION...

### SCION Addressing

A SCION address is composed of the ISD, AS, IP address, and port. So 19-ffaa:0:1303,[10.20.30.40:22] would be read as ISD #19 (EU), AS ffaa:0:1303 (Magdeburg AP, DE), 10.20.30.40 and port 22. The ISD is the isolation domain - a unit of trust and a collection of ASes. The AS is an SCION assigned Autonomous System that belongs to the ISD. A core AS has network connections to ASes at other ISDs. An IA is the combination of the ISD and the AS.

| Attribute     | Name              | Sample         | Description                                                                        | 
| ------------- |-------------------|----------------|-----------------------------------------------------------------------------------| 
| ISD           | Isolation Domain  | 19             | A collection of ASes that belong to the same trust group (i.e. company or nation).|
| AS            | Autonomous System | ffaa:0:1303    | A collection of IP networks. An AS must belong to only one ISD.                   |
| Core AS       | Core AS           | ffaa:0:1301    | An AS that has connections to ASes in other ISDs.                                 |
| Attachment AS | Attachment AS     | ffaa:0:1301    | An AS that allows other ASes to attach to it.                                     |


19-ffaa:0:1303,[10.20.30.40:22] can be broken out as:

| Attribute     |Value           | Description                        | 
| ------------- |----------------|------------------------------------| 
| ISD           | 19             | EU (Europe) Isolation Domain       |
| AS            | ffaa:0:1303    | Magdeburg AS                       |
| IA            | 19-ffaa:0:1303 | Magdeburg AS AP EU                 |
| IP            | 10.20.30.40    | IP address within the Magdeburg AS |
| Port          | 22             | Port on the host 10.20.30.40       |

If there's no port component, you can commit the brackets around the IP address (i.e. 19-ffaa:0:1303,10.20.30.40).

### Workshop Lab Machine Assignments

For this workshop, we have preconfigured a SCION host for your use. The workshop instructor will provide you with login credentials. You will be using SSH (Secure Shell) to log into this host.

### Your SCION AS

As part of this workshop, you'll be using an existing SCION host that has is part of an existing SCION AS. You'll need to take note of the SCION ASN that you are using. To following "scion address" command will provide output of your local AS and IP address.

Run this command:
```
scion address
```

The output should look like:
```
scionlab@scionlab:~$ scion address
19-ffaa:1:e98,127.0.0.1
```

You will see something like "19-ffaa:1:e98,127.0.0.1" indicating an ISD of 19 and an AS of ffaa:1:e98.

### Verify basic SCION connectivity

Verify that your host can another host across the SCION network. This can be done with the "scion ping" command. This pings the core router (127.0.0.1) located at the 19-ffaa:0:1303 AS.

Run this command:
```
scion ping -c 5 19-ffaa:0:1303,127.0.0.1
```

The output should look like:
```
scionlab@scionlab:~$ scion ping -c 5 19-ffaa:0:1303,127.0.0.1
Resolved local address:
  127.0.0.1
Using path:
  Hops: [19-ffaa:1:e98 2>495 19-ffaa:0:1303] MTU: 1472 NextHop: 127.0.0.1:30001

PING 19-ffaa:0:1303,127.0.0.1:0 pld=0B scion_pkt=80B
88 bytes from 19-ffaa:0:1303,127.0.0.1: scmp_seq=0 time=13.002ms
88 bytes from 19-ffaa:0:1303,127.0.0.1: scmp_seq=1 time=13.006ms
88 bytes from 19-ffaa:0:1303,127.0.0.1: scmp_seq=2 time=13.176ms
88 bytes from 19-ffaa:0:1303,127.0.0.1: scmp_seq=3 time=13.130ms
88 bytes from 19-ffaa:0:1303,127.0.0.1: scmp_seq=4 time=13.188ms

--- 19-ffaa:0:1303,127.0.0.1 statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 5001.577ms
rtt min/avg/max/mdev = 13.002/13.100/13.188/0.081 ms
```

## Examine the SCIONLab Network

### SCIONLab Topology

A static representation of the SCIONLab is available at https://www.scionlab.org/topology.png. You can use it as a reference when you examine the network paths that SCION presents.

### Showpaths - Single Hop

One of the most useful tools is the "scion showpaths" command. This displays the paths that are currently available to a remote AS. Let's start small with the path(s) from your AS to the AS that you're connected.

```
scion showpaths 19-ffaa:0:1303
```

*Why is there only one path back to 19-ffaa:0:1303?* Hint: How many attachment links does your host have to the upstream AS? Refer back to the network map https://www.scionlab.org/topology.png.

### Showpaths - Going Further

Let's go a little further into the network reaching the Magdeburg Core at ffaa:0:1301.

Before you run the command, think for a moment how many paths there might be from your AS to ffaa:0:1301 (Magdeburg Core). Take a look at the SCIONLab topology  https://www.scionlab.org/topology.png and trace out all the paths between your from your AS AP (ffaa:0:1303) to ffaa:0:1303. *Not all the paths may be within your ISD. There may be paths that go via other ISDs*

```
scion showpaths 19-ffaa:0:1301
```

*How many paths where there?* Why are there paths through ISD 17 (Switzerland)?

*Why would you utilize an alternate path?* What reasons might there be to utilize an alternate path? Failover? Latency? Bandwidth? Regulatory?

Feel free to run "scion showpaths" to other networks further away. 18-ffaa:0:1201 (Carnegie Mellon University, North America) is a good choice.

### Traceroute

The scion traceroute command will put packets on the wire tracing them out along the full path. SCION routers along the way will respond back mapping out the path. This differs from "showpaths" which uses routing information to display the available paths. Traceroute actually transmits packets.

Rerun the above "scion showpaths" but as "scion traceroute" instead.
```
scion traceroute 19-ffaa:0:1303,127.0.0.1
```

Pick an AS a little further away. 19-ffaa:0:1301 is a few hops further away.
```
scion traceroute 19-ffaa:0:1301,127.0.0.1
```

### Traceroute - Pick your Path

Next we're going to run traceroute but pick the path to be used. Our previous traceroutes used the path that SCION recommended.

Rerun showpaths to the Magdeurg Core AS and take note of the path that goes through ISD 17 (Switzerland).
```
scion showpaths 19-ffaa:0:1301
```

Now we're going to use the *-i* command to run traceroute and pick our path interactively. 
```
scion traceroute 19-ffaa:0:1301,127.0.0.1 -i
```

Enter in the number (i.e. 4) of the path that goes through ISD-17. The traceroute will now utilize that path. Take note of the packets traversing ISD-17.

Congrats! You just made your first path selection use Path Aware Networking!


## SCION Native Application - Sensor

Your SCION Host has a number of SCION native applications installed. A SCION native application is able to directly utilize SCION IP addresses and policy based path networking. scion-sensor is one such application. There is a sensor server running at "17-ffaa:0:1102,[192.33.93.177]:42003". We'll be using this application to examine the Path Aware Networking capabilities of SCION Native Applications.


First we're going to make sure we can ping the remote host.
```
scion ping 17-ffaa:1:f53,127.0.0.1 -c 1
scion ping 17-ffaa:0:1102,[192.33.93.177] -c 1
```
And we're going to check that SCION has a full set of paths to the remote AS. You should see at least half a dozen paths to the remote AS.
```
scion showpaths 17-ffaa:1:f53
```

### Sensor with Default Path Selection

Fetch data with the default PAN (Path Aware Networking) policy. This will use the "best" path available.
```
scion-sensorfetcher -s  17-ffaa:1:f53,127.0.0.1:42003
```
You should get some data back from the remote sensor. The information you receive back from the sensor is irrelevant. What is important is that SCION decided the best path, from all available paths at that moment, and made the connection.

### Sensor with Interactive Path Selection

Next we're going to fetch data from the same remote sensor but interactively select the network path. This is done with the '-i' flag to the command.
```
scion-sensorfetcher -i -s 17-ffaa:1:f53,127.0.0.1:42003
```
Take note of all the paths that are prompted. Select one by typing in the path number and the connection will be initiated across that path. We've done this before with traceroute so nothing new.

### Sensor with Path Preference

Finally we're going to fetch data and allow SCION to select the path based upon a preference. The available preferences configured are MTU, latency, bandwidth, and hops. The path preference is selected with the flag "-preference" and the preference string.

*What preference should we select for our sensors fetcher?* How many bytes does the sensorfetcher return? Not that many so bandwidth need not be one of our criteria. But sensor data is timely so latency would be a better preference. 

Run the sensorfetcher with a preference for a path with low latency.
```
scion-sensorfetcher -preference latency -s 17-ffaa:1:f53,127.0.0.1:42003
```
The extended scion path command can get the details on the MTU, latency, bandwidth, and hops.
```
scion showpaths 17-ffaa:1:f53 --extended
```

### Wrapup

## Thank You
Thanks for running through this workshop. We're glad you joined us!

## Feedback Appreciated
If you see mistakes or have comments, please feel free to submit an issue or a PR on this repo.

## More Info?

Follow us on Twitter: https://twitter.com/SCION_Workshop

Sign up for the SCION newsletter: https://www.scion.org/#contact

Copyright (C) 2024 - JHL Consulting LLC & Martincoit Networks










