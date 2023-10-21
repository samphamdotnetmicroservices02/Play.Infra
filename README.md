# Play.Infra
Play Economy Infrastructure components

## Add the Github package source
```powershell
$owner="samphamdotnetmicroservices02"
$gh_pat="[PAT HERE]"

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"

dotnet nuget list source (check result)
```

```zsh
owner="samphamdotnetmicroservices02"
gh_pat="[PAT HERE]"

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"

dotnet nuget list source

dotnet nuget list source (check result)
```

--username: the username that you are going to use to connect to GitHub packages. Now the interesting thing here
is that when you use personal access tokens, it does not really matter what username you use here, because the
access tokens has all the information needed to authenticate you. So you can use anything here, I will just put
username

--password: the password is going to be our brand new GitHub personal access token

--store-password-in-clear-text: This is a requirement from NuGet in terms of how to store this password in the local box.

--name: The name is going to be the name of the source. Lets name it just GitHub

lastly, you have to specify the address of this new source. So the address for GitHub packages has to be 
https://nuget.pkg.github.com/

after add to nuget, use "dotnet nuget list source" to check the result

## Using Azure
az login (login to azure)
az account set --subscription "[name or subscriptionId]" (choose subscription)
az account show (show the subscription you went for)

## Creating the Azure resource group
```powershell
$appname="playeconomy"
az group create --name $appname --location eastus

az group create: create azure resource group
--location: you can choose the location closest to you to reduce latency

Check resource group was created, go to azure -> subscriptions -> Resource groups

One quick note for the case of this course, I have decided to put all the resources for all micrservices in just one resource
group, but in your case, it could be that you want to define different resource groups for different microservices. So that
will give you a much better level of access control to the resources that each microservice uses. In our case, we are going
to be actually sharing a bunch of things as you are going to see. So I am just going to keep things in one resource group,
which is going to be these play economy resource group.
```

```zsh
appname="playeconomy"
az group create --name $appname --location eastus
```

## Creating the Cosmos DB account
```powershell
$dbAccountName="samphamplayeconomy"
az cosmosdb create --name $dbAccountName --resource-group $appname --kind MongoDB --enable-free-tier

--name: create the name of db account

--resource-group: your resource group name

--kind MongoDB: this is where it tends to be that a Cosmos DB support multiple what they call APIs. And so besides the default API
that they support that they also call, call the core or SQL API, which is a more native one that they support. They also have
APOS for MongoDB, for Cassandra, for Gremlin table API. And there may be even more by the time that you take this course. Now
in our case, since we have been dealing mostly with Mongo DB, and we have been using the Mongo DB client, the API that really 
makes sense, is the Mongo DB API. So these will allow us to just change the connection string that we have been using so far
into a connection string that connects to cause with Cosmos DB and that is all it is. It will be able to start using Mongo Db
as if it was a local container that we are using so far. However, if you want a more native experience, because maybe you want 
to take advantage of some bleeding edge feature of Cosmos DB that are not available for Mongo DB yet, or something only
available in the ghost of the client, you may want to just remove these parameter (--kind MongoDB) that will take you to the
default API. But then in that case, you will have to likely a create a new type of repository. Eh, that is not going to be using 
the Mongo client, but it is going to be using the Cosmos DB client in that case.

--enable-free-tier: this will enable free tier, which will give you the first 1000 request units. And the first 35 gigabytes
of storage are going to be totally free for you. So you will not have to pay for those. So this is a great option for 
learning purposes development or testing purposes, where you are just evaluating if Cosmos DB worked for you.

check your mongo db on azure by navigate to subscriptions -> Resource groups -> your subscription (playeconomy) and you will
see the "samphamplayeconomy" which is your cosmos db
```

```zsh
dbAccountName="samphamplayeconomy"
az cosmosdb create --name $dbAccountName --resource-group $appname --kind MongoDB --enable-free-tier
```

