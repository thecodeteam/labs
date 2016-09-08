# 3-Node ScaleIO Environment Using Vagrant

Simple QuickStart using ScaleIO with VirtualBox.

The [Vagrant ScaleIO](https://github.com/emccode/vagrant/tree/master/scaleio) deployment is simple because the installation of Docker and REX-Ray is done automatically and the IP addresses never change. 

```
$ git clone https://github.com/emccode/vagrant
$ cd vagrant/scaleio/
$ vagrant up --provider virtualbox
```

Open 3 terminal sessions to MDM1, MDM2, and TB. In this scenario, MDM1 is the API gateway and REX-Ray uses it to access the storage platform.

```
...window 1...
$ vagrant ssh mdm1
...window 2...
$ vagrant ssh mdm2
...window 3...
$ vagrant ssh tb
```
