# Setting Up dnsmasq as a Local DHCP and DNS Server
_Using FreeNAS 11.2 U3_

These notes assume the use of iocage jails, new in FreeNAS 11.2, not the deprecated warden jails.<br>
Adapted from a blog post by "Tim": http://www.wordpress.lonbil.co.uk/2013/08/installing-dnsmasq-on-freenas-9-1/

## 1) Create a new jail to install dnsmasq into.
**IMPORTANT: Make sure to give the jail an IP address outside of the router's DHCP range.**<br>
Use the FreeNAS UI to create a new jail, giving it a name, an IP address, set it to auto-start and set it a high priority under Custom Properties (the lower the number, the higher the boot priority).

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

#### 2.1.2) Add GoogleDNS (or other DNS providers) to the jail's upstream nameservers.
Change line 66 to, and add line 67:
```
server=/8.8.8.8
server=/8.8.4.4
```

#### 2.1.3) Set the DHCP range.
Set the DHCP range on line 158. Make sure to also change your router's DHCP range if necessary so there's no conflict here.
```
dhcp-range=192.168.1.210,192.168.1.229,12h
```
Now press escape, select leave editor and save.

### 2.2) Set dnsmasq to auto-start.
```
edit /etc/rc.conf
```
At the end of the file, add the following (comment included for good practice):
```
# Auto-start dnsmasq
dnsmasq_enable="YES"
dnsmasq_conf="/usr/local/etc/dnsmasq.conf"
```
Press escape, select leave editor and save changes.

## 3) Restart
Close the shell console and restart the jail through the FreeNAS UI to make sure it auto-starts.