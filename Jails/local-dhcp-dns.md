# Setting Up dnsmasq as a Local DHCP and DNS Server
_Using FreeNAS 11.2 U3_

These notes assume the use of iocage jails, new in FreeNAS 11.2, not the deprecated warden jails.

Adapted from a blog post by "Tim": http://www.wordpress.lonbil.co.uk/2013/08/installing-dnsmasq-on-freenas-9-1/

## 1) Create a new jail to install dnsmasq into.
**IMPORTANT: There is a bug in FreeNAS 11.2 U3 which causes jail creation to fail using the Advanced Jail Creation button. To work around it, use basic jail creation to create the jail initially, then edit the jail and change the rest of the options. See https://redmine.ixsystems.com/issues/70993 for more.**

Use the FreeNAS UI to create a new jail, using the Advanced Jail Creation button and setting the following properties:
#### Basic Properties
- Name
- Release
- VNET
- DHCP Autoconfigure IPv4
- Auto-Start

Or if not using DHCP:
- IPv4 Interface (The network device to use, e.g. bge0)
- IPv4 Address (The jail's static IP address)
- IPv4 Netmask (e.g. 24, which is equivalent to 255.255.255.0)
- IPv4 Default Router (The router's static IP address)
#### Custom Properties
- Priority (Lower numbers are higher boot priority)

## 2) Open the jail's shell console and set up dnsmasq.
Open the console through the FreeNAS UI. The FreeBSD package manager, pkg, isn't installed when creating a new jail but it will prompt you to install it when you first attempt to use it. It may take a few minutes to finish setting up.
```
pkg install dnsmasq
```

### 2.1) Set up the dnsmasq configuration.
Open the dnsmasq configuration file for editing.
```
edit /usr/local/etc/dnsmasq.conf
```
#### 2.1.1) Be a "good netizen".
Uncomment lines 19 and 21 to enable the `domain-needed` and `bogus-priv` options. The comment above these lines make it sound like a good idea, though I've not gone into depth as to why exactly.

#### 2.1.2) Add DHCP or static IP addresses for hosts.
If using dnsmasq for a DHCP server, set the DHCP range on line 158.
```
dhcp-range=192.168.1.210,192.168.1.229,12h
```
Now press escape, select leave editor and save.

If you're sticking with your router's DHCP server, set DHCP reservations for your devices in the router's configuration and add the appropriate entries to the hosts file:
```
edit /etc/hosts
```
When you're done, press escape and select leave editor and save changes.

### 2.2) Set dnsmasq to auto-start.
```
edit /etc/rc.conf
```
At the end of the file, add the following (comment included for good practice):
```
# Use dnsmasq
dnsmasq_enable="YES"
dnsmasq_conf="/usr/local/etc/dnsmasq.conf"
```
Press escape, select leave editor and save changes.

### 2.3) Test dnsmasq configuration.
Running dnsmasq will tell you if there are any errors in your configuration file. Try it out with
```
dnsmasq
```

## 3) Restart
Once the configuration is finished, close the shell console and restart the jail through the FreeNAS UI to make sure it auto-starts.
