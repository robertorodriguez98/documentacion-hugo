---
title: "Práctica: Servidor de correos"
date: 2023-02-07T01:57:14+01:00
draft: false
---

## Gestión de correo desde el servidor

### Tarea 1

{{< alert "circle-info" >}}
Documenta una prueba de funcionamiento, donde envíes desde tu servidor local al exterior. Muestra el log donde se vea el envío. Muestra el correo que has recibido. Muestra el registro SPF.
{{< /alert >}}

Creo las siguientes entradas en el DNS  de mi dominio:

![MX](https://i.imgur.com/izykpPC.png)

Donde hay un registro MX que apunta a mail.admichin.es, que a su vez es un registro A que apunta a la IP de mi servidor. Además, hay un registro SPF que apunta a la ip de la máquina.

Además, configuro resolución inversa en la configuración de la VPS:

![Reverse](https://i.imgur.com/Q8x5DC1.png)

Ahora, en la VPS instalo los siguientes paquetes:

```bash
apt update
apt install postfix bsd-mailx -y
```

Durante la configuración, selecciono **Internet Site** y **admichin.es**.

Envío un correo a mi cuenta personal:

```bash
mail robertorodriguezmarquez98@gmail.com
Subject: Prueba de funcionamiento
Hola buenos días
Cc:
```

Vemos el log de postfix:

```bash
tail /var/log/mail.log
```

![Log](https://i.imgur.com/lii209w.png)

Y el correo recibido:

![Correo](https://i.imgur.com/FWNO7aB.png)

### Tarea 2

{{< alert "circle-info" >}}
Documenta una prueba de funcionamiento, donde envíes un correo desde el exterior (gmail, hotmail,…) a tu servidor local. Muestra el log donde se vea el envío. Muestra cómo has leído el correo. Muestra el registro MX de tu dominio.
{{< /alert >}}

Ahora envío un correo desde mi cuenta personal a mi servidor:

![Correo](https://i.imgur.com/FWNO7aB.png)

Y compruebo que el correo ha llegado a mi servidor:

![Correo](https://i.imgur.com/JWSTUmj.png)
![Correo](https://i.imgur.com/D0xKQDi.png)

Y el log de postfix:

![Log](https://i.imgur.com/pAWEPrP.png)

## Uso de alias y redirecciones

### Tarea 3

{{< alert "circle-info" >}}
**Usos de alias y redirecciones**. Vamos a comprobar como los procesos del servidor pueden mandar correos para informar sobre su estado. Por ejemplo cada vez que se ejecuta una tarea cron podemos enviar un correo informando del resultado. Normalmente estos correos se mandan al usuario `root` del servidor, para ello:

```bash
 crontab -e
```

E indico donde se envía el correo:

```bash
MAILTO=root
```

Puedes poner alguna tarea en el cron para ver como se mandan correo.

Posteriormente usando alias y redirecciones podemos hacer llegar esos correos a nuestro correo personal.

Configura el cron para enviar correo al usuario root. Comprueba que están llegando esos correos al root. Crea un nuevo alias para que se manden a un usuario sin privilegios. Comprueban que llegan a ese usuario. Por último crea una redirección para enviar esos correo a tu correo personal (gmail,hotmail,…).
{{< /alert >}}

Voy a crear un script que muestre la fecha y el espacio en el disco. Lo guardo en `/root/script-espacio.sh`:

```bash
#!/bin/bash

echo "##################################"
echo "Fecha y hora: $(date)"
echo "##################################"
echo "Espacio en el disco:"
df -h
```

Ahora creamos la tarea de cron para que se ejecute cada 5 minutos:

```bash
crontab -e
```

Y añadimos las siguientes líneas:

```bash
MAILTO = root

*/5 * * * * /root/script-espacio.sh
```

Cuando pasan 5 minutos, recibimos el correo:

![Correo](https://i.imgur.com/XVtl86N.png)

Ahora voy a crear un alias para que se envíen los correos a un usuario sin privilegios (en este caso, calcetines), editando el fichero `/etc/aliases`:

```bash
root: calcetines
```

Y ejecuto el comando `newaliases` para que se actualicen los alias.

Ahora, cuando pasen 5 minutos, recibimos el correo en el usuario sin privilegios:

![Correo](https://i.imgur.com/aogu98o.png)

Ahora voy a crear una redirección para que se envíen los correos a mi correo personal, editando el fichero `/home/calcetines/.forward`:

```bash
robertorodriguezmarquez98@gmail.com
```

Y ahora, cuando pasen 5 minutos, recibimos el correo en mi correo personal:

![Correo](https://i.imgur.com/UdaG5gg.png)

## Para asegurar el envío

### Tarea 4

{{< alert "circle-info" >}}
Configura de manera adecuada DKIM es tu sistema de correos. Comprueba el registro DKIM en la página https://mxtoolbox.com/dkim.aspx. Configura postfix para que firme los correos que envía. Manda un correo y comprueba la verificación de las firmas en ellos.
{{< /alert >}}

Voy a instalar el paquete `opendkim`:

```bash
apt install opendkim opendkim-tools -y
```

En el fichero `/etc/opendkim.conf`, edito las siguientes líneas:

```bash
Domain                  admichin.es
Selector                default
KeyFile                 /etc/opendkim/keys/admichin.es/default.private
#Socket                 local:/run/opendkim/opendkim.sock
Socket                  local:/var/spool/postfix/opendkim/opendkim.sock
PidFile                 /run/opendkim/opendkim.pid
TrustAnchorFile         /usr/share/dns/root.key
```

Ahora añado el socket en el fichero `/etc/default/opendkim`.

Tras eso, en el fichero `/etc/postfix/main.cf`, añado las siguientes líneas:

```bash
milter_default_action = accept
milter_protocol = 6
smtpd_milters = local:opendkim/opendkim.sock
non_smtpd_milters = $smtpd_milters
```

Ahora genero los ficheros de claves:

```bash
mkdir /etc/opendkim/keys/admichin.es
cd /etc/opendkim/keys/admichin.es
opendkim-genkey -b 2048 -d admichin.es -D /etc/opendkim/keys/admichin.es -s default -v
```

Ahora, utilizando el contenido de `/etc/opendkim/keys/admichin.es/default.txt`, añado un registro TXT en el dominio `admichin.es`:

![Registro](https://i.imgur.com/rWLIaWm.png)

Reinicio los servicios:

```bash
systemctl restart opendkim postfix
```

Ahora, cuando envío un correo, se añade la firma DKIM:

![Correo](https://i.imgur.com/SjhdRvn.png)

Finalmente, compruebo la verificación de la firma en la página **mxtoolbox**:

![Verificación](https://i.imgur.com/leht5mi.png)

## Para luchar contra el spam

## Gestión de correos desde un cliente

### Tarea 8

{{< alert "circle-info" >}}
Configura el buzón de los usuarios de tipo Maildir. Envía un correo a tu usuario y comprueba que el correo se ha guardado en el buzón Maildir del usuario del sistema correspondiente. Recuerda que ese tipo de buzón no se puede leer con la utilidad mail.
{{< /alert >}}

Voy a cambiar el tipo de buzón de los usuarios, editando el fichero `/etc/postfix/main.cf`:

```bash
home_mailbox = Maildir/
```

Ahora instalamos el cliente mutt para poder leer los correos:

```bash
apt install mutt -y
systemctl restart postfix
```

Tengo que hacer la siguiente configuración en cada usuario:

```bash
nano ~/.muttrc
```

```bash
set mbox_type=Maildir
set mbox="~/Maildir"
set folder="~/Maildir"
set spoolfile="~/Maildir"
set record="+.Sent"
set postponed="+.Drafts"
set mask="!^\\.[^.]"
```

Podemos ver el contenido del directorio `Maildir`:

![Directorio](https://i.imgur.com/xJmTIsK.png)

Ahora, cuando envío un correo, se guarda en el buzón Maildir del usuario del sistema correspondiente, y lo podemos leer con mutt:

![Correo](https://i.imgur.com/mCg6V9I.png)

### Tarea 9

{{< alert "circle-info" >}}
Instala configura dovecot para ofrecer el protocolo IMAP. Configura dovecot de manera adecuada para ofrecer autentificación y cifrado.
{{< /alert >}}




## Comprobación final

