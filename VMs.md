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

* In your VM Settings, make sure your VM has a NAT network adapter. Activate Port Forwarding from host 3022 to guest 22.

<!--

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
-->

* Next, we would like to enable password-less login using a key. On your host OS, run

```
    ssh-keygen -t rsa -b 2048
```

  Store the keys in some folder (these will be shared). Copy the public key to the VM.

```
    scp id_rsa.pub -p 3022 dd@127.0.0.1:/tmp/id_rsa.pub
    ssh -p 3022 dd@127.0.0.1
    mkdir ~/.ssh
    cat /tmp/id_rsa.pub >> ~/.ssh/authorized_keys
```

  From your host, you should now be able to login using

```
    ssh -i id_rsa -p 3022 dd@127.0.0.1
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

  Your shared folder should now be accessible on both host and guest.
  To make the system mount the device on every restart, do
```
    sudoedit /etc/rc.local
```
  Remove the hash in the first line (before !), and insert the following line before exit:
```
    sudo mount -t vboxsf -o uid=$UID,gid=$(id -g) dd ~/dd
```


## Uploading the Appliance to S3

Subdirectory `mac-impala-dive` contains the install scripts for the appliance. To host the new appliance, do:

1. Package a few of the files

```
tar cvzf dive.tgz keys dive
```

2. Upload the following to S3:
```
dive.tgz
install
dd.ova
```
dd.ova is the appliance you created. Make sure all file permissions are set to public.

3. Test

```
curl s3://...../mac-impala/install | bash
```


## Setting up Cloudera Impala

* Add a Port Forwarding rule for guest port 7180 to host port 7180. Use the virtualbox GUI for that.
<!--
```
VBoxManage modifyvm dd --natpf1 "cloudera,tcp,,7180,,7180"
```
-->
* Login to the virtual machine.

* Run
```
sudo ufw allow 7180
```

* Set up password-less sudo
```
sudo visudo -f /etc/sudoers.d/90-cloudimg-ubuntu
```
and add line
```
dd ALL=(ALL) NOPASSWD:ALL
```

* Change IP for hostname ubuntu
```
sudoedit /etc/hosts
```
Change to
```
127.0.0.1       ubuntu localhost
#127.0.1.1      ubuntu
```

* Run installer

```
    wget http://archive.cloudera.com/cm5/installer/latest/cloudera-manager-installer.bin

    chmod u+x cloudera-manager-installer.bin

    sudo ./cloudera-manager-installer.bin
```

* In the cloudera installer Search for hostname `ubuntu`

* If the installation requires password-less sudo, then SSH into the guest OS, and

```
sudo visudo -f /etc/sudoers.d/90-cloudimg-ubuntu
```
and add line
```
dd ALL=(ALL) NOPASSWD:ALL
```

* Java (for DeepDive)
```
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

* Other required packages
```
sudo apt-get install git make gnuplot unzip
```

* Impyla
```
sudo apt-get install python-pip
sudo apt-get install python-dev
sudo easy_install -U setuptools
pip install impyla

```


## Setting up Greenplum


## TODO:

Answer of http://serverfault.com/questions/243343/headless-ubuntu-server-machine-sometimes-stuck-at-grub-menu

