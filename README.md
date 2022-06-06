# SCIONWorkshop - Path Aware Networking (PAN)

SCION is a next generation, clean slate technology to address some of the fundamental security issues with the Internet. Path Aware Networking (PAN), a feature of SCION, allows multiple paths to be available simultaneously for selection by the application or end user when a network connection is originated. One, or more paths, can be selected based upon a multitude of technical, business, and compliance/regulatory metrics or needs (i.e. bandwidth, latency, hops, mtu, price, must stay in EU, etc).

This workshop walks attaching a host to the SCIONLab network and examining Path Aware Networking in details. This includes examining available paths, the metrics of the paths, and then selecting the desired path for a connection. This workshop also walks through now to attach a legacy IPv4 application to SCION via the SCION Internet Gateway (SIG). The workshop wraps up with an example of connecting to the ETH "Hell" Attachment Point that introduces packet loss, latency, and bandwidth caps to better illustate path selection.

## Prerequisites

Before starting the workshop, you'll need a few things. The workshop presenter can help you as needed.

### Attendee Software

As an workshop attendee, you'll need an Internet attached laptop with an SSH client and a web browser.

### Workshop Attendee Host

At the start of the workshop, you'll be assigned a host to use via an Etherpad. Take note of the IP address and the username/password to log into the 

### Register with SCIONLab

