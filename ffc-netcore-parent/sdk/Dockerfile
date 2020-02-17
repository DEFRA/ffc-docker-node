ARG NETCORE_VERSION=3.1
FROM mcr.microsoft.com/dotnet/core/sdk:${NETCORE_VERSION}-alpine
ENV ASPNETCORE_ENVIRONMENT=development
WORKDIR /app
RUN apk update \
  && apk add unzip \
  && apk --no-cache add curl procps \
  && wget -qO- https://aka.ms/getvsdbgsh | /bin/sh /dev/stdin -v latest -l /vsdbg
RUN adduser --disabled-password www-data
RUN chown -R www-data:www-data /app
USER www-data
ENTRYPOINT [ "dotnet", "watch", "run"]