## Creating the Service Bus namespace
```powershell
$serviceBusName="samphamplayeconomyservicebus"
az servicebus namespace create --name $serviceBusName --resource-group $appname --sku Standard

--sku: Azure Service Bus supports multiple SKUs that offer different levels of features, and of course, will cost
more or less money depending on what features you want to accquire. There are at least three SKUs here available
at the time as I am recording this lesson, which are basic, standard, and premium. And the basic one that costs 
less money. However, in order for you to be able to use Service Bus with MassTransit, you need to have support
for both queues and topics in Service Bus. And unfortunately, tocpics are not supported in the basic SKU. So
you have to use at least the Standard SKU, so that you are able to use MassTransit with Service Bus.


to check service bus you just created, navigate to subscription -> your resource group -> samphamplayeconomyservicebus
```

```zsh
serviceBusName="samphamplayeconomyservicebus"
az servicebus namespace create --name $serviceBusName --resource-group $appname --sku Standard
```

## Creating the Container Registry
```powershell
$acrName="samphamplayeconomyacr"
az acr create --name $acrName --resource-group $appname --sku Basic

--sku: ACR has as any other Azure service, there are tiers or levels of service that you can get with the ACR service.
So ACR has three levels at the time as I am recording this lesson, which are basic, standard, and premium. And so the
basic offering, actually, I mean all these offerings have the same programmatic and access and features, but it is just
that the more cheaper they are, they offer a less amount of storage for your containers and they do not offer that
much throughput for either pulling or pushing your images to the ACR. So for the purpose of this course for learning
purposes, using the basic tier is totally fine, that will fork just great for you. But as you move into an actual
production workload, you may want to consider using the standard SKU, because it has increased storage for your
images of which you are going to grade a lot and increase it throughput to pull ambush images.
```

```zsh
acrName="samphamplayeconomyacr"
az acr create --name $acrName --resource-group $appname --sku Basic
```

## Creating the AKS cluster
```powershell
$aksName="samphamplayeconomyaks"

az aks create -n $aksName -g $appname --node-vm-size Standard_B2s --node-count 2 --attach-acr $acrName --enable-oidc-issuer --enable-workload-identity --generate-ssh-keys

go to your subscriptions, your resource groups, your aks to check your aks cluster.

az aks get-credentials --name $aksName --resource-group $appname

kubectl version

kubectl cluster-info

-n: aks name
-g: resource group name 

--node-vm-size Standard_B2s: The next thing I need to specify is the VM size or the SKU. So as you might know, there are multiple types
of virtual machine sizes in Azure. There are dozens and dozens of ones which have different specs in terms of CPU, RAM, and GPU,
disk space, all these things, and you can choose from them. And a Kubernetes cluster is going to need virtual machines, even if you are
not going to be seeing that, really, it is a little bit abstracted from you. But the cluster will need those virtual machines to put
your containers in that, right? And run them in there. So you have to choose some sort of size of the virtual machine. And so, for
our learning purposes, I have chosen the size that I think is the cheapest one that we can use with the specs that are going to be 
useful for our purpose (Standard_B2s).
And it would be the case also that this size (Standard_B2s) is not available in your subscription because it depends on subscriptions are
a bit special and they do not have all the SKUs available. And also, it could be that it is not available for the region that you are 
trying to use. So I am using, If I am not wrong, I am using eastus

--node-count: this is the number of nodes or VMs that are going to be assigned to the Kubernetes cluster. So you can go as low as one
and as high, I do not actually know how many you, there is a limit but you cannot go unlimited, right? Now, for our purposes, we are
going to be choosing two. We need two nodes because the more containers that you add to your cluster, the more CPU and RAM is being
used from that machine and that machine does not have unlimited resources, so at some point you have to jump into another VM to be
able to properly distribute all your containers. So we will not just be deploying our own microservice containers in here but also
the containers needed for a few other services that we are going to be using or deploying into Kubernetes, so we have enough of those
to require at least two VMs. You could use more if you want to but two should be good enough.

--attach-acr $acrName: The next thing that we are going to need is one more argument to grant permission to this AKS cluster to connect 
to our ACR. Remember that a moment, in a previous lesson, we created a container registry. Now our Kubernetes cluster is going to need 
to connect to that registry to download the container images into the nodes that are part of the cluster, so that it can run the 
containers. To make this simpler and just grant access right away to the cluster as soon as it is created.

--enable-oidc-issuer: this is tied to this feature of AD workload identities. This argument is required to enable that feature. Like
I said, we are goint to talk more about this later on.

--enable-workload-identity: we talk about this in a future lessons.

--generate-ssh-keys: you can authenticate and connect to the AKS cluster.

az aks get-credentials: Get credentials from Kubernetes into you local box, so that we can connect to the cluster. Connect to the cluster
--name $aksName: name of the Kubernetes cluster

kubectl version: kubectl (kube control) command comes from az or docker desktop, this verifies that you are able to use the CLI of Kubernetes.
So, there is a, just like we have a CLI for .NET and a CLI for Azure, there is a CLI also for Kubernetes (kubectl). This command just verifies
that things are working properly with Kubernetes. (run command az aks get-credentials to connect the cluster and after that, run this command)

kubectl cluster-info: to check that things are working properly. This will give you a few more details about different components of Kubernetes
like the Kubernetes control plane, core DNS, metric server 
```

