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

    mkdir ~/dd
    sudo mount -t vboxsf -o uid=$UID,gid=$(id -g) dd ~/dd


## 3. Install DeepDive in VM

To install DeepDive, do the following:

    sudo yum -y update
    sudo yum -y install gnuplot

    cd ~/dd
    git clone -b raphael-impala https://github.com/HazyResearch/deepdive.git

    cd deepdive
    make

    export LD_LIBRARY_PATH=/home/cloudera/dd/deepdive/lib/dw_linux/lib64

You must unset the `TERM` variable (by default it is set to xterm). The
reason is that DeepDive uses stdout to transfer data between processes. With
TERM set to xterm, a special control character is added to the beginning of
an output which causes a problem when reading.

    unset TERM


Unfortunately, the guest OS is based on an older version of centos
that doesn't include the version of glibc that DeepDive's sampler uses.
To get around this, you will need to check out the source and recompile it.
For this you will need a newer version of gcc.

    mkdir ~/software
    cd ~/software 
    wget http://www.netgull.com/gcc/releases/gcc-4.8.4/gcc-4.8.4.tar.gz
    tar xvzf gcc-4.8.4.tar.gz
    cd gcc-4.8.4
    ./contrib/download_prerequisites
    ./configure --disable-multilib
    make
    sudo make install

The compilation takes a very long time. Go take a walk.

    sudo yum install numactl-devel


    locate libstdc++..
    export LD_LIBRARY_PATH=...

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

* Shutdown the guest operating system

* Change Settings, add a second Network Adapter of type "host-only". Add a static IP (should be in different subnet from the other NAT adapter)

* Restart the guest operating system

Now, from guest operating system, can SSH into host.
And, from host operating system, can SSH into guest.

For the latter, check your guest IP with `ifconfig` in your virtualbox VM.
It's likely device eth1.

Then can SSH in from host using

    ssh cloudera@192.168.99.100

and password `cloudera`.


TODO:
http://askubuntu.com/questions/293816/in-virtualbox-how-do-i-set-up-host-only-virtual-machines-that-can-access-the-in
sudo ifconfig eth1 up

Ubuntu Minimal 14.04

http://askubuntu.com/questions/192050/how-to-run-sudo-command-with-no-password

sudo visudo -f /etc/sudoers.d/90-cloudimg-ubuntu
  and add line
  dd ALL=(ALL) NOPASSWD:ALL

https://forums.virtualbox.org/viewtopic.php?f=35&t=50661

http://blog.javabien.net/2014/03/03/setup-docker-on-osx-the-no-brainer-way/
http://www.slideshare.net/cloudera/20140520-impala-meetuppythonudfpptx

install cloudera manager
http://www.slideshare.net/powerhan96/networking-between-host-and-guest-v-ms-in-virtual-box


VBoxManage natnetwork add -t nat-int-network -n "192.168.15.0/24" -e -h on
VBoxManage natnetwork start -t nat-int-network



VBoxManage modifyvm "VM name" --nic<x> hostonly;
VBoxManage dhcpserver ...



wget http://downloads.sourceforge.net/project/tclap/tclap-1.2.1.tar.gz
tar xvzf tclap-1.2.1.tar.gz
./configure
make
sudo make install

sudo yum install numactl-devel

sudo yum install glibc-devel.i686





VBoxManage createvm --name u2 --register

wget https://s3.amazonaws.com/clearcutanalytics/dd.ova
VBoxManage import .....ova
VBoxManage startvm ubuntu_1 --type headless

VBoxManage hostonlyif create

0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%
Interface 'vboxnet4' was successfully created

(must read name vboxnet4 from output)

VBoxManage hostonlyif ipconfig vboxnet4 --ip 192.168.60.1

VBoxManage modifyvm ubuntu_1 --natnet1 default

VBoxManage modifyvm ubuntu_1 --nic2 hostonly

VBoxManage modifyvm ubuntu_1 --natnet1 "192.168/16"
VBoxManage modifyvm ubuntu_1 --nic1 nat











