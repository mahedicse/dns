## Installation and Configuration Domain Name System (DNS) Server in ContOS Linux
For bangla tutorial visit: http://www.mahedi.me

## Domain Name System (DNS)
A Domain Name System (DNS) is a distributed hierarchical system. It maintains a directory of domain names and translate them to Internet Protocol (IP) addresses and Internet Protocol (IP) to domain names or hostname. Inform which are the official Name Servers for a particular Domain.

## DNS Components

* **DNS resolver** — Resides on the client-side of the DNS. When a user sends a hostname request, the resolver sends a DNS query request to the name servers to request the hostname's IP address.
* **Name servers** — Processes the DNS query requests received from the DNS resolver and returns the IP address to the resolver.
* **Resource records** — Data elements that define the basic structure and content of the DNS.

## DNS Server Types

* **Root Server** - It's contains information of all the TLDs and TLD's  server information like `` .com .org .net .bd .edu .gov `` etc. There are 13 root server with multiple instances of each server all over the world you'll find details from http://root-servers.org
* **TLDs Server** - Contains all the domains information in particular TLDs like Name Server of .bd hold all the .bd domains Name Server and IP addresses.
* **Name Server** - Contains all the zones of domains and resource database for each domain.
* **Recursion Server (Resolver)** - It does not contain any information for any domain but initiates a DNS query request to complete DNS address resolution. It maintains a cache database for fast serving.

## Categories of DNS Server
* **Primary Name Server** - It's an authoritative server of a domain and resource records update directly in the server.
* **Secondary Name Server** - It's also an authoritative server of a domain but resource records are not updated directly in the server. It gets the update from a Primary Name Server of a particular domain.

## Installation Bind Name Server

**Primary Server:**
Hostname: ns1.group-xy.ac.bd (Replace XY with your group number)
IP: IP address of your server

**Seconday Server:**
Hostname: ns2.group-xy.ac.bd (Replace XY with your group number)
IP: IP address of your server

### Server Basic Configuration for IRS:


#### Update System:

```` bash
# yum update -y

````

#### Hostname Configuration:
```` bash 
# vim /etc/hostname

ns1.group-XY.ac.bd
````
```` bash 
# hostname ns1.group-XY.ac.bd
````
> **Note:** Please replace **XY** with your group ID like: **ns1.group-01.ac.bd** 

**Check Hostnane configuration:**
```` bash 
# hostname

ns1.group-XY.ac.bd
````
```` bash 
# hostname -d
group-XY.ac.bd
````
#### Disable Selinux:
```` bash
# vim /etc/selinux/config

SELINUX=disabled
````
Now Reboot your server

````
# reboot
````
#### Firewall Configuration: 

```` bash
firewall-cmd --zone=public --permanent --add-port=53/udp
firewall-cmd --zone=public --permanent --add-port=53/tcp
firewall-cmd --reload

firewall-cmd --zone=public --permanent --list-all
````
**Install Bind**
At first we check bind is already installed or not by the following command:

``root@ns1 ~]# rpm –qa|grep bind``

If it's installed you'll found the following output:

``bind-9.8.2-0.17.rc1.el6_4.6.x86_64
bind-libs-9.8.2-0.17.rc1.el6_4.6.x86_64
bind-utils-9.8.2-0.17.rc1.el6_4.6.x86_64
``
And if it's not installed then install by the following command:

``[root@ns1 ~]# yum install –y bind bind-utils ``



