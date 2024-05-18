# Prérequis
Avoir une application ASP.NET Core construite
Avoir Docker installé sur votre machine locale https://www.docker.com/get-started/
Avoir trivy installé pour scanner l'image construite avant sa publication https://aquasecurity.github.io/trivy/v0.18.3/installation/
Connaissances de base de Docker et des images Docker

# Étapes

### Créer un fichier Dockerfile

Le fichier Dockerfile définit les instructions pour construire l'image Docker pour votre application. Un fichier Dockerfile basique pour une application ASP.NET Core pourrait ressembler à ceci:

```
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
...
...
RUN dotnet publish "./WebApp.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=true

FROM mcr.microsoft.com/dotnet/aspnet:8.0
...
ENTRYPOINT ["./WebApp"]
```

Ce fichier Dockerfile utilise l'image `mcr.microsoft.com/dotnet/sdk:8.0` comme image de base (pour la construction de l'artefact) . Il définit ensuite le répertoire de travail sur `/app`, copie les fichiers de projet ASP.NET Core dans le conteneur, restaure les dépendances NuGet, construit l'application . Dans un second temps , il copie le dossier de l'artefact vers une autre image de base `mcr.microsoft.com/dotnet/aspnet:8.0` (pour l'execution de l'artefact) plus precisement vers un nouvel espace de travail `/app`  et définit le point d'entrée de l'application sur `./WebApp`.

### Construire l'image Docker
Une fois que vous avez créé votre fichier Dockerfile, vous pouvez construire l'image Docker pour votre application en exécutant la commande suivante dans le répertoire de votre projet :

```
$ docker build -t webapp:v1.0.0 .
```

Cette commande va construire l'image Docker et le tag `webapp:v1.0.0`

### Exécuter l'application conteneurisée
Pour exécuter l'application conteneurisée, vous pouvez utiliser la commande suivante:

```
$ docker run -p 8000:8080 webapp:v1.0.0 
```

Cette commande va exécuter l'image Docker et mapper le port 8000 du conteneur au port 8080 de l'hôte (-p 8000:8080). Vous devriez maintenant pouvoir accéder à votre application à l'adresse http://localhost:8000.

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
