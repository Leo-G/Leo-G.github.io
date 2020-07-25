Now you have installed fedora, what next?
Well you can install the below packages to make more interesting and fun.

Yum 
Yum is an interactive tool or automated update program which can be used for maintaining systems using rpm (read as RedHat and friends).

Fastest mirror plugin. This will look for the fastest mirror when updating yum.



yum install yum-fastestmirror


Presto Plugin. This will save you downloading a full upgrade of 551mb down to 100mb.



yum install yum-presto


Enable Additional repositories (rpm fusion).



rpm -Uvh http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-stable.noarch.rpm 
rpm -Uvh http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-stable.noarch.rpm


Enable Yakuake ,mc,pavucontrol, htop, unrar, checkgmail
* Yakuake - is a KDE terminal emulator. Its design was inspired from consoles in computer games such as Quake which slide down from the top. I like it.
* mc - GNU Midnight Commander is a visual file manager used in terminal/yakuake.
* Pavucontrol - PulseAudio Volume Control (pavucontrol) is a simple GTK based volume control tool ("mixer") for the PulseAudio sound server. Skype mic refused to work and I had to use this tool.
* htop - an interactive process viewer for Linux.
* unrar - unpack RAR archives
* checkgmail - says it all. Alerts you when you have mail in your gmail account.



yum install yakuake mc pavucontrol htop unrar checkgmail


To play mp3's and videos type



yum install mplayer smplayer vlc mencoder gstreamer-plugins-ugly gstreamer-plugins-bad gstreamer-ffmpeg xine-lib-extras xine-lib-extras-freeworld


Install the Adobe repository for yum:



rpm -Uvh http://linuxdownload.adobe.com/adobe-release/adobe-release-i386-1.0-1.noarch.rpm

Type:



yum install flash-plugin nspluginwrapper.{i386,x86_64} pulseaudio-libs.i386 alsa-plugins-pulseaudio.i386 libcurl.i386
Java


yum install java-1.6.0-openjdk-plugin


To downgrade from a current version of the package


yum install yum-allowdowngrade


To use it type:



su -c 'yum update --allow-downgrade'


You can disable this plugin by changing the option enabled in the file:



su -c 'gedit /etc/yum/pluginconf.d/allowdowngrade.conf'


Wine to run windows apps


yum install wine
Developer tools
It allows you to install single package or group of package. For example let us say you are running a web server with PHP+MySQL. Now you need complete development environment i.e gcc C/C++ compilers, subversion cvs, perl and related libraries. You can install them in one shot with the following command:


 yum groupinstall 'Development Tools'


You can remove all developer tools with the following command:


yum groupremove 'Development Tools'


Or better update all tools to latest version:


yum groupupdate 'Development Tools'


You can obtain list of all groups with the following command:


yum grouplist | less


Install the below groups if you are a developer


yum groupinstall "Development Libraries" "Development Tools" Java "Fedora Eclipse" "Java Development" "KDE (K Desktop Environment)"
