Git is a version control system used for the up keep of your software. It's sort of like a back up of your software which you can revert to incase of any errors or incorrect code change

I am documenting a few Git commands that can be used with any Linux Distro.

Installing Git on CentOS Linux

Use this script to add Repository

 yum install git

Configure your User name and email, every Git commit will include this information, this is useful when multiple users commit to the same Git reporsitory.

$ git config --global user.name "Leo G"
$ git config --global user.email leo@leog.in

Check if everything is ok

#git config -l

Initialize the Repository

#git init

This will initialize an empty repository and create a folder called .git which contains all configuration files.

Create an Empty file

touch test1

Add the file to Git

git add test1

If you would like to add multiple files including sub directory then add a '.'

git add .


 git status 
# On branch master
#
# Initial commit
#
# Changes to be committed:
# (use "git rm --cached ..." to unstage)
#
# new file: test1
Git status will give you an idea of what changes are going to be committed.

Once your sure commit the file or remove it with

 git rm --cached


git commit -m "Initial"

Now you can also upload your files and keep a back up of your files on GitHub.com.

First create a profile on GitHub.com and then add your ssh keys.

Create a new empty repository, do not initialize it with readme or add .gitignore file.

Once you initialize the repository you will get instructions on how to sync you files.

Else you can typoe the below command, here rails was the empty repository I created on git hub.

#git remote add origin git@github.com:Leo-g/rails.git
#git push -u origin master
The authenticity of host 'github.com (192.30.252.131)' can't be established.
RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,192.30.252.131' (RSA) to the list of known hosts.
Counting objects: 120, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (108/108), done.
Writing objects: 100% (120/120), 280.00 KiB, done.
Total 120 (delta 7), reused 0 (delta 0)
To git@github.com:Leo-g/rails.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.

This will add the remote github repository and when you type "git push" from now on sync changes with github.

Useful git commands.

To check a history of your commits use

git log

To delete remote origin use


git remote rm origin

Incase you delete the file by accident and have not yet committed the changes


git checkout -f

Will recover the file.

If you want to push changes to Add an existing project to Github then read, Just remmber to pull before you push else you will get a "non-fast-forward" error.

source [http://ruby.railstutorial.org/ruby-on-rails-tutorial-book#sec-1_3_1_1]
