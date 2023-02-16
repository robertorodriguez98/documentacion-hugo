---
title: "Implantación de aplicaciones web PHP en docker"
date: 2023-02-06T13:41:58+01:00
draft: true
---

{{< alert "circle-info" >}}
Imaginemos que el equipo de desarrollo de nuestra empresa ha desarrollado una aplicación PHP que se llama BookMedik (https://github.com/evilnapsis/bookmedik).

Queremos crear una imagen Docker para implantar dicha aplicación.
{{< /alert >}}

## Tarea 1: Creación de una imagen docker con una aplicación web desde una imagen base

### Configuración de mariadb

```bash
docker run -d --name servidor_mysql \
                --network red_wp \
                -v mariadb:/var/lib/mysql \
                -e MYSQL_DATABASE=bd_bookmedik \
                -e MYSQL_USER=user_bookmedik \
                -e MYSQL_PASSWORD=asdasd \
                -e MYSQL_ROOT_PASSWORD=asdasd \
                mariadb
```

