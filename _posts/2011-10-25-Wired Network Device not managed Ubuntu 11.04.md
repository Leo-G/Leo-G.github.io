This is a common problem in Ubuntu 11.04, I tried to connect via a wired cable and was stunned to find that there was no auto Etho :-)

After Googling for a while, I came across the solution below, hope it helps you guys

Solution is as follows
Step 1. Open the Network manager conf file


sudo gedit /etc/NetworkManager/NetworkManager.conf


Step 2
Change the below line to true



managed=false


Step 3
Reboot your System

 Visit blogadda.com to discover Indian blogs