```zsh
aksName="samphamplayeconomyaks"

az aks create -n $aksName -g $appname --node-vm-size Standard_B2s --node-count 2 --attach-acr $acrName --enable-oidc-issuer --enable-workload-identity --generate-ssh-keys

az aks get-credentials --name $aksName --resource-group $appname

kubectl version

kubectl cluster-info
```

## Creating the Azure Key Vault
```powershell
$keyVaultName="samphamplayeconomykv"
az keyvault create -n $keyVaultName -g $appname

-n: the name of Key Vault

after that, go to Azure Cloud to set value of secret in "Secrets" tab of $keyVaultName you just created based on each microservice like
MongoDbSettings__ConnectionString, ServiceBusSetting__ConnectionString and change underscores "__" to double dash "--"
```

## Installing Emissary-ingress
https://helm.sh/docs/intro/install/
https://www.getambassador.io/docs/emissary/latest/topics/install/helm
follow this guideline to install helm: https://learn.dotnetacademy.io/courses/take/net-microservices-cloud/lessons/38615223-installing-helm

```powershell
helm repo add datawire https://app.getambassador.io (confirm that by using "helm repo list")
helm repo update

The first thing that we have to do is to let our helm tool locally know about the helm repository where the helm charts that we need, actually exist.
And in this case, that location is going to be in https://app.getambassador.io. So that is where the helm chart is. So we have to let our helm
repo know about that location.


kubectl apply -f https://app.getambassador.io/yaml/emissary/3.8.1/emissary-crds.yaml 
kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system

We have to configure the Kubernetes cluster to support both the "getambassador.io/v3alpha1" and "getambassador.io/v2" versions of configuration resources.
So this is a required step, probably temporal for now as people is switching between v2 and v3 and alpha one. So what we are going to do is just follow
these commands as they are.

kubectl apply -f https://app.getambassador.io/yaml/emissary/3.8.1/emissary-crds.yaml: this installs what we know as custom resources. Custom resource 
definitions for emissary-ingress is one type of resource in kubernetes.

kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system: wait for those resource to be available in the cluster.
Also notice that we will be doing this in to the "-n emissary-system" namespace. So this is the namespace that one of the name spaces that emissary-ingress
will be using in the kubernetes cluster for all of their components. After run this command we will see "deployment.apps/emissary-apiext condition met",
it means those resources are ready to go in the cluster.


$emissarynamespace="emissary"
$apiGetwayAppname="samphamplayeconomyapigateway"
$apiGatewayAppNameLocal="playeconomyapigateway"
helm install emissary-ingress datawire/emissary-ingress --set service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"=$apiGetwayAppname -n $emissarynamespace --create-namespace

helm list -n $emissarynamespace (verify Helm release from command above)

kubectl rollout status deployment/emissary-ingress -n $emissarynamespace -w

kubectl get pods -n $emissarynamespace (check your pod with emissary)

kubectl get service emissary-ingress -n $emissarynamespace


the step of installing emissary-ingress in the box using helm.

helm install ...: "helm install emissaray-ingress", so this is going to create what we know as a release and that Helm release is going to be named
emissary-ingress and it is coming from the "datawire/emissary-ingress" helm repository. And then it is going to be installed as it is saying right here
into the emissary namespace (-n emissary). And if that name space does not exist, it is going to create it the very first time "--create-namespace"

--set service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name": specify an annotation that is going to be able to tell the AKS infrastructure
how to associate the DNS address with the IP address that is going to provision for our API gateway. Remember that there is going to be just one IP address
that our clients are going to use to reach out to the API gateway and from there Api gateway routes into the internal microservices, but we do not want them
to use just IP addresses. We want them to use a proper DNS name to reach out to the API gateway. So this annotation that we are going to write is what is
going to enable that. "--set" This allows to override values in that chart that we are installing. So there are multiple values that you can override and 
one of them is the annotations "service.annotations...". And then goes equals to whatever dns we want to provide into our public IP address. So that DNS
that we are going to use is $emissarynamespace (This value must be unique across the Azure region you are using. If you use a value already in use in your
region, Azure will not provision a public IP for Emissary-Ingress and you will not be able to access you Api gateway). So like I said this specific portion
here just overrides one of the values of this helm chart with another value that I want to use there which will be blank if we did not specify it. But with
this we add a DNS name to our IP addresses for our API gateway

kubectl rollout status deployment/emissary-ingress -n $emissarynamespace -w: check the status of rollout

kubectl get pods -n emissary: at least 4 emissary, one of the emissary display on your terminal is agent, and others are pods

kubectl get service emissary-ingress -n emissary: figure out what the service that is associated to this Api gateway (emissary)
after run the command above: "NAME: emissary-ingress" is the service, TYPE: LoadBalancer, CLUSER-IP: internal IP, EXTERNAL-IP: external IP which the clients
are able to use to reach out our Api gateway and from that into our microservices. but as I mentioned before, there should also be a proper DNS address
associated to this external IP.

check your Api Gateway on Azure: you will see the EXTERNAL-IP from the command above, for example EXTERNAL-IP is: "20.237.70.13". Navigating to your resource
group -> hit "MC_${resource_group}_${aksName}_${location}", there are many "kubernetes-${guid}" there, check each of them with the "IP address" you just created
Api gateway for example EXTERNAL-IP is "20.237.70.13", and next to "IP address" is DNS name.
```

