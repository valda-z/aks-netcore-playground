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
* `curl -s https://raw.githubusercontent.com/valda-z/acs-cicd/master/run.sh | bash -s -- --resource-group KUBE --kubernetes-name valdakube --acr-name valdakubeacr --postgresql-name valdakubepostgres --jenkins-password kube123`
* supported parameters are:
 * Mandatory
     * `--resource-group` - Azure resource group name (will be created by script)
     * `--kubernetes-name` - Unique name for kubernetes cluster 
 * Optional (if not provided than script uses defaults - see script)
     * `--location` - Azure region for resource group and all resources 

### After successful deployment:
* deployment script will show necessary information how to access our micro-service application

There is sample output:

```
### DONE
### URL for your application is http://valdaaks02-6de04cb8.westeurope.cloudapp.azure.com after deployment

```


 



