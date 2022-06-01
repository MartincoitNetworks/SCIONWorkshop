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

Copy and set aside the scionlab-config command. You'll be running this command several times so save it onto a notepad or some scratch spot. The scionlab-config command is of the form "sudo scionlab-config --host-id=32characterID --host-secret=32characterSecret

From the drop down, select the "19-ffaa:0:1303 (Magdeburg AP)" SCION Access Points (AP) as your upstream SCION connection. Leave the "Use VPN" option OFF (unchecked).

Enter the IP address of your assigned Attendee Host in the "Public IP" field. The Public Port (UDP) should be "50000".

"Save Changes" to register your new AS. Click "My ASes" and take note that your AS is now registered. Take note of the assigned AS. Make sure it is "Active"

### SCION AS Configuration


## SCION Network Topology


## OpenStack Keystone over SCION