```zsh
helm repo add datawire https://app.getambassador.io (confirm that by using "helm repo list")
helm repo update

kubectl apply -f https://app.getambassador.io/yaml/emissary/3.8.1/emissary-crds.yaml 
kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system

emissarynamespace="emissary"
apiGetwayAppname="samphamplayeconomyapigateway"
helm install emissary-ingress datawire/emissary-ingress --set service.annotations."service\.beta\.kubernetes\.io/azure-dns-label-name"=$apiGetwayAppname -n $emissarynamespace --create-namespace

helm list -n $emissarynamespace (verify Helm release)

kubectl rollout status deployment/emissary-ingress -n $emissarynamespace -w

kubectl get pods -n $emissarynamespace (check your pod with emissary)

kubectl get service emissary-ingress -n $emissarynamespace
```

## Configuring Emissary-ingress routing
```powershell
kubectl apply -f ./emissary-ingress/listener.yaml -n $emissarynamespace
kubectl apply -f ./emissary-ingress/mappings.yaml -n $emissarynamespace
```

## Installing Cert-manager
https://cert-manager.io/docs/installation/helm/

```powershell
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager --version v1.13.0 --set installCRDs=true --namespace $emissarynamespace

kubectl get pods -n $emissarynamespace (check api gateway after run command above)
```

"helm repo add", "helm repo update": Just like we did with emissary-ingress, we also need to first add the corresponding Helm repository that we will calling jetstack, so that our local
Helm installation knows how to fetch Helm charts that are needed for cert-manager.

helm install cert-manager jetstack/cert-manager: "helm install" is installation of Helm chart into our box that we're going to name "cert-manager",
and this is coming from "jetstack/cert-manager" repository. "--set installCRDs=true" is just overriding a value in the helm chart where we want
to say that we want to automatically install the custom resource definitions automatically. So that way we don't have to run yet another Kubernetes line to
install the customer resource definitions. "--namespace cert-manager --create-namespace" which by default would put all cert-manager reosurces in a 
brand new cert-manager namespace, however having cert-manager in a different namespace than the one use to generate your first certificate would likely cause
issues with initialization that cert-manager must perform to ensure that we own the domain that will use in our certificates. For this reason and to keep
things simple, we will just place cert-manager in the same namespace we are using for emissary. After the first certificate is generated, no more validation
is needed and you will be able to generate certificates in any other namespace. So since that is the case, what we're going to do is just grab the name space
variable that we have in emissary $emissarynamespace. And since this name space should exist by now we will remove this "--create-namespace"

