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
IP: IP address of your server [192.168.0.5]

**Seconday Server:**
Hostname: ns2.group-xy.ac.bd (Replace XY with your group number)
IP: IP address of your server [192.168.0.10]

### Server Basic Configuration for IRS:

#### Add EPEL Repository 
```
# yum install epel-release
```
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

Update ``/etc/hosts`` file:

```
[root@ns1 ~]# vim /etc/hosts
127.0.0.1       localhost.localdomain		localhost
192.168.0.5     ns1.group-XY.ac.bd			ns1

:x
```
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

``` 
bind-9.8.2-0.17.rc1.el6_4.6.x86_64
bind-libs-9.8.2-0.17.rc1.el6_4.6.x86_64
bind-utils-9.8.2-0.17.rc1.el6_4.6.x86_64
```
And if it's not installed then install by the following command:

```[root@ns1 ~]# yum install –y bind bind-utils ```

#### Configure Bind Name Server
At first create a backup before change any in main configuration file:

```
[root@ns1 ~]# cd /etc/
[root@ns1 etc]# cp named.conf named.conf.ori
```
And change the configuration like following:
```
[root@ns1 ~]# vim /etc/named.conf
```
```
options {
        listen-on port 53 { 192.168.1.5; };
    //  listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        allow-query { any; };
 	allow-recursion { 192.168.1.0/24; };
        
	 dnssec-enable yes;
        dnssec-validation yes;
        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.iscdlv.key";
        managed-keys-directory "/var/named/dynamic";
        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

// Adding forward zone

zone "group-XY.ac.bd" IN {
        type master;
        file "db.group-XY.ac.bd";
        allow-transfer { none; };
};

// Adding Reverse zone

zone "1.168.192.in-addr.arpa" IN {
        type master;
        file "db.1.168.192.in-addr.arpa";
        allow-transfer { none; };
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

Zone files are contained in /var/named/  directory

````bash
[root@ns1 ~]# cd /var/named/

[root@ns1 named]# ls -la

drwxr-x---.  5 root  named 4096 Jul 24 17:04 .
drwxr-xr-x. 23 root  root  4096 Jul 24 17:04 ..
drwxrwx---.  2 named named    6 Jul  5 06:15 data
drwxrwx---.  2 named named    6 Jul  5 06:15 dynamic
-rw-r-----.  1 root  named 2281 May 22 05:51 named.ca
-rw-r-----.  1 root  named  152 Dec 15  2009 named.empty
-rw-r-----.  1 root  named  152 Jun 21  2007 named.localhost
-rw-r-----.  1 root  named  168 Dec 15  2009 named.loopback
drwxrwx---.  2 named named    6 Jul  5 06:15 slaves
````

Copy existing zone file for sample configuration with your given name in named.conf file like  the following:

```
[root@ns1 named]# cp named.localhost db.group-XY.ac.bd
[root@ns1 named]# cp named.loopback db.1.168.192.in-addr.arpa
```
Now open your forward zone file changed the options like following:

```
[root@ns1 named]# vim db.group-XY.ac.bd
```
```
$TTL 1D
@       IN SOA  ns1.group-XY.ac.bd. root.group-XY.ac.bd. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      ns1.group-XY.ac.bd.
        A       192.168.1.5
ns1			IN    	A       	192.168.1.5
mail			IN    	A       	192.168.1.5
group-XY.ac.bd.		IN	MX	10	mail.group-XY.ac.bd.
www			IN   	CNAME   	ns1.group-XY.ac.bd.
ftp			IN   	A       	192.168.1.50
```
Change in Reverse zone file changed the options like following:
```
[root@ns1 named]# vim db.1.168.192.in-addr.arpa
```
```
$TTL 1D
@       IN SOA  ns1.group-XY.ac.bd. root.group-XY.ac.bd. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      ns1.bdren.net.bd.

5      IN	PTR     ns1.group-XY.ac.bd.
50	IN	PTR     ftp.group-XY.ac.bd.
```

