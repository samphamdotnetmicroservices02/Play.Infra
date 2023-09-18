# Play.Infra
Play Economy Infrastructure components

## Add the Github package source
```powershell
$owner="samphamdotnetmicroservices02"
$gh_pat="[PAT HERE]"

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"

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
```

```mac
owner="samphamdotnetmicroservices02"
gh_pat="[PAT HERE]"

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

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

```mac
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

```mac
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

```mac
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

```mac
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

```mac
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

after that, go to Azure Cloud to set value of secret based on each microservice like MongoDbSettings__ConnectionString, 
ServiceBusSetting__ConnectionString and change underscores "__" to double dash "--"
```