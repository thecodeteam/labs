# Deploy VirtualBox with Docker Machine and Install REX-Ray

Simple QuickStart on using Docker Machine with VirtualBox and installing REX-Ray

Terminal Window 1:
```
$ vboxwebsrv -H 0.0.0.0 -v
```

Terminal Window 2:
```
$ docker-machine create -d virtualbox vbox01
$ docker-machine create -d virtualbox vbox02
$ docker-machine ssh vbox01 "curl -sSL https://dl.bintray.com/emccode/rexray/install | sh" && docker-machine ssh vbox02 "curl -sSL https://dl.bintray.com/emccode/rexray/install | sh"
$ docker-machine ssh vbox01 "sudo tee -a /etc/rexray/config.yml << EOF
libstorage:
  service: virtualbox
  integration:
    volume:
      operations:
        mount:
          preempt: true
virtualbox:
  endpoint: http://192.168.99.1:18083
  volumePath: /Users/<username>/VirtualBox/Volumes
  controllerName: SATA"
$ docker-machine ssh vbox02 "sudo tee -a /etc/rexray/config.yml << EOF
libstorage:
  service: virtualbox
  integration:
    volume:
      operations:
        mount:
          preempt: true
virtualbox:
  endpoint: http://192.168.99.1:18083
  volumePath: /Users/<username>/VirtualBox/Volumes
  controllerName: SATA"
$ docker-machine ssh vbox01 "sudo rexray start" && docker-machine ssh vbox02 "sudo rexray start"
```

Set your local Docker client to point to your host using the following commands
```
$ docker-machine env vbox01
$ eval $(docker-machine env vbox01)
```
