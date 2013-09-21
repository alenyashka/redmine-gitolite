redmine-gitolite
================

This is a redmine plugin that manages git repositories using gitolite.

Setup
-----

Check out the latest version of the plugin and make sure to use the right name.

    cd redmine/plugins
    git clone https://github.com/CtrlC-Root/redmine-gitolite.git redmine_gitolite
    cd redmine_gitolite

At this point you should probably modify init.rb to suit your needs. Then run
the migrate task.

    cd ../..
    RAILS_ENV=production rake redmine:plugins:migrate

The redmine user needs to be able to ssh into the gitolite server. This usually
involves setting up the ssh client for the redmine user.

    mkdir .ssh
    cp gitolite_admin_rsa .ssh/
    cp gitolite_admin_rsa.pus .ssh/
    chown -R redmine:redmine .ssh/

    su - redmine
    ssh gitolite_user@gitolite_server info

The command should return something like this if you have just set up gitolite.

    hello root, this is gitolite3@sandbox running gitolite3 3.5.2-1.el6 on git 1.7.1

     R W    gitolite-admin
     R W    testing

TODO: install the hook, configure git settings for redmine user
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