kubectl get pods -n $emissarynamespace: check api gateway after run command above "helm install cert-manager ...". The main pod is cert-manager-{numbers}-..., 
for example "cert-manager-64d969474b-jg6lc". And then ther's usually two more pods that we will not get really into details in this course, but these are
three pods that we expect to see up and running for cert-manager.

## Creating the cluster issuer
```powershell
kubectl apply -f ./cert-manager/cluster-issuer.yaml -n $emissarynamespace

kubectl get clusterissuer -n $emissarynamespace (check your result from command above)
kubectl describe clusterissuer -n $emissarynamespace
kubectl delete clusterissuer letsencrypt-prod -n $emissarynamespace (delete if failed)

kubectl apply -f ./cert-manager/acme-challenge.yaml -n $emissarynamespace
kubectl delete service acme-challenge-service -n $emissarynamespace (delete if failed)
kubectl delete mapping acme-challenge-mapping -n $emissarynamespace (delete if failed)
```

## Creating TLS certificate
```powershell
kubectl apply -f ./emissary-ingress/tls-certificate.yaml -n $emissarynamespace

kubectl get certificate -n $emissarynamespace (verify your command above)
kubectl describe certificate playeconomy-tls-cert -n $emissarynamespace (check detail your certificate)
kubectl describe challenge -n $emissarynamespace (check your challenge success or not)
kubectl get secret -n $emissarynamespace
kubectl get secret playeconomy-tls -n $emissarynamespace -o yaml
kubectl delete cert playeconomy-tls-cert -n $emissarynamespace (delete your tls-cert if failed)
```

"kubectl apply -f ./emissary-ingress/tls-certificate.yaml ...": This will kick off the whole process of the acme challenge, acme http01 challenge. 
So that cert manager is going to go ahead and try to verify that we actually own this domain that we claim that we own by sending that request 
into our cluster via the API gateway and then into the service and into this temporal port. And when all of that is resolved, cert managaer 
should be able to go ahead and provision the certificate that we have requested.

"kubectl describe certificate playeconomy-tls-cert ...": the "playeconomy-tls-cert" is the name after you run "kubectl get certificate ...", or
the name you define tls-certificate.yaml (metadata.name)

"kubectl get secret -n $emissarynamespace": see secret that has been created

"kubectl get secret playeconomy-tls -n $emissarynamespace -o yaml": get detail your specified secret. "playeconomy-tls" is one of the secrets
and it comes from tls-certificate.yaml "spec.secretName". "-o yaml" to see the yaml view of this Kubernetes resource. After running this command
you will see a very long string of encoded characters coming from "data:tls.crt", this contains all the data for the crt portion of the generated
certificate. And the "data:tls.key", this is the other big part of that certificate. And this is what we expect for that certificate that we can
use in Kubernetes with emissary ingress.

## Enabling TLS and HTTPS
```powerhsell
kubectl apply -f ./emissary-ingress/host.yaml -n $emissarynamespace

after this step, you can use HTTPS for Api gateway and configure forwarded headers for each microservice
```

## Run Kubernetes on local machine
change your cust dns hostname
On Windows, the hosts file is located at C:\Windows\System32\drivers\etc\hosts.
On macOS and Linux, it's located at /private/etc/hosts.

