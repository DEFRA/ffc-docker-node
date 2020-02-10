ARG NETCORE_VERSION=3.1
ARG FILE
ARG WORKING_DIR
FROM mcr.microsoft.com/dotnet/core/sdk:${NETCORE_VERSION}-alpine AS base

# Base
WORKDIR /app
ENV ASPNETCORE_ENVIRONMENT=production

# Development
FROM base AS development
WORKDIR /${WORKING_DIR}
RUN apt-get update \
  && apt-get install -y --no-install-recommends unzip \
  && curl -sSL https://aka.ms/getvsdbgsh | bash /dev/stdin -v latest -l /vsdbg
COPY ./${WORKING_DIR}/*.csproj ./
RUN dotnet restore
COPY ./${NETCORE_VERSION} ./
ENTRYPOINT [ "dotnet", "watch", "run", "--read-only", "COMPlus_EnableDiagnostics=0" "--urls", "http://0.0.0.0:3007" ]

# Test
FROM development AS test 
WORKDIR /${WORKING_DIR}.Tests
COPY ./${WORKING_DIR}.Tests/*.csproj ./
RUN dotnet restore
COPY ./${WORKING_DIR}.Tests ./
ENTRYPOINT [ "dotnet", "test" ]

# Production
FROM base AS production
COPY ./${WORKING_DIR}/*.csproj ./
RUN dotnet restore
COPY ./${WORKING_DIR} ./
RUN dotnet publish -c Release -o out

# Runtime
FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS runtime
WORKDIR /app
COPY --from=production /app/out ./
RUN chown -R www-data:www-data /app
ENV ASPNETCORE_URLS=http://*:3007
ENV DLL-FILE ${WORKING_DIR}.dll
EXPOSE 3007
ENTRYPOINT [ "dotnet", "${DLL-FILE}" ]
