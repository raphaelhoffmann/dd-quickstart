# DeepDive on Impala

Beginning with version 0.6, DeepDive supports Impala as an alternative
to Postgres, Greenplum, and MySQL. Impala reduces load times and allows 
DeepDive to take better advantage of the Hadoop eco-system.

In this tutorial, we show how to set up DeepDive with Impala for
development. 

We assume that the development environment is OSX; steps for Windows are
similar, and setting up a Linux development environment is far easier, as
it doesn't require a virtual machine.

Impala can only be run on Linux, and so we will run Impala on a Linux
virtual machine hosted on OSX. We will also run DeepDive on the same virtual machine, as DeepDive currently
uses impala-shell, a command-line SQL client, which cannot be run
on OSX natively. To facilitate development, however, we will install
DeepDive and DeepDive apps in a folder that is shared between guest and
host operating system. That way we can still use an IDE for development. We
recommend using Intellij for DeepDive (scala), and pycharm for DeepDive apps (python).


## 1. Install the Impala VM

* Install [virtualbox](https://www.virtualbox.org/).

* Download [CDH VM](http://www.cloudera.com/content/cloudera/en/downloads/quickstart_vms/cdh-5-4-x.html)

* You will need to install p7zip to unpack the file, eg. using `brew install p7zip`.

* Unpack with `7z e ...`.

* Open VirtualBox, and select File > Import appliance.


## 2. Create a shared Folder

Create a folder on the Host computer (OSX) that will be shared, for example ~/dd.

Boot the guest operating system in VirtualBox.

In the VirtualBox menu, select Devices, then Shared Folders Settings, and add 

    Folder Path: /Users/../dd
    Folder Name: dd
    Read-only: No
    Auto-mount: Yes
    Make Permanent: Yes

Now, in your guest operating system, mount the shared folder

    sudo mount -t vboxsf -o uid=$UID,gid=$(id -g) dd ~/dd


## 3. Install DeepDive in VM

To install DeepDive, do the following:

    sudo yum -y update

    cd ~/dd
    git clone -b raphael-impala https://github.com/HazyResearch/deepdive.git

    cd deepdive
    make

Unfortunately, the guest OS is based on an older version of centos
that doesn't include the version of glibc that DeepDive's sampler uses.
To get around this, you will need to check out the source and recompile it.
For this you will need a newer version of gcc.

    mkdir ~/software
    cd ~/software 
    wget http://www.netgull.com/gcc/releases/gcc-5.1.0/gcc-5.1.0.tar.gz
    tar xvzf gcc-5.1.0.tar.gz
    cd gcc-5.1.0
    ./contrib/download_prerequisites
    ./configure --disable-multilib
    make
    sudo make install

The compilation takes a very long time. Go take a walk.

You can now check out the sampler and build it.

    cd ~/dd
    git clone https://github.com/hazyresearch/sampler.git
    make


## 4. Test DeepDive on Impala

    cd ~/dd
    git clone https://github.com/raphaelhoffmann/test-impala.git
    cd test-impala
    ./run.sh

<!--
cd /etc/yum.repos.d
sudo wget http://people.centos.org/tru/devtools-2/devtools-2.repo
sudo yum --enablerepo=testing-devtools-2-centos-6 install devtoolset-2-gcc devtoolset-2-gcc-c++
-->

## 5. Install IDE on your host operating system

If you're like me and don't like using a window manager through virtualbox,
you can install an IDE on your host operating system to edit the files
in the shared folder. You can also use iTerm on your host operating system
and SSH into the guest operating system to run commands. 

For SSH, follow these steps:

Shutdown the guest operating system

Change Settings, add a second Network Adapter of type "host-only".
Add a static IP (should be in different subnet from the other NAT adapter)

Restart the guest operating system

Now, from guest operating system, can SSH into host.
And, from host operating system, can SSH into guest.

For the latter, check your guest IP with ifconfig in your virtualbox VM.
It's likely device eth1.

Then can SSH in from host using
user: cloudera
password: cloudera
