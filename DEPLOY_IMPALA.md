# Deploying DeepDive with Impala

On your ec-2 or azure instance with Ubuntu 14.04, perform the following steps.

## Open ports 22, 7180, 7182

To open these ports on ec-2, launch the AWS Management Console and edit the settings for the
security group you are using for your instance. To open these ports on azure, launch the
Azure Management Portal, select your virtual machine and add the ports as endpoints.

## 1. Launch CDH installer

SSH into your instance and run:

    wget http://archive.cloudera.com/cm5/installer/latest/cloudera-manager-installer.bin

    chmod u+x cloudera-manager-installer.bin

    sudo ./cloudera-manager-installer.bin

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

    sudo sysctl vm.swappiness=10
    sudo sh -c 'echo "vm.swappiness=10" >> /etc/sysctl.conf'

* When installer prompts for Cluster Setup, choose "Core with Impala", and again use defaults elsewhere

## 3. Install other DeepDive Dependencies

Install a newer version of JDK

    sudo add-apt-repository ppa:webupd8team/java
    sudo apt-get update
    sudo apt-get install oracle-java8-installer

Install remaining packages

    sudo apt-get install git make gnuplot unzip

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


