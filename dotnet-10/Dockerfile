#----------------- DEVELOPMENT ------------------------
FROM mcr.microsoft.com/devcontainers/dotnet:10.0 AS development
WORKDIR /app
CMD ["sleep", "infinity"]

#----------------- BUILD ------------------------------
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:10.0 AS build
ARG TARGETARCH
WORKDIR /src

COPY . .
RUN dotnet restore -a $TARGETARCH "src/App/App.csproj"

RUN dotnet publish -a $TARGETARCH "src/App/App.csproj" \
    --no-restore \
    -o /app

#----------------- FINAL (PRODUCTION) -----------------
FROM mcr.microsoft.com/dotnet/aspnet:10.0-noble-chiseled AS final
EXPOSE 8080
WORKDIR /app
COPY --link --from=build /app .
USER $APP_UID
ENTRYPOINT ["./App"]