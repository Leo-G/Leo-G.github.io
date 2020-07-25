For those of you who do not know, The Raspberry Pi is a small computer the size of a credit card, good for hobbyist and geeks.

Raspberry pi accessories

Raspberry PI element 14, crazypi.com

Raspberry Pi

It comes with two 2.0 USB ports, an RCA port, a HDMI port, 512MB RAM (model b), 700MHz Arm processor, a 3.5mm audio jack and a 5v micro USB port for power. Everything you need for a computer. :)

Raspberry Pi USB and LAN port
Raspberry Pi ports

Where do I buy a Rasberry Pi in India ?

I bought it from CrazyPi.com, they delivered it in a day, so I would go ahead and recommend buying it from them.

What about accessories, What accessories do I need to have in order to work with the Pi ?

A Computer Monitor or TV ( which has VGA or RCA port)

Raspberry Pi Monitor

USB Mouse, a KeyBoardand a Mobile charger for power.

Rasberry Pi USB port for Mouse

An External Memory card (Around 8GB and a card reader,

Raspberry Pi 8gb Sandisk Memory Card

RCA or HDMI to VGA cable,

Raspberry Pi RCA cable

A LAN cable or a Wifi Dongle

Raspberry Pi Lan cable

Optional An external storage Drive or pen drive
Raspberry Pi NAS

Raspberry Pi Accessories

If you don't have these Accessories, Then to save on Time and Money you can buy an SD card with the Noobs installer pre installed and connect to the pi through your laptop or pc remotely via an ssh connection.

What do I need to know in order to use the Raspberry Pi?

Basics about computers, Like RAM, CPU,SD card (Everything your smart phone has) and some basics about Linux. No programming required :)

I have got the Pi, How do I get started ?

Just like you need Windows or Ubuntu or Os X to get your home computers working you need an operating system to start the Pi., If you do not know how to install an operating system, not to worry, The Noobs installer is a software which let's you install an easy to use Rasbian Operating system. All you need to do is download,unzip and copy it to a flash memory card .

I have also created a video of my installation .



I installed Rasbian, Now what do I do with it?

Well there are a number of things you can do with your Raspberry Pi, But I decided to create a Network Attached Shared drive or NAS drive to back up all my data.

P.S. In order to connect your 1 TB portable hard drive you will need en external USB HUB.

You can use a Pen Drive as an alternative.

Raspberry Pi NAS drive

Steps for creating a Shared or NAS Drive

Step 1

Install packages Compatible with Windows NTFS and Apple File Systems

sudo apt-get update
sudo apt-get install ntfs-3g
sudo apt-get install hfsplus hfsutils

Step 2

Install Samba, the software that will convert our Raspberry Pi into a Shared NAS drive.

sudo apt-get install samba samba-common-bin

Step 3

Mounting your external Drive

First Find your drive using the FDISK command, In my case it was my 16gb Pendrive which has a device id /dev/sda1

fdisk -l

Raspberry Pi NAS drive

Then create a separate directory and run the mount command.Linux

mkdir /media/shares
mount -t auto /dev/sda1 /media/shares

Step 4

Configure SAMBA

Open the conf file and make the following changes

nano /etc/samba/smb.conf

Uncomment or change the following

security = user
map to guest = Bad User

Add the below configuration

[test] comment = Media share
path = /media/shares
read only = no
browseable = yes
guest ok = yes
writeable= yes
force group = users
create mask = 0664
force directory mode = 2775

This will create a password less shared folder called test, which you should be able to access.

Raspberry Pi NAS drive

Rasberry Pi NAS drive

 

And yes if you need to select a memory card for your Raspberry Pi then I recommend you read  my article Strotium vs Sandisk

You can even install a VPN server on your PI and transform your Raspberry PI into your very own Secure VPN server.

Feel free to comment if you have bought the Raspberry Pi and done something cool with it or have any query or doubt with regards to buying it.

 

source [http://www.samba.org/samba/docs/man/manpages-3/smb.conf.5.html]
