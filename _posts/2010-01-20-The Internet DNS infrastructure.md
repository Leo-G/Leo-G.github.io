The DNS structure on the Internet actually works like this.

There are computers that service requests for IP numbers--millions of them. They are called name servers (or DNS servers), and a request is called a DNS lookup (or just a lookup). However, each name server only has information about a specific part of the Internet, and they constantly query each other.

There are 13 root name servers on the Internet.

a.root-servers.net 198.41.0.4
b.root-servers.net 128.9.0.107
c.root-servers.net 192.33.4.12
d.root-servers.net 128.8.10.90
e.root-servers.net 192.203.230.10
f.root-servers.net 192.5.5.241
g.root-servers.net 192.112.36.4
h.root-servers.net 128.63.2.53
i.root-servers.net 192.36.148.17
j.root-servers.net 198.41.0.10
k.root-servers.net 193.0.14.129
l.root-servers.net 198.32.64.12
m.root-servers.net 202.12.27.33

Each country also has a name server, and in turn each organization has a name server. Each name server only has information about machines in its own domain, as well as information about other name servers. The root name servers only have information on the IP addresses of the name servers of .com, .edu, .za, etc. The .za name server only has information on the IP addresses of the name servers of .org.za, .ac.za, .co.za, etc. The .co.za name server only has information on the name servers of all South African companies, like .cranzgot.co.za, .icon.co.za, .mweb.co.za, etc. The .cranzgot.co.za, name server only has info on the machines at Cranzgot Systems, like www.cranzgot.co.za.

Your own machine will defined in its configuration files a name server that is geographically close to it. The responsibilities of this name server will be to directly answer any queries about its own domain that it has information about and to answer any other queries by querying as many other name servers on the Internet as is necessary.
