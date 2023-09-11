# Play.Infra
Play Economy Infrastructure components

## Add the Github package source
```powershell
$owner="samphamdotnetmicroservices02"
$gh_pat="[PAT HERE]"

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

```mac
owner="samphamdotnetmicroservices02"
gh_pat="[PAT HERE]"

dotnet nuget add source --username USERNAME --password $gh_pat --store-password-in-clear-text --name github "https://nuget.pkg.github.com/$owner/index.json"
```

```
--username: the username that you're going to use to connect to GitHub packages. Now the interesting thing here
is that when you use personal access tokens, it doesn't really matter what username you use here, because the
access tokens has all the information needed to authenticate you. So you can use anything here, I'll just put
username

--password: the password is going to be our brand new GitHub personal access token

--store-password-in-clear-text: This's a requirement from NuGet in terms of how to store this password in the local box.

--name: The name is going to be the name of the source. Let's name it just GitHub

lastly, you have to specify the address of this new source. So the address for GitHub packages has to be 
https://nuget.pkg.github.com/

after add to nuget, use dotnet nuget list source to check the result
```