# IMPLEMENTACIÓN DE UN CLUSTER KUBERNETES USANDO RANCHER PARA DESPLEGAR UNA APP.
<p align="center">
  <img width="auto" height="auto" src="https://programmerclick.com/images/570/861fdc8ff7fc326fd6def43388357672.png">
</p>

El propósito de esta guía es instalar Rancher como herramienta para desbloquear todo el potencial de Kubernetes, entendiendo Kubernetes (K8S) como un orquestador que permite manejar aplicaciones dentro de contenedores a través de muchos servidores los cuales pueden ser virtuales o físicos; creada por Google. Rancher nos facilita la supervisión del estado de todos nuestros clústeres de kubernetes en cualquier entorno y/ó distribución certificada de kubernetes como: **GKE, AKS, EKS, RKE, y K3S** a través de un panel de control, en otras palabras corre en cualquier cluster kubernetes y puede gestionar el cluster ya sea localmente ó externamente, es decir,  desplegar y gestionar kubernetes en otros entornos (Clústeres kubernetes alojados en los proveedores de nube). 

## **A. CREACIÓN DE MAQUINAS VIRTUALES CON VAGRANT**
Para lograr la implementación de Rancher resulta fundamental la existencia de dos VM, una para el servidor de Rancher y otra para los clústeres, en esta oportunidad utilizaremos Vagrant la cual es una herramienta para la creación de entornos virtualizados.

Primero definimos una ruta en la cual vamos a trabajar y creamos una carpeta con el nombre que deseen, en nuestro caso ‘rancher’:
```
mkdir rancher
cd rancher
```
Ahora vamos a inicializar vagrant por medio del siguiente código:
```
vagrant init
```
Se creará un archivo llamado ‘Vagrantfile’ el cual vamos a modificar con su editor de texto preferido, en nuestro caso VIM:
```
vim Vagrantfile
```
Dentro de nuestro editor de texto, presionamos la tecla **‘Esc’** y posteriormente **‘:%d’**, por último presionar **'ENTER'** para borrar todo:
<p align="center">
  <img width="auto" height="auto" src="https://user-images.githubusercontent.com/81927109/169581150-9d1cfaf9-c1d5-4a61-be7b-8935c2d847c0.png">
</p>

Una vez esté vacío nuestro Vagrantfile presionamos la tecla **‘Esc’** nuevamente y luego la tecla **‘I’** para insertar nuestros parámetros, con ayuda del mouse presionamos **'click derecho' + pegar** El Vagrantfile deberá contener lo siguiente:
```
Vagrant.configure("2") do |config|
config.vm.define :rancher do |rancher|
rancher.vm.box = "bento/ubuntu-20.04"
rancher.vm.network :private_network, ip: "192.168.100.2"
rancher.vm.hostname = "rancher"
end
config.vm.define :kubernetes do |kubernetes|
kubernetes.vm.box = "bento/ubuntu-20.04"
kubernetes.vm.network :private_network, ip: "192.168.100.3"
kubernetes.vm.hostname = "kubernetes"
end
end
```
En este apartado se establecen los parámetros de las máquinas virtuales como lo es el nombre, el sistema Operativo **(ubuntu 20.04)** y la dirección ip. Para guardar presionamos la tecla **‘Esc’** y luego **‘:qw’ (salir y guardar)** + **'ENTER'**

Ahora vamos a proceder a crear y correr las máquinas virtuales creadas con la siguiente instrucción:
```
vagrant up
```

## **B. INSTALACIÓN DE DOCKER**
Ingresamos a la máquina virtual **'rancher'**:
```
vagrant ssh rancher
```
Comprobamos si la máquina tiene Docker instalado y en caso de que sí, borrar esas dependencias:
```
sudo apt remove docker docker-engine docker.io containerd runc
```
Procedemos a Instalar Docker:
```
sudo apt install docker.io
```
Esperamos que termine y comprobamos la versión del Docker
```
docker --version
```

:+1: ¡Excelente! Ahora que tenemos Docker instalado vamos a iniciar y habilitar algunos servicios de Docker
````
sudo systemctl start docker
sudo systemctl enable docker
````
Ahora procedemos a verificar si el estado de docker se encuentra activo
````
sudo systemctl status docker
````

