---
title: "Cluster_drdb"
date: 2023-02-05T18:42:44+01:00
draft: true
---

## Clúster DRBD single primary

Datos maquinas

Maquina1
* IP 192.168.121.79
Maquina2
* IP 192.168.121.211

En ambas maquinas instalamos DRBD

```bash
apt-get install drbd-utils
```

Creamos el fichero `/etc/drbd.d/wwwdata.res`

```bash
resource wwwdata {
 protocol C;
 meta-disk internal;
 device /dev/drbd1;
 syncer {
  verify-alg sha1;
 }
 net {
  allow-two-primaries;
 }
 on maquina1 {
    disk /dev/vdb;
    address 192.168.121.79:7789;
 }
 on maquina2 {
    disk /dev/vdb;
    address 192.168.121.211:7789;
 }
}
```

Creamos el volumen

```bash
drbdadm create-md wwwdata
drbdadm up wwwdata
```

y en el nodo1 (maquina1) ejecutamos el siguiente comando:

```bash
drbdadm primary --force wwwdata
```

Ahora podemos ver el resultado del comando con `drbdadm status wwwdata`

[FOTO]

En el nodo1 instalamos el sistema de ficheros:

```bash
apt install xfsprogs
mkfs.xfs /dev/drbd1
```

## Clúster DRBD con dual primary

creamos el fichero `/etc/drbd.d/dbdata.res`

```bash
resource dbdata {
 protocol C;
 meta-disk internal;
 device /dev/drbd2;
 syncer {
  verify-alg sha1;
 }
 net {
  allow-two-primaries;
 }
 on maquina1 {
    disk /dev/vdc;
    address 192.168.121.79:7790;
 }
 on maquina2 {
    disk /dev/vdc;
    address 192.168.121.211:7790;
 }
}
```

Creamos el volumen

```bash
drbdadm create-md dbdata
drbdadm up dbdata
```

y en ambos nodos ejecutamos el siguiente comando:

```bash
drbdadm primary --force dbdata
```


## Instalación de OCFS2

En ambos nodos instalamos el sistema de ficheros:

```bash
apt install ocfs2-tools
```

En uno de los nodos ejecutamos los siguientes comandos:

```bash
o2cb add-cluster micluster
```

Ahora añadimos los nodos al cluster:

```bash
o2cb add-node micluster maquina1 --ip 192.168.121.79
o2cb add-node micluster maquina2 --ip 192.168.121.211
```


mkfs.ocfs2 --cluster-stack=o2cb --cluster-name=micluster /dev/drbd2

Para poder volver a usarlo tras el reinicio hay que usar el siguiente comando:

```bash
o2cb register-cluster micluster
```
