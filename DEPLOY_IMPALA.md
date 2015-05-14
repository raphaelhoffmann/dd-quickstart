# Deploying DeepDive with Impala

On your ec-2 or azure instance with Ubuntu 14.04, perform the following steps.
You may want to use a static (reserved) IP address for your instance since
Impala stores the IP internally and it won't start up when the IP changes after
reboot.

## 1. Launch CDH installer

SSH into your instance and run:

    wget http://archive.cloudera.com/cm5/installer/latest/cloudera-manager-installer.bin

    chmod u+x cloudera-manager-installer.bin

    sudo ./cloudera-manager-installer.bin

In order to access the console in step 2, I also had to open ports two ports: 7180 and 7182.

Ignoring security, I opened them in Ubuntu's firewall configuration as follows

    sudo ufw allow 7180
    sudo ufw allow 7182

I also had to open them in the security settings of the cloud provider. On ec-2, launch 
the AWS Management Console and edit the settings for the
security group you are using for your instance. To open these ports on azure, launch the
Azure Management Portal, select your virtual machine and add the ports as endpoints.

For better security, you may want to set up a VPN or use SSH tunnels.

## 2. CDH Setup

Access the CDH installer through your browser. Check your public IP/DNS on ec-2 in the
AWS Management Console.. On azure, you can go to http://MACHINENAME.cloudapp.net:7180. 

* User is admin, password is admin.

* Choose Cloudera Express.

* When it asks for hosts, just enter your MACHINENAME.

* Use default settings elsewhere.

* Install jdk.

* No single user mode

* Login as: ec2-user or azureuser

* Use password or keyfile (whatever you chose when you created the instance)

* Use defaults for remaining setup

* The installer may show you a warning that your system's `swappiness` setting is too high. In an SSH shell, run
```
    sudo sysctl vm.swappiness=10
    sudo sh -c 'echo "vm.swappiness=10" >> /etc/sysctl.conf'
```
* When installer prompts for Cluster Setup, choose "Core with Impala", and again use defaults elsewhere

## 3. Install other DeepDive Dependencies

Install a newer version of JDK

    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java8-installer

Install remaining packages for DeepDive repo

    sudo apt-get install git make gnuplot unzip

Install remaining packages for sampler repo

    sudo apt-get install g++ libnuma-dev libtclap-dev


## 4. Install DeepDive

In the user's home directory, run

    git clone https://github.com/HazyResearch/deepdive.git
    cd deepdive
    make
    cd ..
    git clone https://github.com/raphaelhoffmann/test-impala.git
    cd test-impala
    ./run.sh

This should take a few minutes, and then output a "summary report".

Congratulations, you have successfully installed DeepDive with Impala in a data center.


## 5. Install sampler (optional)

If you would like to make changes to the sampler, run

    git clone https://github.com/HazyResearch/sampler.git
    cd sampler
    make clean
    make
    
You can then copy the generated executable (dw) to deepdive's util dir and
rename to sampler-dw-linux.