```powershell
kubectl apply -f https://app.getambassador.io/yaml/emissary/3.8.1/emissary-crds.yaml 
kubectl wait --timeout=90s --for=condition=available deployment emissary-apiext -n emissary-system

$emissarynamespace="emissary"
$apiGatewayAppNameLocal="playeconomyapigateway"
helm install emissary-ingress datawire/emissary-ingress --set nameOverride=$apiGatewayAppNameLocal -n $emissarynamespace --create-namespace

For mac
helm install emissary-ingress datawire/emissary-ingress --set ingress.hostname=$apiGatewayAppNameLocal -n $emissarynamespace --create-namespace

For window (because when running docker on wondow the "wslrelay.exe" and "com.docker.backend.exe" will use the port 443. Hence when we install emissary-ingress
through the command line below, it will return an error - Run this command line to see the result " ")
helm install emissary-ingress datawire/emissary-ingress --set ingress.hostname=$apiGatewayAppNameLocal,ingress.port=8443 -n $emissarynamespace --create-namespace

helm list -n $emissarynamespace (verify Helm release from command above)

kubectl rollout status deployment/emissary-ingress -n $emissarynamespace -w

kubectl get pods -n $emissarynamespace (check your pod with emissary)

kubectl get service -n $emissarynamespace


kubectl apply -f ./emissary-ingress/listener-local.yaml -n $emissarynamespace
kubectl apply -f ./emissary-ingress/mappings-local.yaml -n $emissarynamespace

kubectl apply -f ./emissary-ingress/host-local.yaml -n $emissarynamespace
```


## Check logs when the Kubernetes always in ContainerStarting or something else
kubectl get events --sort-by=.metadata.creationTimestamp

## Set service for yaml file
kubectl edit service emissary-ingress -n $emissarynamespace

## Check log cluster
kubectl cluster-info dump

## Packaging and publishing the microservice Helm chart
The first instruction that we have to follow here is the one that you used to package the helm chart. So the helm chart itself
has to be packaged in to a compressed file that later on you can go ahead and push or publish into the ACR
```powershell
helm package ./helm/microservice

$helmUser=[guid]::Empty.Guid (or helmUser=00000000-0000-0000-0000-000000000000)
$helmPassword=az acr login --name $acrName --expose-token --output tsv --query accessToken

helm registry login "$acrName.azurecr.io" --username $helmUser --password $helmPassword (login to ACR)

helm push microservice-0.1.2.tgz oci://$acrName.azurecr.io/helm
```
- $helmUser ...: Because we're going to acr, so we don't need to specify which username here. But we need to follow the
convention, so we put Guid.Empty in the username.
- $helmPassword ...: get password to login acr, --output tsv: the format from the output "--expose-token" is not approiate 
to be used as the argument for the next line. So let's actually modify the output a little bit by using the "--output tsv"
argument. So that it will give you a string that we can use in the next command.
- --query accessToken: accessToken is one component of that output. So we only get only that piece as a string that we can 
user later on
- helm push ...: push your compressed helm file to ACR. "oci" the format that you need to use to address the actual container
registry has to start with oci://$acrName.azurecr.io/helm. After pushing to ACR, you can verify that on Repositories -> 
helm/microservice

```zsh
helm package ./helm/microservice

helmUser=00000000-0000-0000-0000-000000000000
export helmPassword="$(az acr login --name $acrName --expose-token --output tsv --query accessToken)"

helm registry login "$acrName.azurecr.io" --username $helmUser --password $helmPassword (login to ACR)
helm push microservice-0.1.2.tgz oci://$acrName.azurecr.io/helm
```

