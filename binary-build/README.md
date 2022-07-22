## Generar una imagen dentro de Openshift desde un archivo local

Estructura del proyecto ejemplo:
```
project
│   README.md
│   Dockerfile    
│
└───files
        index.html
```

Pasos:
1-Nos logueamos al cluster

```sh
> oc login ...
```

2-Creamos el proyecto
```sh
> oc new-project pruebas
```

3-Creamos un nuevo BuildConfig con la estrategia docker
```sh
> oc new-build --name test-binary-build --binary --strategy docker
```

4-Nos ubicamos en el directorio de nuestro proyecto donde se encuentra el Dockerfile
```sh
> cd /path/to/project
```

5-Iniciamos el build
```sh
> oc start-build test-binary-build --from-dir .
```

Con estos pasos, ya vamos a tener una imagen personalizada lista para utilizar dentro de Openshift.

OPCIONAL:

6-Creamos nuestra aplicación en base a la imagen pusheada 
```sh
> oc new-app test-binary-build 
```

7-Exponemos el servicio mediante la creación de una ruta
```sh
> oc expose svc/test-binary-build
```

NOTA: Si no contamos con la imagen base openjdk18-openshift, debemos ejecutar el comando: 
```sh
> oc import-image redhat-openjdk-18/openjdk18-openshift --from=registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift --confirm
```
---------------
## Advanced Builds

1. Compilamos el codigo en nuestro local
```sh
> mvn clean compile package -DskipTests
```
* Opcional:
```sh
> ls -l ./target

➜  hello-world-java git:(master) ✗ ls -l ./target
total 32232
drwxr-xr-x  4 jbondaru  staff       128 Jul 22 10:37 classes
drwxr-xr-x  3 jbondaru  staff        96 Jul 22 10:37 generated-sources
drwxr-xr-x  3 jbondaru  staff        96 Jul 22 10:37 generated-test-sources
-rw-r--r--  1 jbondaru  staff  16495457 Jul 22 10:37 hello-world-0.0.1-SNAPSHOT.jar
-rw-r--r--  1 jbondaru  staff      3706 Jul 22 10:37 hello-world-0.0.1-SNAPSHOT.jar.original
drwxr-xr-x  3 jbondaru  staff        96 Jul 22 10:37 maven-archiver
drwxr-xr-x  3 jbondaru  staff        96 Jul 22 10:37 maven-status
drwxr-xr-x  3 jbondaru  staff        96 Jul 22 10:37 test-classes
➜  hello-world-java git:(master) ✗ 
```

2. Creamos el BuildConfig (*)
```sh
> oc new-build \
   --name hello-world-java \
   --binary \
   --image-stream openjdk18-openshift:latest
```
> (*) Si no hizo el ejemplo anterior, deberá crear en namespace
```sh
> oc new-project pruebas
```
3. Construimos la imagen
```sh
> oc start-build hello-world-java \
   --from-file=./target/hello-world-0.0.1-SNAPSHOT.jar \
   --follow
```
4. Creamos nuestra aplicación en base a la imagen construida
```sh
> oc new-app hello-world-java:latest
```
5. Exponemos el servicio mediante la creación de una ruta
```sh
> oc expose svc/hello-world-java
```
-----------------------

## Anexo docker local

* Para construir una imagen local:  
```sh
> docker build -t test-binary-build:v1 .
```
* Para listar las imagenes disponibles:
```sh
> docker images
```
* Para ejecutar la imagen:
```sh
> docker run -it -d -p 4000:8080 test-binary-build:v1
```
* Para listar las imagenes que estan en ejecucion
```sh
> docker ps
```
* Para matar la imagen en ejecucion
```sh
> docker kill <CONTAINER ID>
```
* Para ingresar dentro del contenedor
```sh
> docker exec -it <CONTAINER ID> bash
```