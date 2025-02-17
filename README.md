## Deploy-Kubernetes-using-Con-gMaps-and-Helm

Creating well-structured, declarative, and reusable deployments within Kubernetes is an essential skill that increases productivity, using a manifest file, which is used to declare cluster resources, along with Helm (v3), which can be used to template the manifests.
![helm](https://github.com/user-attachments/assets/140fcc1f-812f-446b-a459-969155dab672)

First of all, I thank Cloud Academy for teaching these topics: https://www.qa.com/

### What we are going to implement:
An application on a Kubernetes cluster using:
a. ConfigMaps : To manage application settings (e.g. environment variables, configurations).
b. Helm : As a package manager to declaratively define, install, and manage the application.
Install Chart on Kubernetes :
• Use Helm to install the application on the cluster.
Verify and Manage :
• Check the status of the application and its configuration.
• Update or remove the deployment as needed.
Benefits
• Configuration management : ConfigMaps allows you to separate configuration from code.
• Simplified installation and upgrade : Helm makes it easy to manage complex applications on Kubernetes.
• Declarative : You define the desired state, and Kubernetes and Helm take care of getting there.
For recording
• ConfigMaps : For configuration.
• Helm : To define and install applications on Kubernetes efficiently.

# Paso 1 
An NGINX web server container will be configured to redirect incoming HTTP requests downstream to another container running a custom FLASK-based web application. The NGINX web server container will use the publicly available nginx:1.13.7 image. The FLASK-based web application container will be based on a custom Docker image that you will need to build first.

a-Connect to IDE port 8080 of web-based containers
b-Perform image compilation and create namespace

# Paso 2
In this step you will create a custom Docker image containing a ﬂaskapp file. Based on a web application, the custom Docker image will later be deployed to the Kubernetes cluster. Before you deploy within the cluster, you will be shown how to create a new namespace resource and how to set it as the default namespace in which the remaining deployment takes place. List the contents of the ﬂaskapp directory. In the terminal run the following command:

```
ls -la
```

# Paso 3
Perform a Docker build to create a custom Docker image. In the terminal run the following command:

```
docker build -t cloudacademydevops/flaskapp .
```

Check for the presence of the newly created Docker image. In the terminal run the following command:

```
docker images
```

Create a custom kikis namespace inside the Kubernetes cluster. In the terminal run the following command:

```
kubectl create ns kikis
```

Switch to the new kikis namespace. In the terminal run the following command:

```
kubectl config set-context --current --namespace=kikis
```

# Paso 4 Aplicar Nginx ConﬁgMap

ConfigMaps allow you to decouple configuration artifacts from image content to maintain portability of containerized applications.
Change to the k8s directory and list its contents. In the terminal, run the following command:

```
cd ../k8s && ls -la
```

## The updated nginx.configmap.yaml manifest file is already in this repo.

Apply the updated nginx.configmap.yaml file to the K8s cluster: this will create a new ConﬁgMap resource. In the terminal run the following command:
```
kubectl apply -f nginx.configmap.yaml
```
List all ConfigMap resources in the K8s cluster. In the terminal run the following command:
```
kubectl get configmaps
```
## At this step you have updated, saved, and applied the nginx.configmap.yaml manifest file to the K8s cluster. 
This created a new ConfigMap containing the NGINX configuration that will be mounted in the NGINX container. 

# Paso 5 Aplicar implementación

Deployments represent a set of multiple identical pods and have the ability to automatically replace instances that fail or become unresponsive.

## In this repo the deployment.yaml manifest file is already updated

Let's apply the updated deployment.yaml file to the K8s cluster, this will create a new Deployment file.
In the terminal run the following command:
```
kubectl apply -f deployment.yaml
```

Let's number all the deployments in the K8s cluster. In the terminal run the following command:
```
kubectl get deploy
```

List all the pods in the K8s cluster. In the terminal run the following command:
```
kubectl get pods
```
Extract the name of the frontend pod and store it in a variable called POD_NAME. Echo it back to the terminal. In the terminal run the following commands:
```
POD_NAME=`kubectl get pods -o jsonpath='{.items[0].metadata.name}'`echo$POD_NAME
```
Now use the kubectl describe command to get a detailed report on the current status of the frontend pod. In the terminal run the following command:
```
kubectl describe pod $POD_NAME
```
Use the docker images command to confirm the correct Docker image tag for the FLASK-based web application you created in step 1 of the lab. In the terminal, run the following command:
```
docker images | grep cloudacademydevops
```
List all deployment resources in the K8s cluster. In the terminal run the following command:
```
kubectl get deploy
```
Expose the frontend deployment. This will create a new Service that will allow calling frontend pods through a stable cluster network VIP address. In the terminal run the following command:
```
kubectl expose deployment frontend --port=80 --target-port=80
```
List the new frontend service resource created above. In the terminal run the following command:
```
kubectl get svc frontend
```
Extract the IP address of the frontend service cluster and store it in a variable called FRONTEND_SERVICE_IP. Echo it back to the terminal. In the terminal run the following commands:
```
FRONTEND_SERVICE_IP=`kubectl get service/frontend -o jsonpath='{.spec.clusterIP}'` echo$FRONTEND_SERVICE_IP
```
Test the frontend service by sending a curl request to it. In the terminal run the following command:
```
curl -i http://$FRONTEND_SERVICE_IP
```
Retest the frontend service by sending a new curl request to it. In the terminal run the following command:
```
curl -i http://$FRONTEND_SERVICE_IP
```
Let’s now examine the logs associated with sending traffic through the NGINX web server. To start, list the current pods within the cluster. In the terminal, run the following command:
```
kubectl get pods
```
Extract the name of the frontend pod and store it in a variable called FRONTEND_POD_NAME. Echo it back to the terminal. In the terminal run the following commands:
```
FRONTEND_POD_NAME=`kubectl get pods --no-headers -o custom-columns=":metadata.name"` echo$FRONTEND_POD_NAME
```
Perform a directory listing directly inside the NGINX container by listing the contents of the /var/log/nginx directory. In the terminal run the following command:
```
kubectl exec -it $FRONTEND_POD_NAME -c nginx -- ls -la /var/log/nginx/
```
Use the kubectl logs command to examine the NGINX log generated by the curl commands executed above. In the terminal, run the following command:
```
kubectl logs $FRONTEND_POD_NAME nginx
```
Use the kubectl logs command to examine the FLASK log generated by the curl commands executed above. In the terminal, run the following command:
```
kubectl logs $FRONTEND_POD_NAME flask
```

# Paso 6 Crear proyecto de Helm

To start, navigate up one directory to the directory:
```
cd ..
```
Use the helm create command to generate a new Helm chart project named test-app. In the terminal, run the following command:
```
helm create test-app
```
Use the tree command to render the directory structure on the screen. In the terminal run the following command:
```
tree test-app/
```
Use the helm template command to convert Helm templates into a single deployable Kubernetes manifest file. In the terminal, run the following command:
```
helm template test-app/
```
In this step, you learned how to use the Helm method

# Step 7 Update the deployment to use Helm templates

In this scenario, the provided application Helm chart has been configured to create a deployment that launches a single pod containing two containers. The first container is an NGINX web server container that sends incoming HTTP requests to a second container running a FLASK-based web application (based on the Docker image you created earlier).

Start by deleting all previous resources started within the cluster. Use the kubectl delete command to delete the previously created deployment, pod, and service resources. In the terminal run the following command:
```
kubectl delete deploy,pods,svc --all
```
## In this repo the Helm values.yaml file and the deployment.yaml file are already modified

Use the helm template command to generate a deployable manifest file. In the terminal, run the following command:
```
helm template ./app
```
With this command you ONLY generated the deployable manifest, it has not been deployed to the cluster yet, this is useful to validate the resulting manifest BEFORE actually deploying it.

This time you will perform the actual deployment to the cluster. To do this, you will again use the helm template command to generate a deployable manifest file, piping the output (manifest) directly to the kubectl apply command. In the terminal run the following command:
```
helm template kikis ./app | kubectl apply -f -
```
Examine the current services available within the cluster. In the terminal run the following command:
```
kubectl get svc
```
Extract the IP address of the cloudacademy-app service cluster and store it in a variable named KIKIS_APP_IP. Echo it back to the terminal. In the terminal run the following commands:
```
KIKIS_APP_IP=`kubectl get service/kikis-app -o jsonpath='{.spec.clusterIP}'`
echo $KIKIS_APP_IP
```
Test the kikis-app service by sending a curl request to it. In the terminal run the following command:
```
curl -i http://$KIKIS_APP_IP
```

```
cp app/values.yaml app/values.dev.yaml 
cp app/values.yaml app/values.prod.yaml
```
Perform a dev redeployment to the cluster by referencing the values.dev.yaml Values ​​file within the helm template command, run the following command:
```
helm template kikis -f ./app/values.dev.yaml ./app | kubectl apply -f -
```
Retest the cloudacademy-app service by sending another curl request to it. In the terminal run the following command:
```
curl -i http://$KIKIS_APP_IP
```
Perform a prod redeployment back to the cluster by referencing the values.prod.yaml Values ​​file within the helm template command. In the terminal run the following command:
```
helm template kikis -f ./app/values.prod.yaml ./app | kubectl apply -f -
```
Retest the cloudacademy-app service by sending it another curl request. In the terminal, run the following command:
```
curl -i http://$KIKIS_APP_IP
```
Use the helm command to package the application into a graph. In the terminal, run the following command:
```
helm package app/
```
Perform a directory listing to confirm that the graph was created successfully. In the terminal run the following command:
```
ls -la
```
Before installing the new chart, revert (remove) all previously deployed resources in Helm. In the terminal, run the following command:
```
helm template kikis -f ./app/values.prod.yaml ./app | kubectl delete -f -
```
Install the new graphic. In the terminal run the following command:
```
helm install kikis-app app-0.1.0.tgz
```
List all current versions of Helm for the current namespace context. In the terminal run the following command:
```
helm ls
```
Show the deployment, pod, and service resources that were created due to the Helm chart installation. In the terminal, run the following command:
```
kubectl get deploy,pods,svc
```
Test the cloudacademy-app service by sending a curl request to it. In the terminal run the following command:
```
curl -i http://$KIKIS_APP_IP
```

The deployment consisted of a single pod containing two containers. The first container was an NGINX web server container that redirected incoming HTTP requests to a second container running a FLASK-based web application, based on the Docker image you created in the first lab step. We used yaml as a values ​​file to externalize runtime variables, ensuring that your Helm templates are reusable across different environments, in this case dev and prod environments, via the values.dev.yaml and values.prod.yaml files respectively.

## Code of conduct

Participation in the Helm community is governed by the [Code of Conduct](code-of-conduct.md).
