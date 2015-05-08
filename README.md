# DeepDive QuickStart

The following guides will help you get DeepDive quickly up and running. Development configurations allow you to run DeepDive locally on your laptop, and are easiest to install. Deployment configurations allow you to scale to larger amounts of data, and work well on ec-2 and azure. In each case, you can choose between a setup on Impala and a setup on Greenplum.

## For Development

We assume that the development platform is OSX. Since some dependencies such as Impala require Linux, we run DeepDive inside a virtual machine.

* DeepDive with Impala  

```
curl    | bash
```

* DeepDive with Greenplum

```
curl | bash
```

Running above command will take you through the entire setup. If you would like to know how the virtual machines were created, see [here](VMs.md).

Note: on OSX, you can also run DeepDive without a virtual machine using a local postgres database.

## For Deployment

On ec-2 or azure, create an instance running Ubuntu 14.04 with at least 16G of memory and a 100G EBS volume.

* DeepDive with Impala

  [Installation guide](DEPLOY_IMPALA.md)

* DeepDive with Greenplum

  [Installation guide](DEPLOY_GREENPLUM.md)


[OTHER](OTHER.md)
