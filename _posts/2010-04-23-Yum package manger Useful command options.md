# yum check-update
This allows you to check to see if there is any update available for your installed packages.

# yum update
This will update all packages to the latest version.

# yum update
Use this to just update one package at a time.

# yum search
You can use this command to search for any packages containing specific keyword. Pretty useful to locate the full name of a particular package.

# yum install
This allows you to install the latest version of a package. Yum will automatically install all dependencies.

# yum remove
Used to delete/remove a package, along with its dependencies if any.

# yum localinstall
With “yum install”, yum automatically downloads the package for you. In case you already downloaded the package yourself, you can use this command to install it. Make sure to state the full path to the package you downloaded. i.e. yum localinstall /home/test.tar.gz

# yum info
Lists information about a package.
