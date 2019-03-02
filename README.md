# A Virtual Machine for Ruby on Rails Core Development

## Introduction

**Please note this VM is not designed for Rails application development, only Rails core development.**

This project automates the setup of a development environment for working on Ruby on Rails itself. Use this virtual machine to work on a pull request with everything ready to hack and run the test suites.

## Requirements

* [VirtualBox](https://www.virtualbox.org)

* [Vagrant 2](http://vagrantup.com)

* [Oracle Database Express Edition (XE) Release 18.4.0.0.0 (18c) "oracle-database-xe-18c-1.0-1.x86_64.rpm"](https://www.oracle.com/technetwork/database/database-technologies/express-edition/downloads/index.html)

* [Version 18.3.0.0.0 Basic Package - All files required to run OCI, OCCI, and JDBC-OCI applications "oracle-instantclient18.3-basic-18.3.0.0.0-3.x86_64.rpm"](https://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)

* [Version 18.3.0.0.0 SQL\*Plus Package - The SQL\*Plus command line tool for SQL and PL/SQL queries "oracle-instantclient18.3-sqlplus-18.3.0.0.0-3.x86_64.rpm"](https://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)

* [Version 18.3.0.0.0 SDK Package - Additional header files and an example makefile for developing Oracle applications with Instant Client "oracle-instantclient18.3-devel-18.3.0.0.0-3.x86_64.rpm"](https://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)


## How To Build The Virtual Machine

Building the virtual machine is this easy:

    host $ git clone -b runs_oracle_18c_on_docker https://github.com/yahonda/rails-dev-box.git
    host $ cd rails-dev-box
    host $ cp /path/to/oracle-database-xe-18c-1.0-1.x86_64.rpm .
    host $ cp /path/to/oracle-instantclient18.3-basic-18.3.0.0.0-3.x86_64.rpm  .
    host $ cp /path/to/oracle-instantclient18.3-sqlplus-18.3.0.0.0-3.x86_64.rpm .
    host $ cp /path/to/oracle-instantclient18.3-devel-18.3.0.0.0-3.x86_64.rpm .
    host $ vagrant up

That's it.

After the installation has finished, you can access the virtual machine with

    host $ vagrant ssh
    Welcome to Ubuntu 18.10 (GNU/Linux 4.18.0-10-generic x86_64)
    ...
    vagrant@rails-dev-box:~$

Port 3000 in the host computer is forwarded to port 3000 in the virtual machine. Thus, applications running in the virtual machine can be accessed via localhost:3000 in the host computer. Be sure the web server is bound to the IP 0.0.0.0, instead of 127.0.0.1, so it can access all interfaces:

    bin/rails server -b 0.0.0.0

## RAM and CPUs

By default, the VM launches with 2 GB of RAM and 2 CPUs.

These can be overridden by setting the environment variables `RAILS_DEV_BOX_RAM` and `RAILS_DEV_BOX_CPUS`, respectively. Settings on VM creation don't matter, the environment variables are checked each time the VM boots.

`RAILS_DEV_BOX_RAM` has to be expressed in megabytes, so configure 4096 if you want the VM to have 4 GB of RAM.

## What's In The Box

* Development tools

* Git

* Ruby 2.5

* Bundler

* SQLite3, MySQL, and Postgres

* Databases and users needed to run the Active Record test suite

* System dependencies for `nokogiri`, `sqlite3`, `mysql2`, and `pg`

* Memcached

* Redis

* RabbitMQ

* An ExecJS runtime

## Recommended Workflow

The recommended workflow is

* edit in the host computer and

* test within the virtual machine.

Just clone your Rails fork into the rails-dev-box directory on the host computer:

    host $ ls
    bootstrap.sh MIT-LICENSE README.md Vagrantfile
    host $ git clone git@github.com:<your username>/rails.git

Vagrant mounts that directory as _/vagrant_ within the virtual machine:

    vagrant@rails-dev-box:~$ ls /vagrant
    bootstrap.sh MIT-LICENSE rails README.md Vagrantfile

Install gem dependencies in there:

    vagrant@rails-dev-box:~$ cd /vagrant/rails
    vagrant@rails-dev-box:/vagrant/rails$ bundle

We are ready to go to edit in the host, and test in the virtual machine.

Please have a look at the [Contributing to Ruby on Rails](http://edgeguides.rubyonrails.org/contributing_to_ruby_on_rails.html) guide for tips on how to run test suites, how to generate an application that uses your local checkout of Rails, etc.

This workflow is convenient because in the host computer you normally have your editor of choice fine-tuned, Git configured, and SSH keys in place.

## Virtual Machine Management

When done just log out with `^D` and suspend the virtual machine

    host $ vagrant suspend

then, resume to hack again

    host $ vagrant resume

Run

    host $ vagrant halt

to shutdown the virtual machine, and

    host $ vagrant up

to boot it again.

You can find out the state of a virtual machine anytime by invoking

    host $ vagrant status

Finally, to completely wipe the virtual machine from the disk **destroying all its contents**:

    host $ vagrant destroy # DANGER: all is gone

Please check the [Vagrant documentation](http://docs.vagrantup.com/v2/) for more information on Vagrant.

## Faster Rails test suites

The default mechanism for sharing folders is convenient and works out the box in
all Vagrant versions, but there are a couple of alternatives that are more
performant.

### rsync

Vagrant 1.5 implements a [sharing mechanism based on rsync](https://www.vagrantup.com/blog/feature-preview-vagrant-1-5-rsync.html)
that dramatically improves read/write because files are actually stored in the
guest. Just throw

    config.vm.synced_folder '.', '/vagrant', type: 'rsync'

to the _Vagrantfile_ and either rsync manually with

    vagrant rsync

or run

    vagrant rsync-auto

for automatic syncs. See the post linked above for details.

### NFS

If you're using Mac OS X or Linux you can increase the speed of Rails test suites with Vagrant's NFS synced folders.

With an NFS server installed (already installed on Mac OS X), add the following to the Vagrantfile:

    config.vm.synced_folder '.', '/vagrant', type: 'nfs'
    config.vm.network 'private_network', ip: '192.168.50.4' # ensure this is available

Then

    host $ vagrant up

Please check the Vagrant documentation on [NFS synced folders](http://docs.vagrantup.com/v2/synced-folders/nfs.html) for more information.

## Troubleshooting

On `vagrant up`, it's possible to get this error message:

```
The box 'ubuntu/yakkety64' could not be found or
could not be accessed in the remote catalog. If this is a private
box on HashiCorp's Atlas, please verify you're logged in via
vagrant login. Also, please double-check the name. The expanded
URL and error message are shown below:

URL: ["https://atlas.hashicorp.com/ubuntu/yakkety64"]
Error:
```

And a known work-around (https://github.com/Varying-Vagrant-Vagrants/VVV/issues/354) can be:

    sudo rm /opt/vagrant/embedded/bin/curl

## License

Released under the MIT License, Copyright (c) 2012–<i>ω</i> Xavier Noria.
