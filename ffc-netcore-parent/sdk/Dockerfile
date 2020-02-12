ARG NETCORE_VERSION=3.1
FROM mcr.microsoft.com/dotnet/core/sdk:${NETCORE_VERSION}-alpine
WORKDIR /app
RUN adduser --disabled-password www-data
RUN chown -R www-data:www-data /app
USER www-data
ENTRYPOINT [ "dotnet", "watch", "run"]