# Creating a New DeepDive Appliance

To simplify setting up DeepDive on a local machine for development, we use virtual machines.
Users can download a pre-packaged virtual machine from our website. DeepDive virtual machines
are based on Ubuntu, and there exists one virtual machine for every supported database backend.
This guide shows how we create new virtual machines.

## Creating an Ubuntu virtual machine

* Download Ubuntu 14.04 Minimal 

```
    wget http://archive.ubuntu.com/ubuntu/dists/trusty/main/installer-amd64/current/images/netboot/mini.iso
```

* In VirtualBox, create a new virtual machine with

```
    Name:    dd
    Type:    Linux
    Version: Ubuntu (64 bit)
    Memory:  4096MB or more
    Create a virtual hard drive now
    VMDK
    Dynamically allocated
    20GB or more
```

* Select new VM, Settings, Storage, and add CD/DVD drive to IDE controller. Choose Disk and select mini.iso from above.
  Check `Live CD/DVD` for the new drive, and make sure it is the Primary IDE drive.

* Now start the VM, and install Ubuntu Minimal. Choose username dd and password dd.
  Select OpenSSH server as software to install.

* Must shutdown VM, and remove CD/DVD drive in Settings.

## Setting up Networking and a Shared Volume

* In menu, select VirtualBox, Preferences, Network, Host-only Networks, and add a new host-only network.
  Change its settings to the following:

```
  IPv4 Address:      192.168.60.1
  IPv4 Network Mask: 255.255.255.0
  IPv6 Address:      (empty)
  IPv6 Network Mask Length: 0
```

  Do not enable DHCP.

* While VM is shutdown, open Settings and add two network adapters. The first one is attached to NAT, the second one to Host-only Adapter. Here, you select the newly created host-only network.
  
* Start the VM and login. If you run `ifconfig`, the host-only adapter does not show up yet. Do the following.

  Run `ls /sys/class/net` to make sure the new adapter is there.

  Then run `sudoedit /etc/network/interfaces` and add the following lines at the bottom:

```
    # Host-only interface
    auto eth1
    iface eth1 inet static
        address         192.168.60.100
        netmask         255.255.255.0
        network         192.168.60.0
        broadcast       192.168.60.255
```

  Then run `sudo ifconfig eth1 up`. The new adapter shows up, but without IP. You need to restart your virtual machine for it to work.

  You can now login from your host OS using `ssh dd@192.168.60.100`.

* Next, we would like to enable password-less login using a key. On your host OS, run

```
    ssh-keygen -t rsa -b 2048
```

  Store the keys in some folder (these will be shared). Copy the public key to the VM.

```
    scp id_rsa.pub dd@192.168.60.100:/tmp/id_rsa.pub
    ssh dd@192.168.60.100
    mkdir ~/.ssh
    cat /tmp/id_rsa.pub >> ~/.ssh/authorized_keys
```

  From your host, you should now be able to login using

```
    ssh -i id_rsa dd@192.168.60.100
```

* Finally, we would like to share a directory between VM and host OS.
  In the VirtualBox menu, select Devices, then Shared Folders Settings, and add

```
    Folder Path: /Users/USERNAME/dd
    Folder Name: dd
    Read-only: No
    Auto-mount: Yes
    Make Permanent: Yes
```

  We still need to mount this folder into your VM. In a VM shell, type

```
    sudo apt-get -y update
    sudo apt-get -y install virtualbox-guest-utils
```

  You need to restart your VM now. In a new VM shell, type

```
    mkdir ~/dd
    sudo mount -t vboxsf -o uid=$UID,gid=$(id -g) dd ~/dd
```


## Setting up Cloudera Impala

* Login to the virtual machine.

```
    wget http://archive.cloudera.com/cm5/installer/latest/cloudera-manager-installer.bin

    chmod u+x cloudera-manager-installer.bin

    sudo ./cloudera-manager-installer.bin
```


