# Dockerfile basé sur l'exemple de Microsoft

# Base de l'image de construction sur le SDK .NET officiel
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS scratch

WORKDIR /App

COPY . ./

# Restaure les dépendances spécifiées dans les fichiers projet .NET
RUN dotnet restore

# Publie l'application .NET pour la configuration 'Release'
# La publication est sans dépendances externes (--self-contained false) et ne restaure pas les dépendances (--no-restore)
RUN dotnet publish -c release --self-contained false --no-restore

# Étape de production, utilisation de l'image ASP.NET runtime comme base
FROM mcr.microsoft.com/dotnet/aspnet:7.0

WORKDIR /App

# Copie les artefacts de construction depuis l'étape de build précédente dans le répertoire de travail actuel
COPY --from=scratch /App/bin/release/net7.0/ .

ENTRYPOINT ["dotnet", "Worker.dll"]