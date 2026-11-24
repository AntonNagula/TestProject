# Build context: repository root
#   docker build -f AwesomeApi/Dockerfile -t awesomeapi .

FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
ARG BUILD_CONFIGURATION=Release
WORKDIR /src

# Restore first so this layer is cached until project files change
COPY TestProject.slnx ./
COPY AwesomeApi/AwesomeApi.csproj AwesomeApi/
RUN dotnet restore AwesomeApi/AwesomeApi.csproj

COPY . .
RUN dotnet publish AwesomeApi/AwesomeApi.csproj \
    --configuration $BUILD_CONFIGURATION \
    --no-restore \
    --output /app/publish \
    /p:UseAppHost=false

FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS final
WORKDIR /app
EXPOSE 8080
USER $APP_UID
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "AwesomeApi.dll"]
