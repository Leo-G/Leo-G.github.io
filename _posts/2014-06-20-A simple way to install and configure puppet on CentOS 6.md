Puppet is an automation tool which allows you to automate the configuration of software like apache and nginx across multiple servers.

Puppet installation
In this tutorial we will be installing Puppet in the Puppet/Agent mode.You can install it in a Stand Alone mode as well.

OS & software Versions
Centos 6.5
Linux kernel 2.6.32
Puppet 3.6.2

Let’s get to it then.

Puppet server configuration

#Add Puppet repos 
[user@puppet ~]# sudo rpm -ivh http://yum.puppetlabs.com/puppetlabs-release-el-6.noarch.rpm
[user@puppet ~]# sudo yum install puppet-server
# Add your puppet server hostnames to the conf file under the [main] section
[user@puppet ~]# sudo vim /etc/puppet/puppet.conf
dns_alt_names = puppet,puppet.yourserver.com
[user@puppet ~]# sudo service puppetmaster start 

Puppet listens on port no 8140, ensure to unblock it in CSF or your firewall.
Puppet client configuration

#Add Puppet repos 
[user@client ~]# sudo rpm -ivh http://yum.puppetlabs.com/puppetlabs-release-el-6.noarch.rpm
[user@client ~]# sudo yum install puppet
#Open the conf file and add the puppet server hostname 
[user@client ~]#sudo vim /etc/puppet/puppet.conf
[main]
# The puppetmaster server
server=puppet.yourserver.com
[user@client ~]# sudo service puppet start

In the log file you should see the following lines.

info: Creating a new SSL key for vps.client.com
warning: peer certificate won't be verified in this SSL session
info: Caching certificate for ca
warning: peer certificate won't be verified in this SSL session
warning: peer certificate won't be verified in this SSL session
info: Creating a new SSL certificate request for agent1.localdomain
info: Certificate Request fingerprint (md5): FD:E7:41:C9:5C:B7:5C:27:11:0C:8F:9C:1D:F6:F9:46
warning: peer certificate won't be verified in this SSL session
warning: peer certificate won't be verified in this SSL session
warning: peer certificate won't be verified in this SSL session
Exiting; no certificate found and waitforcert is disabled

Puppet uses SSL to communicate with it’s clients, when you start puppet on a client, it will automatically connect to the puppet server in it’s conf file and request for it’s certificate to be signed.

On the puppet server run

[user@puppet ~]# sudo  puppet cert list
vps.client.com (FD:E7:41:C9:2C:B7:5C:27:11:0C:8F:9C:1D:F6:F9:46)
[user@puppet ~]# sudo puppet cert sign vps.client.com
notice: Signed certificate request for vps.client.com
notice: Removing file Puppet::SSL::CertificateRequest vps.client.com at '/etc/puppetlabs/puppet/ssl/ca/requests/vps.client.pem'
Now our client server “vps.client.com” is authorized to fetch and apply configurations from the puppet server. To understand how puppet ssl works and to troubleshoot any issues you can read http://docs.puppetlabs.com/learning/agent_master_basic.html

Let’s look at a sample puppet configuration.

Installing apache web server with puppet
Although puppet server configuration is stored in “/etc/puppet/puppet.conf”, client configurations are stored in files called manifests.

#On the puppet server run
[user@puppet ~]# sudo vim /etc/puppet/manifests/site.pp
node ‘vps.client.com’ {
 
 package { ‘httpd’ :
 ensure => installed,
 }
}
The configuration is pretty self explanatory, the first line indicates that we need to install this configuration on a client machine with the hostname ‘vps.client.com’. If you want to apply the configuration to the puppet server then replace ‘vps.client.com’ with ‘default’ .
Read node definitions for multiple node configurations.

The next two lines tell puppet that we need to ensure that the apache web server is installed. Puppet will check if apache is installed and if not, install it.

Think of a “package” as an object, “httpd” as the name of the object and “ensure => present” as the action to be performed on the object.

So if I wanted puppet to install a mysql database server, the configuration would be

