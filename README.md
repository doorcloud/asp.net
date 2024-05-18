# Pr�requis
Avoir une application ASP.NET Core construite
Avoir Docker install� sur votre machine locale https://www.docker.com/get-started/
Avoir trivy install� pour scanner l'image construite avant sa publication https://aquasecurity.github.io/trivy/v0.18.3/installation/
Connaissances de base de Docker et des images Docker

# �tapes

### Cr�er un fichier Dockerfile

Le fichier Dockerfile d�finit les instructions pour construire l'image Docker pour votre application. Un fichier Dockerfile basique pour une application ASP.NET Core pourrait ressembler � ceci:

```
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
...
...
RUN dotnet publish "./WebApp.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=true

FROM mcr.microsoft.com/dotnet/aspnet:8.0
...
ENTRYPOINT ["./WebApp"]
```

Ce fichier Dockerfile utilise l'image `mcr.microsoft.com/dotnet/sdk:8.0` comme image de base (pour la construction de l'artefact) . Il d�finit ensuite le r�pertoire de travail sur `/app`, copie les fichiers de projet ASP.NET Core dans le conteneur, restaure les d�pendances NuGet, construit l'application . Dans un second temps , il copie le dossier de l'artefact vers une autre image de base `mcr.microsoft.com/dotnet/aspnet:8.0` (pour l'execution de l'artefact) plus precisement vers un nouvel espace de travail `/app`  et d�finit le point d'entr�e de l'application sur `./WebApp`.

### Construire l'image Docker
Une fois que vous avez cr�� votre fichier Dockerfile, vous pouvez construire l'image Docker pour votre application en ex�cutant la commande suivante dans le r�pertoire de votre projet :

```
$ docker build -t webapp:v1.0.0 .
```

Cette commande va construire l'image Docker et le tag `webapp:v1.0.0`

### Ex�cuter l'application conteneuris�e
Pour ex�cuter l'application conteneuris�e, vous pouvez utiliser la commande suivante:

```
$ docker run -p 8000:8080 webapp:v1.0.0 
```

Cette commande va ex�cuter l'image Docker et mapper le port 8000 du conteneur au port 8080 de l'h�te (-p 8000:8080). Vous devriez maintenant pouvoir acc�der � votre application � l'adresse http://localhost:8000.

### Scanner et Publier votre image vers un depot d'image dockerhub
Avant de publier l'image, il faut la renommer avec les reference de la registry 
```
$ docker tag webapp:v1.0.0 <userrname|organization>/webapp:v1.0.0
```

Ensuite la scanner pour evaluer les vulnerabilites de l'image

```
$ trivy image --severity CRITICAL,HIGH <userrname|organization>/webapp:v1.0.0
```
Et publier 

```
$ docker push <userrname|organization>/webapp:v1.0.0
```
