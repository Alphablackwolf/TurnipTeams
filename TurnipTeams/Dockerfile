#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/core/aspnet:3.1-buster-slim AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM node:10.15-alpine AS client
ARG skip_client_build=false
WORKDIR /app
COPY TurnipTeams/ClientApp .
RUN [[ ${skip_client_build} = true ]] && echo "Skipping npm install" || npm install 
RUN [[ ${skip_client_build} = true ]] && mkdir -p dist || npm run-script build

FROM mcr.microsoft.com/dotnet/core/sdk:3.1-buster AS build
WORKDIR /src
COPY ["TurnipTeams/TurnipTeams.Web.csproj", "TurnipTeams/"]
RUN dotnet restore "TurnipTeams/TurnipTeams.Web.csproj"
COPY . .
WORKDIR "/src/TurnipTeams"
RUN dotnet build "TurnipTeams.Web.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "TurnipTeams.Web.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
COPY --from=client /app/dist /app/ClientApp/dist
EXPOSE 8080
ENV ASPNETCORE_URLS=http://*:8080
ENV ASPNETCORE_ENVIRONMENT=production
ENTRYPOINT ["dotnet", "TurnipTeams.Web.dll"]