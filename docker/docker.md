# Introducción

## Imágenes y Contenedores

| Imágenes                                                                | Contenedores                                                   |
| ----------------------------------------------------------------------- | -------------------------------------------------------------- |
| Plantilla de solo lectura                                               | Instancia de una imagen                                        |
| Sistema de ficheros y parámetros listos para ejecutar                   | Es un directorio dentro del sistema, similar a los “jail root” |
| Basadas en sistemas operativos LinuxImágenes y ContenedoresContenedores | Pueden ser ejecutados, reiniciados, parados...                 |



## Gestión de Imágenes

* `docker images` permite ver las imágenes instaladas en el sistema.
* `docker rmi <nameImage>` para borrar una imagen del systema.

## Gestión de contenedores

* `Docker run`&#x20;
* `Docker ps` muestra solo contenedores en ejecución
  * `docker ps -a` mostraría todos los contenedores.
  * `docker ps -l` muestra el último contenedor que se ha ejecutado.
* `docker rm <containerID>` borra un contenedor.
* `docker inspect nameContainer` muestra la información del contenedor.

## Exposición de puertos

`docker run -p puertoMáquinaHOST:puertoAplicaciónDocker nameImages`

Por ejemplo:

* `docker run -p 81:5000 test`

`docker inspect nameContainer` buscar el apartado de `PortBindings` que mostrará la redirección hecha con la configuración de 81:5000.

```bash
"PortBindings": {
                "5000/tcp": [ #puerto del contenedor
                    {
                        "HostIp": "",
                        "HostPort": "81" #puerto de la máquina HOST
                    }
                ]
            },
```

## Crear redes

`docker network create -d bridge nameNewRed`, -d indica el driver utilizado para crear la nueva red.

Para conectar 2 contenedores que estén conectado en redes distintas, se utiliza: `docker network connect nameRed containerToConnect`

## Almacenamiento de Datos

* **comando**: `docker volume`
* Un volumen de datos se inicia cuando se inicia un nuevo container.
* Un volumen no se elimina cuando se elimina un container

**Ejemplo:**

* `docker run -d -v ~/dirToVolumeIn Host:/dirToSaveFromContainer -p host:container -ti nameContainer`

## Usar contenedor como almacenamiento

1. Crear un contenedor como contenedor de almacenamiento
   * `docker create -v /tmp --name datacontainer ubuntu`
2. Crear contenedor asociado al contenedor de almacenamiento
   * `docker run -t -i --volumes-from datacontainer ubuntu /bin/bash`
