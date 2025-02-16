## Deploy-Kubernetes-using-Con-gMaps-and-Helm
#Creating well-structured, declarative, and reusable deployments within Kubernetes is an essential skill that increases productivity, using a manifest file, which is used to declare cluster resources, along with Helm (v3), which can be used to template the manifests.

First of all, I thank Cloud Academy for teaching these topics: https://www.qa.com/

## Qué Vamos a implementar:
Una aplicación en un clúster de Kubernetes utilizando:
a.	ConfigMaps : Para gestionar los ajustes de la aplicación (por ejemplo, variables de entorno, configuraciones).
b.	Helm : Como administrador de paquetes para definir, instalar y gestionar la aplicación de manera declarativa.
Instalar el Chart en Kubernetes :
•	Utilice Helm para instalar la aplicación en el clúster.
Verificar y Gestionar :
•	Comprobar el estado de la aplicación y su configuración.
•	Actualizar o eliminar la implementación según sea necesario.
Beneficios
•	Gestión de configuración : ConfigMaps permite separar la configuración del código.
•	Instalación y actualización simplificadas : Helm facilita la gestión de aplicaciones complejas en Kubernetes.
•	Declarativa : Se define el estado deseado, y Kubernetes y Helm se encargan de alcanzarlo.
Para grabar
•	ConfigMaps : Para configuración.
•	Helm : Para definir e instalar aplicaciones en Kubernetes de manera eficiente.

### Paso 1 
Se conﬁgurará un contenedor de servidor web NGINX para redirigir las solicitudes HTTP entrantes en sentido descendente a otro contenedor que ejecute una aplicación web personalizada basada en FLASK. El contenedor del servidor web NGINX utilizará la imagen nginx:1.13.7 disponible públicamente. El contenedor de la aplicación web basado en FLASK se basará en una imagen de Docker personalizada que primero deberá crear.

a-Conexión al puerto IDE 8080 de contenedores basados en web 

b-Realizar la compilación de imágenes y crear espacio de nombres

### Paso 2
##En este paso creará una imagen de Docker personalizada que contiene un archivo MATRAZ
Aplicación web basada. La imagen de Docker personalizada se implementará más adelante en el clúster de Kubernetes. Antes de realizar la implementación dentro del clúster, se le
mostrará cómo crear un nuevo recurso de espacio de nombres y cómo establecerlo como el espacio de nombres predeterminado en el que se lleva a cabo la implementación restante.
Enumera el contenido del directorio ﬂaskapp. En el terminal ejecute el siguiente comando:
ls -la

### Paso 3
##Realice una compilación de Docker para crear una imagen de Docker personalizada. En el terminal ejecute el siguiente comando:
docker build -t cloudacademydevops/flaskapp .


comprobar ebe la presencia de la imagen de Docker recién creada. En el terminal ejecute el siguiente comando:

docker images


Cree un espacio de nombres personalizado de cloudacademy dentro del clúster de Kubernetes. En el terminal ejecute el siguiente comando:

kubectl create ns cloudacademy

Cambie al nuevo espacio de nombres cloudacademy. En el terminal ejecute el siguiente comando:

kubectl config set-context --current --namespace=cloudacademy

paso 4 Aplicar Nginx ConﬁgMap

Los ConﬁgMaps permiten desacoplar los artefactos de
conﬁguración del contenido de la imagen para mantener la portabilidad de las aplicaciones en contenedores.

Cambie al directorio k8s y enumere su contenido. En el terminal ejecute el siguiente comando:

cd ../k8s && ls -la

Ya en este repo se encuentra el archivo actualizado de el archivo de maniﬁesto nginx.conﬁgmap.yaml 
Aplique el archivo nginx.conﬁgmap.yaml actualizado en el clúster K8s: esto creará un nuevo recurso ConﬁgMap. En el terminal ejecute el siguiente comando:

kubectl apply -f nginx.configmap.yaml

Enumere todos los recursos de ConﬁgMap en el clúster K8s. En el terminal ejecute el siguiente comando:

kubectl get configmaps

A este paso ya actualizó, guardó y aplicó el archivo de maniﬁesto
nginx.conﬁgmap.yaml en el clúster K8s. Esto creó un nuevo ConﬁgMap que contenía la conﬁguración de NGINX que se montará en el contenedor NGINX 

Paso 5 Aplicar implementación

Las implementaciones representan un conjunto de varios pods idénticos y tienen la capacidad de reemplazar automáticamente las instancias que fallan o que no responden.

En este repo ya se encuentra actualizado el archivo de maniﬁesto deployment.yaml

Vamos a aplicar el archivo deployment.yaml actualizado en el clúster K8s, esto creará un nuevo archivo Despliegue.
En el terminal ejecute el siguiente comando:

