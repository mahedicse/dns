# 
## Domain Name System (DNS)
A Domain Name System (DNS) is a distributed hierarchical system. It’s maintain a directory of domain names and translate them to Internet Protocol (IP) addresses and Internet Protocol (IP) to domain names or hostname. Inform which are the official Name Servers for a particular Domain.

## DNS Components

* **DNS resolver** — Resides on the client side of the DNS. When a user sends a hostname request, the resolver sends a DNS query request to the name servers to request the hostname's IP address.
* **Name servers** — Processes the DNS query requests received from the DNS resolver and returns the IP address to the resolver.
* **Resource records** — Data elements that define the basic structure and content of the DNS.

## DNS Server Types

* **Root Server** - It's contains information of all the TLDs and it's  server information like ``` .com .org .net .bd .edu .gov ``` etc. There are 13 root server with multiple instances of each server all over the world you'll find details from http://root-servers.org
* **TLDs Server** - Contains all the domains information in particular TLDs like Name Server of .bd hold all the .bd domains Name Server and IP addresses.
* **Name Server** - Contains all the zones of domains and resource database for each domain.
* **Recursion Server (Resolver)** - It's not contains any information for any domain but initiate DNS qeury request to complete DNS address resolve. It's maintain a cache database for fast serving.

