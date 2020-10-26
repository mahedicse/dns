---
description: 'For Bangla tutorial visit: http://www.mahedi.me'
---

# Installation and Configuration Domain Name System \(DNS\) Server in CentOS Linux

## Domain Name System \(DNS\)

A Domain Name System \(DNS\) is a distributed hierarchical system. It maintains a directory of domain names and translates them to Internet Protocol \(IP\) addresses and Internet Protocol \(IP\) to domain names or hostname. Inform which are the official Name Servers for a particular Domain.

## DNS Components

* **DNS resolver** — Resides on the client-side of the DNS. When a user sends a hostname request, the resolver sends a DNS query request to the name servers to request the hostname's IP address.
* **Name servers** — Processes the DNS query requests received from the DNS resolver and returns the IP address to the resolver.
* **Resource records** — Data elements that define the basic structure and content of the DNS.

## DNS Server Types

* **Root Server** - It's contains information of all the TLDs and TLD's server information like `.com .org .net .bd .edu .gov` etc. There are 13 root server with multiple instances of each server all over the world you'll find details from [http://root-servers.org](http://root-servers.org/)
* **TLDs Server** - Contains all the domain's information in particular TLDs like Name Server of .bd hold all the .bd domains Name Server and IP addresses.
* **Name Server** - Contains all the zones of domains and resource database for each domain.
* **Recursion Server \(Resolver\)** - It does not contain any information for any domain but initiates a DNS query request to complete DNS address resolution. It maintains a cache database for fast serving.

## Categories of DNS Server

* **Primary Name Server** - It's an authoritative server of a domain and resource records update directly in the server.
* **Secondary Name Server** - It's also an authoritative server of a domain but resource records are not updated directly in the server. It gets the update from a Primary Name Server of a particular domain.

## Installation Bind Name Server

**Primary Server:** Hostname: ns1.group-XY.ac.bd \(Replace XY with your group number\) IP: IP address of your server \[192.168.0.5\]

**Secondary Server:** Hostname: ns2.group-XY.ac.bd \(Replace XY with your group number\) IP: IP address of your server \[192.168.0.10\]

### Server Basic Configuration for IRS:

**Add EPEL Repository**

```text
# yum install epel-release
```

**Update System:**

```text
# yum update -y
```

**Host name Configuration:**

```text
# vim /etc/hostname

ns1.group-XY.ac.bd
```

```text
# hostname ns1.group-XY.ac.bd
```

> **Note:** Please replace **XY** with your group ID like: **ns1.group-01.ac.bd**

Update `/etc/hosts` file:

```text
[root@ns1 ~]# vim /etc/hosts
127.0.0.1       localhost.localdomain        localhost
192.168.0.5     ns1.group-XY.ac.bd            ns1

:x
```

**Check Host name configuration:**

```text
# hostname

ns1.group-XY.ac.bd
```

```text
# hostname -d
group-XY.ac.bd
```

**Disable Selinux:**

```text
# vim /etc/selinux/config

SELINUX=disabled
```

Now Reboot your server

```text
# reboot
```

**Firewall Configuration:**

```text
firewall-cmd --zone=public --permanent --add-port=53/udp
firewall-cmd --zone=public --permanent --add-port=53/tcp
firewall-cmd --reload

firewall-cmd --zone=public --permanent --list-all
```

**Install Bind** At first, we check bind is already installed or not by the following command:

`root@ns1 ~]# rpm -qa|grep bind`

If it's installed you'll find the following output:

```text
bind-9.8.2-0.17.rc1.el6_4.6.x86_64
bind-libs-9.8.2-0.17.rc1.el6_4.6.x86_64
bind-utils-9.8.2-0.17.rc1.el6_4.6.x86_64
```

And if it's not installed then install by the following command:

`[root@ns1 ~]# yum install –y bind bind-utils`

**Configure Bind Name Server**

At first create a backup before change any in main configuration file:

```text
cd /etc/
cp named.conf named.conf.ori
```

And change the configuration like following:

```text
[root@ns1 ~]# vim /etc/named.conf
```

```text
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
};

// Adding Reverse zone

zone "1.168.192.in-addr.arpa" IN {
        type master;
        file "db.1.168.192.in-addr.arpa";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

Zone files are contained in /var/named/ directory

```text
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
```

Copy existing zone file for sample configuration with your given name in named.conf file like the following:

```text
cp named.localhost db.group-XY.ac.bd
cp named.loopback db.1.168.192.in-addr.arpa
```

Now open your forward zone file changed the options like following:

```text
vim db.group-XY.ac.bd
```

```text
$TTL 1D
@       IN SOA  ns1.group-XY.ac.bd. root.group-XY.ac.bd. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      ns1.group-XY.ac.bd.
        A       192.168.1.5
ns1            IN        A           192.168.1.5
mail            IN        A           192.168.1.5
group-XY.ac.bd.        IN    MX    10    mail.group-XY.ac.bd.
www            IN       CNAME       ns1.group-XY.ac.bd.
ftp            IN       A           192.168.1.50
```

Change in Reverse zone file changed the options like following:

```text
vim db.1.168.192.in-addr.arpa
```

```text
$TTL 1D
@       IN SOA  ns1.group-XY.ac.bd. root.group-XY.ac.bd. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      ns1.group-XY.ac.bd.
    A       192.168.1.5

5    IN    PTR     ns1.group-XY.ac.bd.
50    IN    PTR     ftp.group-XY.ac.bd.
```

check your configuration and zone file using the following command:

```text
named-checkconf -z /etc/named.conf

zone localhost.localdomain/IN: loaded serial 0
zone localhost/IN: loaded serial 0
```

```text
named-checkzone zone db.group-XY.ac.bd

zone zone/IN: loaded serial 0
OK
```

```text
[root@ns1 named]# named-checkzone zone db.110.168.192.in-addr.arpa
zone zone/IN: loaded serial 0
OK
```

If shown not OK then you need to check your zone file, it has something wrong in syntax, correct and checks again.

Changed group ownership:

```text
[root@ns1 named]# chgrp named db.group-XY.ac.bd
[root@ns1 named]# chgrp named db.1.168.192.in-addr.arpa
```

To start service and ensure start this service at startup run following command:

```text
[root@ns1 named]# systemctl restart  named.service
```

```text
[root@ns1 named]# systemctl enable named.service
ln -s '/usr/lib/systemd/system/named.service' '/etc/systemd/system/multi-user.target.wants/named.service'
```

Test from Linux clients you need to add name server address in /etc/resolv.conf file:

```text
[root@ns1 named]# vim /etc/resolv.conf
search group-XY.ac.bd
nameserver 192.168.1.5

:x
```

Now time to check your configuration:

```text
[root@ns1 named]# nslookup
> group-XY.ac.bd
Server:        192.168.1.5
Address:    192.168.1.5#53

Name:    group-XY.ac.bd
Address: 192.168.1.5
> www
Server:        192.168.1.5
Address:    192.168.1.5#53

www.group-XY.ac.bd    canonical name = ns1.group-XY.ac.bd.

Name:    ns1.group-XY.ac.bd
Address: 192.168.1.5
>
```

**Yes! You have done it!!!**

## **Secondary DNS Server Configuration:**

**Change in Primary Server:**

Add the following clause `allow-transfer { 192.168.1.10; };` in the zone 

```text
// Adding forward zone

zone "mahedi.me" IN {
 type master;
 file "db.mahedi.me.for";
 allow-transfer { 192.168.1.10; };
};

// Adding Reverse zone

zone "1.168.192.in-addr.arpa" IN {
 type master;
 file "db.110.168.192.in-addr.arpa";
 allow-transfer { 192.168.1.10; };

};
```

And add the secondary server information in the db files 

```text
[root@ns1 named]# vim db.mahedi.me.for

