
MariaDB is a community-developed fork of the MySQL intended to remain free under the GNU GPL. It is led by the original developers of MySQL, who forked it due to concerns over its acquisition by Oracle.

So what is the difference  between MySQl and MariaDB?

Well besides  the fact that it is completely opensourced, It also supports a number of Storage Engines including Percona's famous Xtradb.

How do I migrate from MySQL to MariaDB?

It is not that difficult to migrate since mariadb uses almost the same source code.

The below steps are for Red Hat, Centos and Fedora for Ubuntu or Debian please use apt-get.

First we need to take a backup of our MySQL Database with the following command


mysqldump -u root -p --all-databases > mysqlbackup.sql
Then backup our my.cnf file


cp -a /etc/my.cnf my.bak

Now stop mysql server and remove mysql install


service mysqld stop
yum remove mysql mysql-server mysql-libs
Now you need atomic repo or mariadb repo 

I prefer atomic so as it has the latest php software as well


wget -q -O - http://www.atomicorp.com/installers/atomic | sh
yum install mariadb-server
This will install the client and libraries with the one command.

Copy the my.bak to mariadb conf file.


cp my.bak /etc/my.cnf
service mysqld start
mysql -u root -p -Be 'show databases'
enter password
Should show you all the databases. I also had my Replication up an running which was great.

Update: Incase you get a mysql-libs error as follows


Error: mariadb-libs conflicts with mysql-libs
Then uninstall mysql-libs with the yum command below and then try installing again.


yum remove mysql-lib
Please ensure you be careful while doing this and this post assumes you know what you are doing and take full responsibility for the same