kubectl apply -f deployment.yaml

Vamos a numerar todas las implementaciones en el clúster K8s. En el terminal ejecute el siguiente comando:

kubectl get deploy

Enumere todos los pods del grupo K8s. En el terminal ejecute el siguiente comando:

kubectl get pods

Extraiga el nombre del pod de frontend y guárdelo en una variable llamada POD_NAME. Hazlo eco de vuelta a la terminal. En el terminal ejecute los siguientes comandos:

POD_NAME=`kubectl get pods -o jsonpath='{.items[0].metadata.name}'`echo$POD_NAME

Ahora use el comando kubectl describe para obtener un informe detallado sobre el estado actual del pod de frontend. En el terminal ejecute el siguiente comando:

kubectl describe pod $POD_NAME

Utilice el comando docker images para conﬁrmar la etiqueta de imagen de Docker correcta para la aplicación web basada en FLASK que creó en el paso 1 del laboratorio. En el terminal ejecute el siguiente comando:

docker images | grep cloudacademydevops

Enumere todos los recursos de implementación en el clúster K8s. En el terminal ejecute el siguiente comando:

kubectl get deploy

Exponga la implementación de frontend. Esto creará un nuevo Servicio que permitirá llamar a los pods de frontend a través de una dirección VIP de red de clúster estable. En el terminal ejecute el siguiente comando:

kubectl expose deployment frontend --port=80 --target-port=80

Enumere el nuevo recurso de servicio frontend creado anteriormente. En el terminal ejecute el siguiente comando:

kubectl get svc frontend

Extraiga la dirección IP del clúster de servicios frontend y almacénela en una variable llamada FRONTEND_SERVICE_IP. Hazlo eco de vuelta a la terminal. En el terminal ejecute los siguientes comandos:

FRONTEND_SERVICE_IP=`kubectl get service/frontend -o jsonpath='{.spec.clusterIP}'` echo$FRONTEND_SERVICE_IP

Pruebe el servicio de frontend enviándole una solicitud de curl. En el terminal ejecute el siguiente comando:

curl -i http://$FRONTEND_SERVICE_IP

Vuelva a probar el servicio de frontend enviándole una nueva solicitud curl. En el terminal ejecute el siguiente comando:

curl -i http://$FRONTEND_SERVICE_IP

Examinemos ahora los registros asociados con el envío de tráﬁco a través del servidor web NGINX. Para empezar, enumere los pods actuales dentro del clúster. En el terminal ejecute el siguiente comando:

kubectl get pods

Extraiga el nombre del pod de frontend y guárdelo en una variable llamada FRONTEND_POD_NAME. Hazlo eco de vuelta a la terminal. En el terminal ejecute los siguientes comandos:

FRONTEND_POD_NAME=`kubectl get pods --no-headers -o custom-columns=":metadata.name"` echo$FRONTEND_POD_NAME

Realice una lista de directorios directamente dentro del contenedor NGINX enumerando el contenido del directorio /var/log/nginx. En el terminal ejecute el siguiente comando:

kubectl exec -it $FRONTEND_POD_NAME -c nginx -- ls -la /var/log/nginx/

Utilice el comando kubectl logs para examinar el registro NGINX generado por los comandos curl ejecutados anteriormente. En el terminal ejecute el siguiente comando:

kubectl logs $FRONTEND_POD_NAME nginx

Utilice el comando kubectl logs para examinar el registro FLASK generado por los comandos curl ejecutados anteriormente. En el terminal ejecute el siguiente comando:

kubectl logs $FRONTEND_POD_NAME flask

Paso 6 Crear proyecto de Helm

Para empezar, navegue hacia arriba en un directorio hasta el directorio:

cd ..

Utilice el comando helm create para generar un nuevo proyecto de gráﬁco de Helm denominado test-app. En el terminal ejecute el siguiente comando:

helm create test-app

Utilice el comando de árbol para renderizar la estructura de directorios en la pantalla. En el terminal ejecute el siguiente comando:

tree test-app/

Utilice el comando helm template para convertir las plantillas de Helm en un único archivo de maniﬁesto de Kubernetes implementable. En el terminal ejecute el siguiente comando:

helm template test-app/

En este paso, aprendió a usar el método Helm

Paso 7 Actualización de la implementación para usar plantillas de Helm

En este escenario, el gráﬁco de Helm de la aplicación proporcionado se ha conﬁgurado para crear una implementación que inicia un solo pod que contiene dos contenedores. El primer contenedor es un contenedor de servidor web NGINX que envía las solicitudes HTTP entrantes a un segundo contenedor que ejecuta una aplicación web basada en FLASK (basada en la imagen de Docker que creó anteriormente). 

