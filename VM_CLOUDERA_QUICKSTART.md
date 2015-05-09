# Create a DeepDive Appliance on the Impala Quickstart Appliance

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

## 6. Other pointers

* Set up host only network: 
http://askubuntu.com/questions/293816/in-virtualbox-how-do-i-set-up-host-only-virtual-machines-that-can-access-the-in
sudo ifconfig eth1 up

* Set up sudo with no password: 
http://askubuntu.com/questions/192050/how-to-run-sudo-command-with-no-password

```
sudo visudo -f /etc/sudoers.d/90-cloudimg-ubuntu
  and add line
  dd ALL=(ALL) NOPASSWD:ALL
```

* https://forums.virtualbox.org/viewtopic.php?f=35&t=50661
* http://blog.javabien.net/2014/03/03/setup-docker-on-osx-the-no-brainer-way/
* http://www.slideshare.net/cloudera/20140520-impala-meetuppythonudfpptx
* http://www.slideshare.net/powerhan96/networking-between-host-and-guest-v-ms-in-virtual-box
* VBoxManage commands

```
VBoxManage natnetwork add -t nat-int-network -n "192.168.15.0/24" -e -h on
VBoxManage natnetwork start -t nat-int-network
VBoxManage modifyvm "VM name" --nic<x> hostonly;
VBoxManage dhcpserver ...
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
```

* required for sampler
```
wget http://downloads.sourceforge.net/project/tclap/tclap-1.2.1.tar.gz
tar xvzf tclap-1.2.1.tar.gz
./configure
make
sudo make install

sudo yum install numactl-devel

sudo yum install glibc-devel.i686
```

