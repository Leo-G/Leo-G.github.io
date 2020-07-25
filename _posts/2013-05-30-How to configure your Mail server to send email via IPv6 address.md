How to configure your Mail server to send email via IPv6 address

I recently decided to send mail via an IPv6 address for the Following reasons

1) We will soon need to start using IPv6 addresses as we have run out of IPv4

2) How will Anti-spam techniques like DNS black list work incase of IPv6

First, What is IPv6?

IPv6 is a 128 bit Hexadecimal address for example 2605:ea00:1:1::31e4:807e

How to configure Postfix to send mail via Postfix?

There are two settings you need to change in /etc/postfix/main.cf:

# You must stop/start Postfix after changing this parameter.
inet_protocols = ipv4 (DEFAULT: enable IPv4 only)
inet_protocols = all (enable IPv4, and IPv6 if supported)
inet_protocols = ipv4, ipv6 (enable both IPv4 and IPv6)
inet_protocols = ipv6 (enable IPv6 only)

Change this to

inet_protocols = all

And add

smtp_bind_address6 = 2605:ea00:1:1::31e4:807e (No 6 is not a mistake)

Also if you have specified mynetworks parameter value in main.cf, then you need to add it there

mynetworks = 127.0.0.0/8 168.100.189.0/28 [::1]/128 [fe80::]/10 [2605:ea00:1:1::31e4:807e]/64

NOTE: when configuring Postfix match lists such as mynetworks or debug_peer_list, you must specify IPv6 address information inside "[]" in the main.cf parameter value and in files specified with a "/file/name" pattern. IPv6 addresses contain the ":" character, and would otherwise be confused with a " type:table" pattern.

Currently There is No Blacklisting for IPv6.

Source
[
linuxtopia
IPv4 Subnetting refer http://en.wikipedia.org/wiki/IPv4_subnetting_reference
http://www.microbuilder.eu/Tutorials/Fundamentals/Hexadecimal.aspx
http://www.pantz.org/software/tcpip/subnetchart.html
]
