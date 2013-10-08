redmine-gitolite
================

This is a redmine plugin that manages git repositories using gitolite. The
plugin has been tested to work with redmine 2.2.1 and 2.3.3 using gitolite2 and
gitolite3.

Requirements
------------

This section is a work in progress and bound to change.

 * rubygems(net-ssh)
 * rubygems(gitolite) = 0.0.3.alpha
 * rubygems(lockfile)
 * redmine >= 2.0.0
 * gitolite or gitolite3
 * curl

Setup
-----

Start by logging in to the root user and store the redmine root folder in an
environment variable for eacy access.

    $ su
    $ whoami
    root
    $ RM=/usr/share/redmine
    $ RMP=$RM/plugins/redmine_gitolite

Check out the latest version of the plugin and install the needed ruby gems.

    $ cd $RM/plugins
    $ git clone https://github.com/CtrlC-Root/redmine-gitolite.git redmine_gitolite
    $ cd redmine_gitolite
    $ bundle install

At this point you should probably modify init.rb to suit your needs. Then run
the migrate task.

    $ cd $RM
    $ RAILS_ENV=production rake redmine:plugins:migrate

The redmine user needs to be able to ssh into the gitolite server. This usually
involves setting up the ssh client for the redmine user.

    $ mkdir .ssh
    $ cp gitolite_admin_rsa .ssh/id_rsa
    $ cp gitolite_admin_rsa.pub .ssh/id_rsa.pub
    $ chmod go-rwx .ssh
    $ chown -R redmine:redmine .ssh/

Make sure to log in to the gitolite server at least once as the redmine user to
accept the server's host key.

    $ su - redmine
    $ ssh gitolite_user@gitolite_server info

The command should return something like this if you have just set up gitolite.

    hello root, this is gitolite3@sandbox running gitolite3 3.5.2-1.el6 on git 1.7.1

     R W    gitolite-admin
     R W    testing

Configure global git settings for the redmine user.

    $ git config --global user.name "redmine"
    $ git config --global user.email "redmine@example.com"

Next we need to install the post-receive hook. Log out of the redmine user and
copy the hook from the plugin's contrib folder to the appropriate place.

    $ exit
    $ whoami
    root

    $ cd /var/lib/gitolite3/.gitolite/hooks/common
    $ cp $RMP/contrib/post-receive .
    $ chown gitolite3:gitolite3 post-receive
    $ chmod a+rx post-receive

Modify the post-receive hook with the generated API key and the redmine server
url. If your server is using https but you don't have a valid certificate
(maybe this is a test server) then make sure to set the curl security flag to
false. Go to the gitolite user's home folder and mofity the RC file to allow
redmine to configure git settings through gitolite.conf.

    $ cd /var/lib/gitolite3
    $ cat .gitolite.rc
    ...
        # look for "git-config" in the documentation
        GIT_CONFIG_KEYS =>  'hooks\.redmine_gitolite\..*',
    ...

Run the gitolite setup command.

    $ su - gitolite
    $ gitolite setup
    $ exit

TODO: what about needing the ssh key to be named 'redmine'??

History
-------

There seem to be a million versions of every redmine gitolite and git hosting
plugin in existence. This can make setting up a redmine instance rather
difficult. Not only are there differing versions of the plugins but a plethora
of small bugfix branches for each version. This plugin is an attempt to pull
most of those together.

There seem to be two larger camps when it comes to gitolite integration: those
based on redmine-gitolite and those based on redmine-git-hosting. After some
careful thought (how many plugins of this type work / how many are there total)
I decided to fork redmine-gitolite. I started by merging the biggest offshoots
first (everyone can give their thanks to gdoffe for doing most of the work) and
then slowly merged the smaller bug fix branches.
