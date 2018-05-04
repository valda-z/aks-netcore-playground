# AKS .Net Core playground - Kubernetes (AKS), VSTS, ACR, helm

This demo contains helm templates and description of k8s use-cases which can be used with .Net Core dockerized services (linux based containers).

## Demonstrated scenario:
* automatically create infrastructure environment AKS (kubernetes), ACR.
* deploy nginx-ingerss controller to kubernetes cluster for future use

### Solution can be provisioned by this simple script from Azure Cloud Shell:
* **prerequisites**
 * you need your SSH (private and public) key prepared in cloud shell (description how to generate keys: https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#platform-linux )
 * public ssh key is in file `~/.ssh/id_rsa.pub` 
* run Azure Cloud Shell
* ![img1.png](img/img1.png "")
* There run install script and provide necessary parameters
* `curl -s https://raw.githubusercontent.com/valda-z/aks-netcore-playground/master/run.sh | bash -s -- --resource-group KUBE --kubernetes-name valdakube `
* supported parameters are:
 * Mandatory
     * `--resource-group` - Azure resource group name (will be created by script)
     * `--kubernetes-name` - Unique name for kubernetes cluster 
 * Optional (if not provided than script uses defaults - see script)
     * `--location` - Azure region for resource group and all resources 

### After successful deployment:
* deployment script will show necessary information how to access our micro-service application

There is sample output - please store DNS name for service [APP DNS NAME]:

```
### DONE
### URL for your application is http://valdaaks02-6de04cb8.westeurope.cloudapp.azure.com after deployment

```

## Experiments

### #1 Deploy Replica Set and Service

#### create dockercloud replicaset and service

Create and validate replica set with simple http service and then create service with external load balancer and public IP address.
ReplicaSet and Service can be created by scripts or directly from Kubernetes control plane.

##### dp-rs.yaml
```yaml
apiVersion: extensions/v1beta1
kind: ReplicaSet
metadata:
  name: dockercloud
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: dockercloud
    spec:
      containers:
        - name: hostname
          image: dockercloud/hello-world
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
```

##### dp-svc.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: dockercloud
  name: dockercloud
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
    name: http
  selector:
    app: dockercloud
  type: LoadBalancer
```

##### run commands ..

`kubectl create -f dp-rs.yaml`

`kubectl create -f dp-svc.yaml`

Or you can use kubernetes control plane for creating replicaset and service

##### wait for provisioning of services

You can check status of provisioning by running command (or you can use kubernetes control plane for it):

`kubectl get svc`

### #2 Create ASP.NET Core Web Application in Linux Docker

#### Create new project
* Use new project, select type "ASP.NET Core Web Application"
* In project detail page select "ASP.NET Core 2.0" and project type "API"
* Open Controller in your project and change Get method to return some environment variable:

```cs
[HttpGet("{id}")]
public string Get(int id)
{
    return Environment.GetEnvironmentVariable("MYTESTENVIRONMENT");
}
```
* Insight your Controller create method to handle default path "/" - kubernetes will use this entrypoint for health probe.

```cs
[HttpGet]
[Route("/")]
public string Probe()
{
    return "OK";
}
```

* build and test project (also you can test setting environment variable `MYTESTENVIRONMENT` for project

#### Enable docker support for project
* Select "Docker support" from "Project" menu
* Select type "Linux"
* Visual Studio will create for you dockerfile and now you can build and debug your application in running docker (select right target for debug).

#### Publish docker image to Azure Container Registry

##### Visual Studio 2017
* Select "Publish" option on project
* Create new publishing profile for Container Registry and select your existing Azure Container Registry in your subscription
* Publish image


##### Alternative process from command line
* Create tag on your docker image, first of all list images by `docker images` and than `docker tag [YOUR APP IMAGE] [YOUR REGISTRY NAME].azurecr.io/[YOUR APP IMAGE]`
* login to container registry `docker login [YOUR REGISTRY NAME].azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword` - credentials for registry are visible in Azure Portal.
* push image to registry `docker push [YOUR REGISTRY NAME].azurecr.io/[YOUR APP IMAGE]`

#### install application and configure security configmap

##### Create secrets for application
Create secrets from kubectl command line, secret name and secret keys correlates to  helm chrt for our app:

`kubectl create secret generic myrelease-mynetcoreapp --from-literal=mytestenvironment="My supersecret string"`

##### install application with helm
Run following command to install application

`helm upgrade --install --wait myrelease mynetcoreapp --set-string imagePullSecrets='[YOUR REGISTRY NAME].azurecr.io',image.repository='[YOUR REGISTRY NAME].azurecr.io/[YOUR APP IMAGE]',image.tag='[BUILDNUMBER]',track=stable,branchName='master',branchSubdomain='',ingress.host='[APP DNS NAME]' --namespace='default'`

Now we can test our api on URL:

http://[APP DNS NAME]/myapi1/values/1

Clean-up deployment after tests..

`helm del --purge myrelease` 





 