## Create github service principal
In order to authenticate our Github repository into Azure, what we need to do is to define a service principal
which represents kind of an identity for Github repository in Azure subscription.
```powershell
$resourceGroupId=az group show --name $appname --query id --output tsv
$appId=az ad sp create-for-rbac -n "GitHub" --query appId --output tsv
or
$appId=az ad sp create-for-rbac -n "GitHub" --scopes $resourceGroupId --role Contributor --query appId --output tsv

"scope is required for creating role assignment in fall of 2023."
$ACR_ID=$(az acr show --name $ACR_NAME --query id --output tsv)
az role assignment create --assignee $appId --role "AcrPush" --resource-group $appname
or
az role assignment create --assignee $appId --role "AcrPush" --scope $ACR_ID --resource-group $appname

az role assignment create --assignee $appId --role "Azure Kubernetes Service Cluster User Role" --resource-group $appname
az role assignment create --assignee $appId --role "Azure Kubernetes Service Contributor Role" --resource-group $appname

One more thing you need to do is to establish what is named as federated credentials. What we have to specify "add new a new credential" that defines the specific details about the repository that needs to get access via this application into Azure. Navigating to your Azure Directory --> click on "GitHub" service principal --> click on Certificates & secrets --> Federated credentials --> click on Add credential --> select "GitHub Actions deploying Azure resources" --> fill out some information: organization: "samphamdotnetmicroservices02", repository: "Play.Identity" (your repository name), Entity type: "Branch" (whenever we make changes to "main" branch the github publish to Azure), Github branch name: "main". Credential details section, name: "Play.Identity", description: "Identity microservice repository". Click on Add. 
If you want to use this service pricipal "GitHub" for other services, you just add more federated credentials for these services.
```
- az ad sp create-for-rbac: "ad": Azure Directory, "sp": Service Principal, "create-for-rbac" meaning that this service principal is going to be created for the purposes of role based access control. "-n 'GitHub'":  you can pick any name, we pick "GitHub" because this is going to represent the Github repository in the Azure subscription. "query": we want to query just one element which is the appId , and then to make sure that we get it in a format which is a very simple string which is what we need in that variable there, we're going to say "--output tsv"
- "az role assignment create": give role assignments to this service principal, so that it can perform the necessary actions that our Github workflow needs. "--assignee": is that identity or the service principal that's going to receive the assignment. In this case, that will be our $appId. "--role": is the definition of the set of permissions that this assignee should receive.
There are three specific roles that we're going to need in the case of our Github workflow. The first one is "AcrPush", this is a role that allows whoever has this role to push docker images into Azure Container Registry. "Azure Kubernetes Service Cluster User Role" retrieve information of how to connect to the Kubernetes cluster. "Azure Kubernetes Service Contributor Role" is the one we actually deploy or create resources in the Kubernetes cluster, this allows the Github action or the service principal in this case to be able to create resources in Kubernetes cluster.

- Verify your Azure Directory by navigate to Microsoft Entra ID (or Azure Active Directory) --> App Registrations (you will see the "Github" on display name column)
- Verify your role by navigating your resource group --> Access Control (IAM) --> Role assignments. You will see three roles such as "AcrPush", "Azure Kubernetes Service Cluster User Role", "Azure Kubernetes Service Contributor Role"
```zsh
resourceGroupId=$(az group show --name $appname --query id --output tsv)
export appId="$(az ad sp create-for-rbac -n "GitHub" --query appId --output tsv)"
or
export appId="$(az ad sp create-for-rbac -n "GitHub" --scopes $resourceGroupId --role Contributor --query appId --output tsv)"

az role assignment create --assignee $appId --role "Azure Kubernetes Service Cluster User Role" --resource-group $appname
az role assignment create --assignee $appId --role "Azure Kubernetes Service Contributor Role" --resource-group $appname
```

## Deploying Seq to AKS
```powershell
$observabilityNamespace="observability"
helm repo add datalust https://helm.datalust.co
helm repo update

helm install seq datalust/seq -n $observabilityNamespace --create-namespace
kubectl get pods -n observability (verify your seq)
kubectl get services -n observability (get service)
```
- -n observability: we choose this name because everything deal with observability, monitoring, tracing, etc.
- --create-namespace: create namespace if it does not exist
- after calling "helm install": we receive the internal service of seq "seq.observability.svc.cluster.local"


## Deploy Jaeger to AKS
```powershell
helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
helm repo update

helm upgrade jaeger jaegertracing/jaeger --values ./jaeger/values.yaml -n $observabilityNamespace --install
kubectl get pods -n $observabilityNamespace (verify your command)
```
- helm repo add jaegertracing: jaegertracing is the name of the repository.
- jaegertracing/jaeger: this the repo that we need


Not deploy to AKS yet.
## Deploy Prometheus and Grafana
```powershell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade prometheus prometheus-community/kube-prometheus-stack --values ./prometheus/values.yml -n $observabilityNamespace --install
kubectl --namespace $observabilityNamespace get pods -l "release=prometheus"
```
- "kubectl ... "release=prometheus"": verify the status of the pods. When you run the command "helm upgrade ...", you will receive
this command.
- password by default when logging in Grafana is admin/promp-operator