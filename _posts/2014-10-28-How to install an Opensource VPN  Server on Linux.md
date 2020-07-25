One of the most concerning thoughts I have while browsing, Is how can I ensure that my data remains private and secure ? In my search for answers, I came cross a number of ways in which you can remain anonymous like using a proxy website. But still using a third party service was not assuring enough. What I needed was a software which could be installed and run by me thus ensuring that I and only I would have access to the data.

So what is such a software called?

It's called a VPN or short for Virtual Private Network. It allows you to encrypt your data when you connect through it.

In this Linux Tutorial , I will show you how you can install an OpenVPN Access Server on Linux.

Step 1. Install OpenVPN Access Server on your Ubuntu 14.04 or CentOS 7 Linux Server or VPS.

Download the package from https://openvpn.net/index.php/access-server/download-openvpn-as-sw.html, Ubuntu users can download appropriate packages and install OpenVPN Access Server on Ubuntu and Ubuntu based distros like raspbian as well.

Ubuntu 14.04 steps

[leo@vps ]$ cd /tmp
[leo@vps tmp]$ wget http://swupdate.openvpn.org/as/openvpn-as-2.0.25-Ubuntu14.amd_64.deb
[leo@vps tmp]$ sudo dpkg -i openvpn-as-2.0.25-Ubuntu14.amd_64.deb
Preparing to unpack openvpn-as-2.0.25-Ubuntu14.amd_64.deb ...
Unpacking openvpn-as (2.0.25-Ubuntu14) ...
Setting up openvpn-as (2.0.25-Ubuntu14) ...
The Access Server has been successfully installed in /usr/local/openvpn_as
Configuration log file has been written to /usr/local/openvpn_as/init.log
Please enter "passwd openvpn" to set the initial
administrative password, then login as "openvpn" to continue
configuration here: https://2.2.18.72:943/admin
To reconfigure manually, use the /usr/local/openvpn_as/bin/ovpn-init tool.
Access Server web UIs are available here:
Admin UI: https://2.2.18.72:943/admin
Client UI: https://2.2.18.72:943/

CentOS 7 steps

[leo@vps ]$ cd /tmp
[leo@vps tmp]$ wget http://swupdate.openvpn.org/as/openvpn-as-2.0.10-CentOS7.x86_64.rpm
[leo@vps tmp]$ sudo rpm -Uvh openvpn-as-2.0.10-CentOS7.x86_64.rpm
Preparing...                          ################################# [100%]
Updating / installing...
   1:openvpn-as-0:2.0.10-CentOSrelease################################# [100%]
The Access Server has been successfully installed in /usr/local/openvpn_as
Configuration log file has been written to /usr/local/openvpn_as/init.log
Please enter "passwd openvpn" to set the initial
administrative password, then login as "openvpn" to continue
configuration here: https://yourhostIP:943/admin
To reconfigure manually, use the /usr/local/openvpn_as/bin/ovpn-init tool.
Access Server web UIs are available here:
Admin  UI: https://yourhostIP:943/admin
Client UI: https://yourhostIP:943/

Step 2: Setup a password for the OpenVPN admin account

[leo_g@vps ]$passwd openvpn

Step 3: Log into the server via the admin Url https://yourhostIP:943/admin and start the server.
Install OpenVPN on Cent OS 7

Type in the username "openvpn" and the password you set earlier and click on "Start the Server" if the server is stopped.
Install OpenVPN on CentOS 7

Step 4: Install the  the OpenVPN Client on your PC

Steps for downloading and installing the client are available at https://openvpn.net/index.php/access-server/docs/admin-guides-sp-859543150/howto-connect-client-configuration.html
How to Install OpenVPN on CentOS 7

.

Step 5: Connect to your server with your username and password that you created earlier.

Install OpenVPN on CentOS 7

That's it, you can goto whatismyip.com and check your IP, should be that of your CentOS 7 or Ubuntu 14.04 OpenVPN Access Server.

Note: OpenVPN gives only 2 free user accounts, if you need more you will need to purchase a License.

To configure your Firewall to accept OpenVPN connections please see  Firewall Configuration for Linux Servers

Update: For a list of Linux Administration and Developer Tutorials see https://github.com/Leo-G/DevopsWiki