node ‘vps.client.com’ {
package { ‘mysql-server’ :
ensure => installed,
}
}
The puppet server will compile this configuration into a catalog and serve it to a client when a request is sent to it.

How do I pull my configuration to a client immediately?
Puppet client’s usually pull configuration once every 30 minutes, But you can pull a configuration immediately buy running “service puppet restart or the following command.

[user@puppet ~]# sudo puppet agent --test

What if I wanted puppet to add a user ‘Tom’?
Then the object would be user, the name of the object would be ‘tom’ and the action would be ‘present’.

node ‘vps.client.com’ {
             
              user { ‘tomr’ :
                     ensure => present,
                           }
}

In puppet terms, these objects are known as Resources, the name of the objects are Titles and the actions are called Attributes.

Puppet has a number of these resources to help ease your automation, You can read about them at http://docs.puppetlabs.com/references/latest/type.html

How to ensure a service is running with puppet?
Once you have package like apache installed, you will want to ensure that it is running. On the command line you can do this with the service command, However in puppet you will need to use the manifest file and add the configuration as follows.

node ‘vps.client.com’ {
             
              package { ‘httpd’ :  
                     ensure => installed, 
                           }
             ->
             service { ‘httpd’ :  #Our resource and it’s title
                     ensure => running,  #Action to be performed on resource or attribute
                     enable     => true,   # Start apache at boot
}
}
Now you must have noticed I have added an “->” symbol. This is because Puppet is not particular about ordering, But we want the service command to run only after apache is installed and not before, hence I have added the arrow symbol which tells Puppet to run only after “httpd” is installed.
To know more about puppet ordering read.

How to automate installation of predefined conf files?
You may want to have a customised apache conf file for this client, which will have the vhost entry and other specific parameters you choose. In this case we need to use the file resource.

Before we go into the configuration, you should know how puppet serves files. A Puppet server provides access to custom files via mount points. One such mount point by default is the modules directory.
The modules directory is where you would add your modules. Modules make it easier to reuse configurations, rather than having to write configurations for every node we can store them as a module and call them whenever we like.

In order to write a module, you need to create a subdirectory inside the modules directory with the module name and create a manifest file called init.pp which should contain a class with the same name as the subdirectory.

[user@puppet ~]# cd /etc/puppet/modules
[user@puppet ~]# mkdir httpd
[user@puppet ~]# mkdir -p httpd/manifests httpd/files
[user@puppet ~]# vim httpd/manifests/init.pp
class httpd { #Same name as our Sub Directory
package { 'httpd':
 ensure => present,
}
 ->
file {'/etc/httpd/conf/httpd.conf': #Path to file on the client we want puppet to administer
 ensure => file, #Ensure it is a file, 
 mode => 0644, #Permissions for the file
 source => 'puppet:///modules/httpd/httpd.conf', #Path to our customised file on the puppet server
 }
->
service { 'httpd':
 ensure => running,
 enable => true,
 subscribe => File['/etc/httpd/conf/httpd.conf'] # Restart service if any any change is made to httpd.conf
}
}
You need to add your custom httpd.conf file in the files subdirectory located at “/etc/puppet/modules/httpd/files/”

To understand the how the URI to the source attribute works read http://docs.puppetlabs.com/guides/file_serving.html

Now call the module in our main manifest file.

[user@puppet ~]#sudo vim /etc/puppet/manifests/site.pp
node ‘vps.client.com’ {
 
 include httpd
}
Incase you need a Web interface to  Manage your Linux Servers then read my tutorial Using Foreman, an Opensource Frontend for Puppet

Update: For more Automation and other System Administration/Devops Guides see https://github.com/Leo-G/DevopsWiki
Puppet FAQ
How do I change the time interval for a client to fetch it’s configuration from the server ?
Add “runinterval = 3600 “ under [main] section in “/etc/puppet/puppet.conf” on the client.

Time is in seconds.

How do I install modules from puppet forge?

[user@puppet ~]#sudo puppet module install "full module name"
#Example
[user@puppet ~]#sudo puppet module install puppetlabs-mysql
read more here and for publishing your own modules read http://docs.puppetlabs.com/puppet/latest/reference/modules_publishing.html
