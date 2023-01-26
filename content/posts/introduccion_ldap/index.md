---
title: "Introducción a OpenLDAP"
date: 2023-01-25T13:47:32+01:00
draft: true
---

El protocolo **LDAP** es muy utilizado actualmente por empresa que apuestan por el software libre al utilizar distribuciones de Linux para ejercer las funciones propias de un **directorio activo** en el que se gestionarán las credenciales y permisos de los trabajadores y estaciones de trabajo en redes LAN corporativas en conexiones cliente/servidor.

{{< alert "circle-info" >}}
Realiza la instalación y configuración básica de OpenLDAP en alfa,utilizando como base el nombre DNS asignado. Deberás crear un usuario llamado prueba y configurar una máquina cliente basada en Debian y Rocky para que pueda validarse en servidor ldap configurado anteriormente con el usuario prueba.
{{< /alert >}}

## Instalación de OpenLDAP

Instalaremos OpenLDAP en el servidor alfa, para ello ejecutaremos los siguientes comandos:

```bash
apt update
apt install slapd
```

Durante la instalación nos pedirá que introduzcamos la contraseña de administrador del directorio:

![Instalación de OpenLDAP](https://i.imgur.com/NaNjKFA.png)

Una vez instalado con el comando `netstat -tulpn` comprobaremos que el servicio está escuchando en el puerto 389:

![Instalación de OpenLDAP](https://i.imgur.com/KGi2Vz6.png)

Ahora, utilizando el binario `ldapsearch` incluido en el paquete `ldap-utils` podemos buscar sobre el directorio:

```bash
ldapsearch -x -b "dc=roberto,dc=gonzalonazareno,dc=org"
```

![ldapsearch1](https://i.imgur.com/FgmXMPY.png)

## Configuración de OpenLDAP

![esquemaLDAP](https://i.imgur.com/qRgLvAz.png)

Para lograr una mejor estructura, la información suele organizarse en forma de **ramas** de las que cuelgan objetos similares (por ejemplo, una rama para usuarios y otra para grupos). Organizar de esta manera la estructura nos aporta también una mayor agilidad en las búsquedas, así como una gestión más eficiente sobre los permisos. Cada rama se denomina **organizational unit** (OU) y cada objeto que cuelga de ella se denomina **entry**.

ara definir dichos objetos, haremos uso de un fichero con extensión `.ldif`, en este caso he creado el fichero `base.ldif` con el siguiente contenido:

```ldif
dn: ou=Usuarios,dc=roberto,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Usuarios
```