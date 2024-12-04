# CURSO DOCKER & KUBERNETES UTN

##### DICIEMBRE 2024

### ALUMNO: PABLO GALEANO

### DOCENTE : MARCOS TONINA

---

## Trabajo integrador de Kubernetes

Consigna:

Tomando como punto de partida lo entregado en el trabajo práctico número 1, entregar un archivo de texto (Word, MD o similar) donde se explique cómo armaría la infraestructura en Kubernetes, y por qué, para correr la aplicación. Indicar si haría algún cambio en su arquitectura o tecnologías para lograr un buen escalamiento.

Incluir, además, ejemplos de yaml donde se tenga en consideración el cómputo (pods/deployments/replicaset), persistencia (en caso de ser necesario) y red (servicios).

Al menos el YAML de cómputo y persistencia debe correr en Kubernetes para Docker Dektop o GKE (indicar en cuál).

---

ESTE DEPLOYMENT ESTÁ REALIZADO PARA KUBERNETES EN DOCKER DESKTOP

---

## Instrucciones de uso

- Descargar todo los archivos en una carpeta.

- Posicionarse en el path donde están los YAML.

- Abrir una línea de comandos y correr los siguiente comandos para descargar la imagenes y preparar el entorno dentro de Kubernetes para Docker Desktop.

- creamos las service de las redes:

```
kubectl apply -f servicenetwork.yaml
```

- creamos codificamos en base 64 la contraseña de la base de datos para el backend:

```
kubectl apply -f secrets.yaml
```

- creamos el Persistent Volumen Claim (PVC) para la base de datos, en el cual, creamos un volumen de 1GB para la base de datos, evitando que en algun caso, si el pod de mysql crashea, no se pierdan los datos y luego cuando levante el pod vuelva a conectarse al volumen.

```
kubectl apply -f pvc.yaml
```

- creamos los pods de backend, frontend y mysql.

```
kubectl apply -f deploymentback.yaml
kubectl apply -f deploymentfront.yaml
kubectl apply -f deploymentmysql.yaml
```

Para ingresar al front click
[Aquí](http://localhost:32249/)

---

### Explicaciones de tecnología usada y posibles mejoras.

#### Secrets

Si bien está codificado en base 64, lo óptimo sería encriptar con otra aplicación.

#### Deployments

El front se desarrolló en VITE (herramienta para react), el build se reliazó en node y luego corre en nginx.

El backend funciona en una imagen de node.

Para MySQL se utilizó una imagen de MySQL 8.0, pero para facilitar el deployment se creó una imagen con la DB incorporada y las tablas creadas.

#### Persistent Volumen Claim

En el caso de MySQL se utilizó un pvc para que en el caso de que se caiga el pod, no se pierdan los datos de la DB y cuando el pod vuelva a iniciar, se conecte con el volumen.

Aquí se podría utilizar una mejora apuntando, por ejemplo, a un storage en la nube o en un NFS en un server, para luego implementar políticas de backup.

#### Services de networks

Decidí separa los services entre volumenes, secrets y networks, por el lado de las redes de cada pod, opté para MySQL utilizar CLUSTERIP debido a que mas allá de que podría necesitar ingresar con MySQL workbench desde el host, se puede gestionar desde kubernetes, por lo tanto, no necesito que se exponga el puerto hacia fuera del cluster.

Por el lado del frontend decidí utilizar NODEPORT ya que es una solución óptima para un proyecto de este tipo mas orientado a una prueba o un producto no terminado.
Algo que se podría mejorar notablemente es utilizando un Ingress en el caso de utilizar GCP, en este caso lo utilizaría para producción donde tendría una IP pública y un puerto HTTP.
También opté por esta solución para no realizar instalaciones, en el caso de Ingress se debe instalar un controlador para Kubernetes.

También pordría implementar un Loadbalancer para distribuir las cargas. En este proyecto no lo consideré necesario, además de que no está en producción realmente.

Al utilizar NODEPORT opté por fijar los puertos del frontend y del backend.
Para el frontend, expongo el puerto 32249 y para el backend el puerto 30000.
de esta forma, evito posibles conflictos con otros puerto además de que NODEPORT asigna puertos desde el 30000 en adelante.

El backend lo debí exponer considerando que el front hace un POST con la url que se desea acortar, pero realmente el POST lo hace le navegador del host. Por lo tanto, se debe exponer el puerto del backend para recibir las peticiones.

---

### CONCLUSIONES FINALES

Decidí descartar el proyecto anterior que utilicé en Docker porque considero que era un proyecto sencillo en donde no utilizaría muchas de las herramientas de Kubernetes.

En el proyecto actual con la utilización de MySQL y la interconexión de servicios, pude realizar distintas pruebas con las configuraciones como es el caso del pvc y los servicios de networking, encontrandome dificultades con,por ejemplo, la interconexión entre el front,el back y el host del usuario final para realizar las peticiones.
