# Quick Commands

MDM1 -> `docker swarm init --advertise-addr 192.168.50.12`
MDM2 & TB are workers

```
docker volume create -d rexray --name mc_data --opt=size=5 && docker volume create -d rexray --name mc_mods --opt=size=5 && docker volume create -d rexray --name mc_config --opt=size=5 && docker volume create -d rexray --name mc_plugins --opt=size=5 && docker volume create -d rexray --name mc_home_minecraft --opt=size=5
```

```
docker service create --replicas 1 --name mc -p 25565:25565 -e EULA=TRUE --mount type=volume,target=/data,source=mc_data,volume-driver=rexray --mount type=volume,target=/mods,source=mc_mods,volume-driver=rexray --mount type=volume,target=/config,source=mc_config,volume-driver=rexray --mount type=volume,target=/plugins,source=mc_plugins,volume-driver=rexray --mount type=volume,target=/home/minecraft,source=mc_home_minecraft,volume-driver=rexray --constraint 'node.role == worker' itzg/minecraft-server
```