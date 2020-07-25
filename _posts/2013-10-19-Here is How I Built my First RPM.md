I was building a rpmpackage for Tengine the Dynamic Module loading Nginx fork. As usual since there was a no decent tutorial I decided to write my own.

rpmbuild
Warning: DO NOT try as root for obvious Reasons

First Install the necessary packages.

sudo yum install rpm-build
sudo yum install redhat-rpm-config
Create the rpmbuild directories

[userid@hostname ~]$ mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
<!> Beware: this next command will overwrite an existing .rpmmacros file if it exists, so check that you don't already have one before continuing.
[userid@hostname ~]$ echo '%_topdir %(echo $HOME)/rpmbuild' > ~/.rpmmacros
Explanation of these directories as per rpm.org is as follows.

/usr/src/redhat/SOURCES — Contains the original sources, patches, and icon files.

/usr/src/redhat/SPECS — Contains the spec files used to control the build process.

/usr/src/redhat/BUILD — The directory in which the sources are unpacked, and the software is built.

/usr/src/redhat/RPMS — Contains the binary package files created by the build process.

/usr/src/redhat/SRPMS — Contains the source package files created by the build process.

Add the source or TAR file in the SOURCES directory.

Goto the SPEC directory to create your spec file.

The spec file is where you will need to add all the details of the package that needs to be installed.
From the files that need to be installed to the version of your package.
When you are creating a SPEC file for the first time, vim or emacs will automatically create a template for you:

vim tengine.spec
Below is the template

Name: Tengine
Version:  1.5.1
Release:        1%{?dist}
Summary:       Tengine web server forked out of Nginx
Group: Applications/Internet
License: open BSD license
URL: http://tengine.taobao.org/download.html
Source0: tengine-1.5.1.tar.gz
#BuildRequires:
#Requires:
%description
Tengine by taboa which enables dso support for nginx
%prep
%setup -n tengine-1.5.1
%build
%_configure
make %{?_smp_mflags}
%install
rm -rf $RPM_BUILD_ROOT
make install DESTDIR=$RPM_BUILD_ROOT
%clean
rm -rf $RPM_BUILD_ROOT
%files
%defattr(-,root,root,-)
%doc LICENSE
%doc README
%config(noreplace) /usr/local/nginx/conf/*
/usr/local/nginx/html/
/usr/local/nginx/sbin/nginx
/usr/local/nginx/sbin/dso_tool
%changelog
The First part which is your name version etc is pretty much self explanatory I am not going to go into it

Anything with % before is a a Macros. Macros can be used to set a variable, and there are a few that are already set. To see these macros look in /usr/lib/rpm/macros or /usr/share/doc/rpm-[version]/macros, where [version] is the version of rpm. The value of the macro is returned by putting the name of the macro in curly braces: %{ }.



%description : Add a short description of your package.
%prep : Here is where you file will be untared Just use %setup -q here and it will untar it.
%build : This is where you file is build, %_configure will run ./configure in tengine directory and you need to add "make %{?_smp_mflags} " to create a make file.
%install : Where your software is installed with make install
%files : The list of files that will be installed.


Once you have your spec file is ready run the below command

rpmbuild -ba tengine.spec

This will generate the rpm file in the RPMS directory

I have added my rpm on GitHUB. Feel free to fork and would love here inputs from others on this.

If you have a small shell script that you would like to manage via rpm then you can check out rpmwand. Just remember that this is for those packages which do not have a compile process.

Another good tutorial that will help you with Linux system administration is How to create Init scripts

Update : The Good Folks at Reddit suggested fpm you may want to check that out as well.

Update : For a list of Linux System Administration and Programming guides please see https://github.com/Leo-G/DevopsWiki

 

 

Images are not mine and are found on the internet.

Source

http://rpmbuildtut.wordpress.com/getting-started/

http://fedoraproject.org/wiki/How_to_create_an_RPM_package