Create an account at [https://www.scionlab.org/](https://www.scionlab.org/registration/register/). SCIONLab is a global research network to test the SCION next-generation internet architecture. This will be used to connect your host to a SCION network.

You may need to disable ad blocking in order to view the CAPTCHA.

## SCION Basics...

Just some initial information and nomenclature when it comes to SCION...

### SCION Addressing

A SCION address is composed of the ISD, AS, IP address, and port. So 19-ffaa:0:1303,[10.20.30.40:22] would be read as ISD #19 (EU), AS ffaa:0:1303 (Magdeburg AP, DE), 10.20.30.40 and port 22. The ISD is the isolation domain - a unit of trust and a colletion of ASes. The AS is an SCION assigned Autonomous System that belongs to the ISD. A core AS has network connections to ASes at other ISDs. An IA is the combination of the ISD and the AS.

Putting it all together...

```
19-ffaa:0:1303,[10.20.30.40:22] can be broken out as:
ISD: 19                     -> EU (Europe) Isolation Domain
AS: ffaa:0:1303             -> Magdeburg AS
IA: 19-ffaa:0:1303
IP: 10.20.30.40             -> IP address within the Magdeburg AS
Port: 22                    -> Port on the host
```

If there's no port component, you can commit the brackets around the IP address (i.e. 19-ffaa:0:1303,10.20.30.40).

### SCIONLab versus SCION

SCION is in production across several ISPs and financial institutions in Switzerland. SCIONLab is a seperate instance of SCION from the production instance in Switzerland. SCIONLab spans the globe and is designed to allow researchers and hobbyists alike to work with SCION in a collaborative fashion. The SCIONLab website was designed to faciliate the easy connection of hosts. SCIONLab primarily runs atop tradtional Internet connections as opposed to dedicated WAN circuits. This compromise means that SCIONLab does not have the true resilience of a SCION network running atop dedicated circuits.

## Your SCION AS

As part of this workshop, you'll be creating your own AS under one of the existing SCIONLab ISDs. 

### Register your SCION AS

On the SCIONLab website https://www.scionlab.org/user/, navigate to "My ASes" and click "Create a new SCIONLab AS". Name your AS after your workshop host (i.e. Workshop 13). Select the "SCION installation from packages". The host you have been provided already has the SCION packages installed. There's no need to rerun the package installs.

Copy and set aside the scionlab-config command. You'll be running this command several times so save it onto a notepad or some scratch spot. The scionlab-config command is of the form "sudo scionlab-config --host-id=<...> --host-secret=<...>"

From the drop down, select the "19-ffaa:0:1303 (Magdeburg AP)" SCION Access Points (AP) as your upstream SCION connection. Leave the "Use VPN" option OFF (unchecked).

Enter the IP address of your assigned Attendee Host in the "Public IP" field. The Public Port (UDP) should be "50000".

"Save Changes" to register your new AS. Click "My ASes" and take note that your AS is now registered. Take note of the assigned AS. Make sure it is "Active".

Go back to the Etherpad and update your host entry with your SCION AS ID (19-ffaa:1:XXX).

### SCION AS Configuration

Login into (via SSH) the host that you've self-assigned. Use the username and password/key provided on the Etherpad.

Configure the host with the SCION AS you created above using scionlab-config. It will pull down the information about your AS and set it up on your host.

```
sudo scionlab-config --host-id=<...> --host-secret=<...>
```

Restart the SCION services.
```
sudo systemctl restart scionlab.target
```

Verify that your host has been assigned the correct AS with "scion address" command
```
scion address
```

Verify connectivity with a ping.

```
scion ping -c 5 19-ffaa:0:1303,127.0.0.1
```

## Examine the SCIONLab Network

### SCIONLab Topology

A static representation of the SCIONLab is available at https://www.scionlab.org/topology.png. You can use it as a reference when you examine the network paths that SCION presents.

### Showpaths

One of the most useful tools is the "scion showpaths" command. This displays the paths that are currently available to a remote AS. Let's start small with the path(s) from your AS to the AS that you're connected.

```
scion showpaths 19-ffaa:0:1303
```

You should just see the one path since there's just a single connection from your host to the upstream Attachment Point.

Let's do a more complicated path. 19-ffaa:0:1301 is just a single hop from the upstream AP but has some additional paths with more hops.

```
scion showpaths 19-ffaa:0:1301
```

Pull up the SCIONLab topology https://www.scionlab.org/topology.png and trace out all the paths that are displayed.

Feel free to run "scion showpaths" to other networks further away.

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
Feel free to run "scion traceroute" to other networks further away.

### Web Visualization Tools - Installation

Installed on your workshop host is a collection of web (browser) based applications. Let's start up the webapps and take a look.

Startup the web apps with the command:
```
sudo systemctl start scion-webapp
```
Navigate your web browser to the web app page running on port 8000 of your workshop host.
```
http://<workshop_host_ipv4>:8000/
```

### Web Visualization Tools - Traceroute

We will repeat the traceroute command done above via the web interface. 

Click the "traceroute" tab and enter in the destination of 19-ffaa:0:1303,[127.0.0.1:1]. Then click tab "Execute" and the "Run Once" button. You'll see the textual representation of the traceroute. Click the "Paths" tab and toggle on the "Topology" radio button to see a graphical representation of the traceroute. It's pretty simple since the Magdeburg AS is a single hop away.

We'll do a more complex route next going to the Magdeburg Core AS (19-ffaa:0:1301). Change the AS on the web page and once again click "Execute" and "Run Once". You'll see a much more complex traceroute. Click over to "Paths" and "Topology" to see a graphical representation showing the additional hops.

### Web Visualization Tools - Showpaths

The web visualization tools can also be used to see all the paths available. From the web app, click over to the "Update Paths" on the far right web panel. You'll see all the paths available between the two ASes. Select one of the longer ones that goes through ISD 17 (Switzerland). 

With the longer path selected, go ahead and rerun the traceroute. You'll see the traffic now leaves the EU ISD, traversing through the Swiss ISD, and then back into the EU ISD. The arrows on the graphic show the full path.


## SCION Native Application - Sensor

Your SCION Host has a number of SCION native applications installed. A SCION native application is able to direclty utilize SCION IP addresses and policy based path networking. scion-sensor is one such application. There is a sensor server running at "17-ffaa:0:1102,[192.33.93.177]:42003". We'll be using this application to examine the Path Aware Networking capabilities of SCION Native Applications.

First we're going to make sure we can ping the remote host.
```
scion ping 17-ffaa:0:1102,192.33.93.177 -c 1
```
And we're going to check that SCION has a full set of paths to the remote AS. You should see at least half a dozen paths to the remote AS.
```
scion showpaths 17-ffaa:0:1102
```

### Sensor with Default Path Selection

Fetch data with the default PAN (Path Aware Networking) policy. This will use the "best" path available.
```
scion-sensorfetcher -s 17-ffaa:0:1102,[192.33.93.177]:42003
```
You should get some data back from the remote sensor. The information you receive back from the sensor is irrelvant. What is important is that SCION decided the best path, from all available paths at that moment, and made the connection.

### Sensor with Interactive Path Selection

Next we're going to fetch data from the same remote sensor but interactively select the network path. This is done with the '-i' flag to the command.
```
scion-sensorfetcher -i -s 17-ffaa:0:1102,[192.33.93.177]:42003
```
Take note of all the paths that are prompted. Select one by typing in the path number and the connection will be initiated across that path.

### Sensor with Path Preference

Finally we're going to fetch data and allow SCION to select the path based upon a preference. The available preferences configured are mtu, latency, bandwidth, and hops. The path preference is selected with the flag "-preference" and the preference string.
```
scion-sensorfetcher -preference latency -s 17-ffaa:0:1102,[192.33.93.177]:42003
```
The extended scion path command can get the details on the mtu, latency, bandwidth, and hops.
```
scion showpaths 17-ffaa:0:1102 --extended
```
## SCION Native Application - Bandwidth Tester

The bandwidth tester can be used to measure the network throughput across various paths through the SCION network. It's available as a command line app as well as through the web app. We're going to be running it via the command line.

Run a test sending to ISD 17 (Switzerland) of 10 second duration (seconds), 1000 bytes packets, 1250 packets, and target 1 Mbps bandwidth in both directions (client to server and server to client). Take a look at the bandwidth values and inter packet arrival variance.
```
scion-bwtestclient -s 17-ffaa:0:1108,[195.176.28.157]:30100 -cs 10,1000,1250,1Mbps -sc 10,1000,12500,10Mbps
```
Repeat the above command with path selection turned on interactively (-i flag) and select a less optimal route (i.e. the last one on the interactive list).
```
scion-bwtestclient -i -s 17-ffaa:0:1108,[195.176.28.157]:30100 -cs 10,1000,1250,1Mbps -sc 10,1000,12500,10Mbps
```
Keep on increasing the target bandwidth to see what happens between the optimal route and alternate paths.

A full list of bandwidth tester servers is available online if you want to try a different bandwidth server.

https://docs.scionlab.org/content/apps/bwtester.html
 

## Legacy Application atop SCION

All of our examples so far have shown SCION native applications running atop a SCION network. Now we're going to show how to a legacy IPv4 application can benefit from a SCION network without application modifications. We'll be using running the OpenStack Keystone identity service over SCION by setting up a SCION Internet Gateway (SIG) between the OpenStack client and OpenStack Keystone server.

There is a Keystone server running at 19-ffaa:1:e98,[127.0.0.1:5000]

### SCION Internet Gateway (SIG)

The SCION Internet Gateway (SIG) creates a tunnel between two ASes. In our case, it is going to create a tunnel between your host SCIONLab AS and the 19-ffaa:1:e98 AS where Keystone is running.

The remote IPv4 network will be 172.16.1.0/24. The local IPv4 network on your host will be 172.16.X.0/24 where X is your workshop ID.

Make sure you update the sign-up sheet with your AS information! The presenter will use this information to setup the remote side of the tunnel

Update /etc/scion/sig.json with the information about the remote end of the SIG tunnel. sudo vi /etc/scion/sig.json

```
# /etc/scion/sig.json
{
    "ASes": {
        "19-ffaa:1:e98": { # Keystone AS - do not change this AS
            "Nets": [
                "172.16.10.0/24"  # do not change this IP
            ]
        }
    },
    "ConfigVersion": 9001
}
```

Add the following lines to /etc/scion/sig.toml
```
# add these lines

[tunnel]
src_ipv4 = "172.16.13.1"   # replace 13 with your workshop number
```

Add the new subnet to your workstation host via the loopback and restart the SIG gateway
```
sudo ip address add 172.16.13.1 dev lo  # replace 13 with your workshop number
systemctl restart scion-ip-gateway.service
```

Test the connection.
```
ping 172.16.10.1 -c 1
```

### OpenStack Keystone atop SIG

An Openstack credential file has been saved in the root directory for use. Modify it with the desired remote IP address.

```
sudo -i
vi ~root/demo-openrc
```

Change the IP address to the 172.16.10.1 in the demo-openrc.

Request a token from the remote Keystone server with the SCION network through the SIG.

```
source ~root/demo-openrc
openstack token issue
```

You've now successfully run an IPv4 service over SCION.