$TTL 1D
@ IN SOA          ns1.mahedi.me.   root.mahedi.me. (
                                                   0 ; serial
                                                  1D ; refresh
                                                  1H ; retry
                                                  1W ; expire
                                                3H ) ; minimum

               NS  ns1.mahedi.me.
               A   192.168.1.5
               NS  ns2.mahedi.me.                
               A   192.168.1.10
ns1     IN     A   192.168.1.5
ns1     IN     A   192.168.1.5
mail    IN     A   192.168.1.5
mahedi.me.      IN  MX    10    mail.mahedi.me.
www     IN     CNAME     ns1.mahedi.me.
ftp     IN     A         192.168.1.50

 

:x
```

```text
[root@ns1 named]# vim db.1.168.192.in-addr.arpa

$TTL 1D

@        IN      SOA       ns1.mahedi.me.      root.mahedi.me. (
                                                       0 ; serial
                                                      1D ; refresh
                                                      1H ; retry
                                                      1W ; expire
                                                    3H ) ; minimum

               NS  ns1.mahedi.me.                
               A   192.168.1.5
               NS  ns2.mahedi.me.                
               A   192.168.1.10

5          IN     PTR      ns1.mahedi.me.
10         IN     PTR      ns2.mahedi.me.
50         IN     PTR      ftp.mahedi.me.

:x
```

At first, you have to install software, configure firewall, hostname, and FQDN like the same as primary DNS server:

Changed /etc/named.conf in ns2 like following:

```text
[root@ns2 ~]# vim /etc/named.conf
```

```text
options {
        listen-on port 53 { 192.168.1.10; };
    //    listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
     allow-query { any; };
     allow-recursion { 192.168.1.0/24; };

    dnssec-enable yes;
        dnssec-validation yes;
        dnssec-lookaside auto;
        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.iscdlv.key";
        managed-keys-directory "/var/named/dynamic";
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
    type slave;
    masters { 192.168.1.5; };
    file "slaves/db.group-XY.ac.bd";
};

// Adding Reverse zone

