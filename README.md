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