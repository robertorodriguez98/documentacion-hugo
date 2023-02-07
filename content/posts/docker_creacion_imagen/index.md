---
title: "Docker_creacion_imagen"
date: 2023-02-02T12:21:24+01:00
draft: true
---

1. Crea una página web estática (por ejemplo busca una plantilla HTML5). O simplemente crea un index.html.
2. Crea un fichero Dockerfile que permita crear una imagen con un servidor web sirviendo la página. Puedes usar una imagen base debian o ubuntu, o una imagen que tenga ya un servicio web, como hemos visto en el apartado **Ejemplo 1: Construcción de imágenes con una página estática.**
3. Ejecuta el comando docker que crea la nueva imagen. La imagen se debe llamar /mi_servidor_web:v1.
4. Conéctate a Docker Hub y sube la imagen que acabas de crear.
5. Descarga la imagen en otro ordenador donde tengas docker instalado, y crea un contenedor a partir de ella. (Si no tienes otro ordenador con docker instalado, borra la imagen en tu ordenador y bájala de Docker Hub).
6. Vamos a hacer una modificación de la página web: haz una modificación al fichero index.html.
7. Vuelve a crear una nueva imagen, en esta caso pon ta eiqueta v2. Súbela a Docker Hub.
8. Por último, baja la nueva imagen en el ordenador donde está corriendo el contenedor. Para hacer la implantación de la nueva versión debes borrar el contenedor y crear uno nuevo desde la nueva versión de la imagen.

## Ejemplo 2: Construcción de imágenes con una página estática.

En este ejemplo vamos a crear una imagen con un servidor web que sirva una página estática. Para ello vamos a usar una imagen base de Debian, y vamos a instalar en ella un servidor web.