zone "1.168.192.in-addr.arpa" IN {
        type slave;
        masters { 192.168.1.5; };
        file "slaves/db.1.168.192.in-addr.arpa";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

To start service and ensure start this service at startup run the following command:

```text
[root@ns1 named]# systemctl restart  named.service
```

```text
[root@ns1 named]# systemctl enable named.service
ln -s '/usr/lib/systemd/system/named.service' '/etc/systemd/system/multi-user.target.wants/named.service'
```

Let Check the resource records file are transferred or not

```text
[root@ns2 ~]# cd /var/named/slaves/

[root@ns2 ~]# ls -la
-rw-r-----   1 named  named  421 May 27 21:37 db.group-XY.ac.bd
-rw-r-----.  1 named  named  292 May 13 13:58 db.110.168.192.in-addr.arpa
```

Yes it has transferred

Test from Linux clients you need to add name server address in /ete/reslov.conf file:

```text
[root@ns2 named]# vim /ete/reslov.conf 

nameserver 192.168.1.10

:x
```

```text
[root@ns1 named]# nslookup
> group-XY.ac.bd
Server:        192.168.1.10
Address:    192.168.1.10#53

Name:    group-XY.ac.bd
Address: 192.168.1.10
> www
Server:        192.168.1.10
Address:    192.168.1.10#53

www.group-XY.ac.bd    canonical name = ns1.group-XY.ac.bd.

Name:    ns1.group-XY.ac.bd
Address: 192.168.1.10
>
```

## DNS Security Configuration

**Configure Log**

Create Log Directory:

```text
# mkdir /var/log/named
# chown named:named /var/log/named
```

Edit `/etc/named.conf` file in `loggong { }` options like the following:

```text
# vim /etc/named.conf
```

```text
// named.conf fragment

logging {
        channel normal_log {
                file "/var/log/named/normal.log" versions 3 size 2m;
                print-time yes;
                print-severity yes;
                print-category yes;
        };

        channel security_log { // streamed security log
                file "/var/log/named/security.log" versions 3 size 2m;
                severity info;
                print-time yes;
                print-severity yes;
                print-category yes;
        };

        category default{
                normal_log;
        };

        category security{
                security_log;
        };

};
```

And restart the service and check the log files:

```text
# service named restart
```

## **Securing Zones Transfer:**

Changed in Primary DNS servers /etc/named.conf file in only zone section like following:

Deny All, Allow Selectively

```text
options {
....
    allow-transfer {none;}; // no transfer by default
....
};
```

```text
// Adding forward zone

zone "group-XY.ac.bd" IN {
        type master;
        file "db.group-XY.ac.bd";
        allow-transfer { 192.168.1.10; };
};

// Adding Reverse zone

zone "1.168.192.in-addr.arpa" IN {
        type master;
        file "db.1.168.192.in-addr.arpa";
        allow-transfer { 192.168.1.10; };
};
```

```text
[root@ns2 ~]# systemctl restart named.service

[root@ns2 ~]# dig bdren.net @IP-of-NS2
```

## **Authentication and Integrity of Zone Transfers**

TSIG Configuration

```text
# mkdir /var/named/keys
# chown named.named /var/named/keys
# cd /var/named/keys
```

```text
[root@ns1 keys]# dnssec-keygen -a hmac-md5 -b 128 -n host -C transfer-key
Ktransfer-key.+157+62145
```

```text
[root@ns1 keys]# ls -la

-rw------- 1 root  root   56 Apr 16 23:51 Ktransfer-key.+157+62145.key
-rw------- 1 root  root   92 Apr 16 23:51 Ktransfer-key.+157+62145.private
Viewing the file Ktransfer-key.+157+62145.private displays something like the following data:
```

```text
[root@ns1 keys]# cat Ktransfer-key.+157+62145.private

Private-key-format: v1.2
Algorithm: 157 (HMAC_MD5)
Key: ndu1mcSlVRqGl3Qb35Clqw==
Bits: AAA=
```

The preceding information contains four lines. The line beginning with the text Key: is the base64 \(RFC 4648\) encoded version of the shared-secret key. Next step is to edit this data into a key clause that will be used in the named.conf file, as shown here:

```text
key "transfer-key" {
         algorithm hmac-md5;
         secret ndu1mcSlVRqGl3Qb35Clqw==;
};

server 192.168.0.10 {
        keys {"transfer-key";}; // name used in key clause
};

options {
....
directory "/var/named";
dnssec-enable yes; // default and could be omitted
....
};

// Adding forward zone
zone "group-XY.ac.bd" IN {
        type master;
        file "db.group-XY.ac.bd";
        // allow transfer only if key (TSIG) present
        allow-transfer {key "transfer-key";};
};
```

Test Configuration:

```text
[root@ns1 keys]# dig @192.168.1.5 group-XY.ac.bd AXFR -k Ktransfer-key.+157+62145.key
```

### **Configure in slave server:**

```text
key "transfer-key" {
         algorithm hmac-md5;
         secret NJSXJmLkg2VCqnEXhp60wQ==;
};


server 192.168.1.5 {
        keys {"transfer-key";}; // name used in key clause
};
```

Ensure dnssec-enable yes;

```text
options {
....
directory "/var/named";
dnssec-enable yes; // default and could be omitted
....
};

zone "group-XY.ac.bd" IN {
        type slave;
        masters { 192.168.1.5; };
        file "slaves/slave.group-XY.ac.bd";
};
```

```text
[root@ns2 ~]# systemctl restart named.service
```

```text
[root@ns2 ~]# dig group-XY.ac.bd @192.168.1.10
```

