# 3-Node ScaleIO Environment Using Vagrant

The Vagrant file provided will use VirtualBox to create three (3) hosts. Each
host will have ScaleIO (a software that turns DAS storage into shared and
scale-out block storage) installed and configured. The hosts will also
automatically install and configure Docker Engine and REX-Ray. This gives a
fully configured environment ready to test stateful applications and clustering
functionality using the ScaleIO driver for REX-Ray.

---

The [Vagrant ScaleIO](https://github.com/codedellemc/vagrant/tree/master/scaleio) deployment is simple because the installation of Docker and REX-Ray is done automatically and the IP addresses never change.

```
$ git clone https://github.com/codedellemc/vagrant
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