## **C. INSTALACIÓN DE RANCHER**
A partir de este comando se procese a la instalación de rancher dentro de la máquina virtual(Verificar la documentación: https://rancher.com/docs/rancher/v2.5/en/installation/other-installation-methods/single-node-docker/):

````
docker run -d --restart=unless-stopped \
  -p 80:80 -p 443:443 \
  --privileged \
  rancher/rancher:latest
````

En caso de no funcionar o haber conflictos por el privileged ejecutar el siguiente comando, y repetir el paso anterior:
````
sudo chmod 666 /var/run/docker.sock
````

###### **VISUALIZACIÓN DE LA IP**
Una vez descargado el rancher accedemos a la IP de la VM la podemos visualizar ejecutando el comando:
````
hostname -I
````
También podrías utilzar **'sudo su-'** para ingresar como **root** e **'ip s a'** para ver las ip:
````
sudo su -
ip s a
````
**copy + paste** IP en el navegador de preferencia, el navegador va a identificar que es un sitio no seguro de esta manera deben darle en habilitar o visualizar para poder continuar, así que no te preocupes :no_good_man:

<p align="center">
  <img width="auto" height="auto" src="https://blog.unelink.es/wp-content/uploads/2018/06/conexion-no-privada-chrome.jpg">
</p>

###### **IDENTIFICADOR DEL CONTAINER**
Despues de realizar este proceso se debe verificar el identificador para ello se toman los 4 primeros dígitos del container para asi obtener la contraseña del y poder acceder a docker.
````
docker ps
````

###### **OBTENCIÓN DE LA CONTRASEÑA**
Después de tener el identificador se colocan esos dígitos en el siguiente comando:
````
docker logs  84e5  2>&1 | grep "Bootstrap Password:"
````
:teacher: **Nota:**  Los dígitos en negrilla son parte del id de mi contenedor. Colocar los dígitos de su contenedor.

Se arroja una contraseña como la que se  ve a continuación la cual se debe colocar en el apartado de contraseña en rancher:
**Bootstrap Password: clrcxpqcm77………**

Una vez ingresado en el en la aplicación web de rancher la que nos va a permitir la administración de nuestros clusters tenemos la opción de importar un cluster ya existente. Utilizaremos esta opción una vez hayamos creado nuestro cluster de kubernetes usando minikube. Este punto nos permite linkear nuestro cluster de kubernetes con rancher para asi poder desplegar unas aplicaciones.

## **D. INSTALAR MINIKUBE**
En una VM distinta vamos a crear nuestro cluster de kubernetes con **minikube**
````
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
````
Ejecutamos la siguiente línea para poder tener disponible en consola el comando de minikube:
````
sudo cp minikube /usr/local/bin && rm minikube
````
Una vez instalado minkube debemos instalar kubectl para poder ejecutar los comandos en cluster de los kubernetes:
````
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
````
Hay que tener en cuenta que para poder iniciar el cluster de kubernetes debemos tener instalado docker. en caso no se así ejecutar el siguiente comando.  De lo contrario seguir con el siguiente paso:
````
sudo apt install docker.io
````
Para desplegar el cluster de kubernetes  para desarrollo local usando minikube debemos primero crear el grupo de docker  y agregar el usuario vagrant (Para este desarrollo se esta utilizando vagrant):

###### **CREAR EL GRUPO DE DOCKER**
````
 sudo groupadd docker
````
###### **AGREGAR EL USUARIO DE VAGRANT AL GRUPO DE DOCKER**
````
sudo usermod -aG docker $USER
````
###### **CERRAR LA VM Y VOLVER A INGRESAR**
````
$ exit
$ vagrant ssh servidorUbuntu
````
###### **VERIFICAR QUE MINIKUBE ESTÁ INSTALADO CORRECTAMENTE**
````
minikube version
````
###### **INICIAR CLUSTER**
````
minikube start
````
###### **VERIFICAR LA VERSIÓN DEL CLUSTER**
````
kubectl version
````
###### **VISUALIZAR LA VERSIÓN DEL CLUSTER**
````
kubectl version
````
###### **VISUALIZAR LA INFORMACIÓN DEL CLUSTER**
````
kubectl cluster-info
````
###### **VER LOS NODOS CREADOS EN EL CLUSTER**
````
kubectl get nodes
````
## D: RANCHER UI
Una vez creado el cluster con sus respectivos nodos debemos dirigirnos a la aplicación web de rancher y seleccionar la opción de  **IMPORT EXISTING CLUSTER.**

<p align="center">
  <img width="auto" height="auto" src="https://user-images.githubusercontent.com/81927109/169591827-b0a01355-103f-4475-823d-de450c58ac52.png">
</p>

Posteriormente se selecciona la opción de Generic, para poder importar cualquier cluster de kubernetes:

<p align="center">
  <img width="auto" height="auto" src="https://user-images.githubusercontent.com/81927109/169591981-83f4899d-f8d0-4fc0-8e68-71df8e11487c.png">
</p>

Al ingresar a este apartado se le debe dar un nombre al cluster  posteriormente se selecciona la opción de crear:

<p align="center">
  <img width="auto" height="auto" src="https://user-images.githubusercontent.com/81927109/169592097-fd8e44f2-c1d1-4c81-b517-d39380cff73c.png">
</p>

Una vez creado el cluster  en rancher, este nos provee unos comandos que debemos ejecutar en el cluster de kubernetes que ya teníamos creado anteriormente. Inicialmente se debe ejecutar la primer línea de comandos, en caso de que nos arroje un error debemos continuar con la segunda línea:

<p align="center">
  <img width="auto" height="auto" src="https://user-images.githubusercontent.com/81927109/169592330-f7c040ec-93be-4d32-857a-9a8fc382fe51.png">
</p>

Una vez ejecutado este comando rancher comienza a hacer la conexión con nuestro cluster. Dentro de la aplicación de rancher al lado izquierdo del cluster podemos ver su estado, tanto en proceso como activo, una vez finalizada la conexión se verá de la siguiente manera:

<p align="center">
  <img width="auto" height="auto" src="https://user-images.githubusercontent.com/81927109/169592482-0d570e42-0526-4646-a8d2-baf9f8852a53.png">
</p>

Una vez  conectado nuestro cluster podemos desplegar una cantidad de aplicaciones que nos provee rancher o importar una que tengamos a partir del archivo YAML:

<p align="center">
  <img width="auto" height="auto" src="https://user-images.githubusercontent.com/81927109/169592578-fdc50e39-e114-4d13-94f2-1da86fb9b9bf.png">
</p>

Podemos seleccionar la aplicación que deseemos del catálogo de rancher ó importarla a partir de la opcion que observamos en la esquina superior derecha.

<p align="center">
  ¡GRACIAS POR SEGUIR NUESTRA GUÍA!
  <img width="auto" height="auto" src="https://codster.io/wp-content/uploads/2021/09/servicios-cloud-computing.png">
</p>

