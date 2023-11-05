#Dockerfile based from Microsoft's example.

# Build app
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS scratch

WORKDIR /App

COPY . ./

#Install dependencies
RUN dotnet restore

RUN dotnet publish -c release --self-contained false --no-restore

# Production
FROM mcr.microsoft.com/dotnet/aspnet:7.0

WORKDIR /App

COPY --from=scratch /App/bin/release/net7.0/ .

ENTRYPOINT ["dotnet", "Worker.dll"]