Comience por eliminar todos los recursos anteriores iniciados dentro del clúster. Utilice el comando kubectl delete para eliminar los recursos de implementación, pod y servicio creados anteriormente. En el terminal ejecute el siguiente comando:

kubectl delete deploy,pods,svc --all

En este repo ya estan modificados el archivo values.yaml de Helm y el archivo de deployment.yaml 

Utilice el comando helm template para generar un archivo de maniﬁesto desplegable. En el terminal ejecute el siguiente comando:

helm template ./app

Con este comando SOLO generó el maniﬁesto implementable, aún no se ha implementado en el clúster, esto es útil para validar el maniﬁesto resultante ANTES de implementarlo realmente.

Esta vez realizará la implementación real en el clúster. Para ello, volverá a utilizar el comando helm template para generar un archivo de maniﬁesto implementable, canalizando la salida (maniﬁesto) directamente al comando kubectl apply. En el terminal ejecute el siguiente comando:

helm template cloudacademy ./app | kubectl apply -f -

Examine los servicios actuales disponibles dentro del clúster. En el terminal ejecute el siguiente comando:

kubectl get svc

Extraiga la dirección IP del clúster de cloudacademy-app service y almacénela en una variable denominada CLOUDACADEMY_APP_IP. Hazlo eco de vuelta a la terminal. En el terminal ejecute los siguientes comandos:

CLOUDACADEMY_APP_IP=`kubectl get service/cloudacademy-app -o jsonpath='{.spec.clusterIP}'`
echo $CLOUDACADEMY_APP_IP

Pruebe el servicio cloudacademy-app enviándole una solicitud curl. En el terminal ejecute el siguiente comando:

curl -i http://$CLOUDACADEMY_APP_IP

Seguir adelante. Cree copias especíﬁcas de dev y prod del archivo app/values.yaml. En el terminal ejecute el siguiente comando:

cp app/values.yaml app/values.dev.yaml 
cp app/values.yaml app/values.prod.yaml

Realice una reimplementación de dev en el clúster haciendo referencia al archivo values.dev.yaml Values dentro del comando helm template, ejecute el siguiente comando:

helm template cloudacademy -f ./app/values.dev.yaml ./app | kubectl apply -f -

Vuelva a probar el servicio cloudacademy-app enviándole otra solicitud curl. En el terminal ejecute el siguiente comando:

curl -i http://$CLOUDACADEMY_APP_IP

Realice una reimplementación de prod de nuevo en el clúster haciendo referencia al archivo values.prod.yaml Values dentro del comando helm template. En el terminal ejecute el siguiente comando:

helm template cloudacademy -f ./app/values.prod.yaml ./app | kubectl apply -f -

Vuelva a probar el servicio cloudacademy-app enviándole otra solicitud de curl. En el terminal ejecute el siguiente comando:

curl -i http://$CLOUDACADEMY_APP_IP

Utilice el comando helm para empaquetar la aplicación en un gráﬁco. En el terminal ejecute el siguiente comando:

helm package app/

Realice una lista de directorio para conﬁrmar que el gráﬁco se creó correctamente. En el terminal ejecute el siguiente comando:

ls -la

Antes de instalar el nuevo gráﬁco, invierta (elimine) todos los recursos desplegados anteriormente en Helm. En el terminal ejecute el siguiente comando:

helm template cloudacademy -f ./app/values.prod.yaml ./app | kubectl delete -f -

Instale el nuevo gráﬁco. En el terminal ejecute el siguiente comando:

helm install cloudacademy-app app-0.1.0.tgz

Enumere todas las versiones actuales de Helm para el contexto actual del espacio de nombres. En el terminal ejecute el siguiente comando:

helm ls

Muestre la implementación, el pod y los recursos de servicio que se crearon debido a la instalación del gráﬁco de Helm. En el terminal ejecute el siguiente comando:

kubectl get deploy,pods,svc

Pruebe el servicio cloudacademy-app enviándole una solicitud curl. En el terminal ejecute el siguiente comando:

curl -i http://$CLOUDACADEMY_APP_IP

La implementación consistió en un solo pod que contenía dos contenedores. El primer contenedor era un contenedor de servidor web NGINX que redirigía las solicitudes HTTP entrantes a un segundo contenedor que ejecutaba una aplicación web basada en FLASK, basada en la imagen de Docker que creó en el primer paso de laboratorio. Utilizamos yaml como archivo de valores para externalizar variables de tiempo de ejecución, asegurándose de que sus plantillas de Helm sean reutilizables en diferentes entornos, en este caso entornos dev y prod, a través de los archivos values.dev.yaml y values.prod.yaml respectivamente. 

### Code of conduct

Participation in the Helm community is governed by the [Code of Conduct](code-of-conduct.md).
