# SCIONWorkshop
SCION Next Generation Internet Workshop &amp; Tutorial

## Prerequisites

### Attendee Software

As an workshop attendee, you'll need an Internet attached laptop with an SSH client and a web browser.

### Workshop Attendee Host

At the start of the workshop, you'll be assigned a host to use via an Etherpad. Take note of the IP address and the username/password to log into the 

### Register with SCIONLab

Create an account at [https://www.scionlab.org/](https://www.scionlab.org/registration/register/). SCIONLab is a global research network to test the SCION next-generation internet architecture. This will be used to connect your host to a SCION network.

## Your SCION AS

### Register your SCION AS

On the SCIONLab website https://www.scionlab.org/user/, navigate to "My ASes" and click "Create a new SCIONLab AS". Pick a name for your AS and select the "SCION installation from packages". The host you have been provided already has the SCION packages installed. There's no need to rerun the package installs.

Copy and set aside the scionlab-config command. You'll be running this command several times so save it onto a notepad or some scratch spot. The scionlab-config command is of the form "sudo scionlab-config --host-id=<...> --host-secret=<...>"

From the drop down, select the "19-ffaa:0:1303 (Magdeburg AP)" SCION Access Points (AP) as your upstream SCION connection. Leave the "Use VPN" option OFF (unchecked).

Enter the IP address of your assigned Attendee Host in the "Public IP" field. The Public Port (UDP) should be "50000".

"Save Changes" to register your new AS. Click "My ASes" and take note that your AS is now registered. Take note of the assigned AS. Make sure it is "Active".

Go back to the Etherpad and update your host entry with your SCION AS ID (19-ffaa:1:XXX).

### SCION AS Configuration

Login into (via SSH) the host that you've self-assigned. Use the username and password/key provided on the Etherpad.

Configure the host with the SCION AS you created above. This done by running "sudo scionlab-config --host-id=<...> --host-secret=<...>" that you cut and paste from the SCIONLab website above.

Restart the SCION services "sudo systemctl start scionlab.target"

Verify that your host has been assigned the correct AS with "scion address" command

Verify connectivity with "scion ping -c 5 19-ffaa:0:1303,127.0.0.1".

## Examine the SCIONLab Network

### SCIONLab Topology

### Traceroute

### Path Aware Networking


scion-sensorfetcher -s 17-ffaa:0:1102,[192.33.93.177]:42003


### Visualization Tools


## SCION Native Application - Sensor

Your SCION Host has a number of SCION native applications installed. A SCION native application is able to direclty utilize SCION IP addresses and policy based path networking. scion-sensor is one such application. There is a sensor server running at "17-ffaa:0:1102,[192.33.93.177]:42003". We'll be using this application to examine the Path Aware Networking capabilities of SCION Native Applications.

First we're going to make sure we can ping the remote host.

scion ping 17-ffaa:0:1102,192.33.93.177 -c 1

And we're going to check that SCION has a full set of paths to the remote AS. You should see at least half a dozen paths to the remote AS.

scion showpaths 17-ffaa:0:1102


### Sensor with Default Path Selection

Fetch data with the default PAN (Path Aware Networking) policy. This will use the "best" path available.

scion-sensorfetcher -s 17-ffaa:0:1102,[192.33.93.177]:42003

You should get some data back from the remote sensor. The information you receive back from the sensor is irrelvant. What is important is that SCION decided the best path, from all available paths at that moment, and made the connection.

### Sensor with Interactive Path Selection

Next we're going to fetch data from the same remote sensor but interactively select the network path. This is done with the '-i' flag to the command.

scion-sensorfetcher -i -s 17-ffaa:0:1102,[192.33.93.177]:42003

Take note of all the paths that are prompted. Select one by typing in the path number and the connection will be initiated across that path.

### Sensor with Path Preference

Finally we're going to fetch data and allow SCION to select the path based upon a preference. The available preferences configured are mtu, latency, bandwidth, and hops. The path preference is selected with the flag "-preference" and the preference string.

scion-sensorfetcher -preference latency -s 17-ffaa:0:1102,[192.33.93.177]:42003

The extended scion path command can get the details on the mtu, latency, bandwidth, and hops.

scion showpaths 17-ffaa:0:1102 --extended


## SCION Native Application - Bandwidth Tester

https://docs.scionlab.org/content/apps/bwtester.html

### Policy Based Path Networking  

## Legacy Application atop SCION

### SCION Internet Gateway (SIG)

### OpenStack Keystone atop SIG
 
