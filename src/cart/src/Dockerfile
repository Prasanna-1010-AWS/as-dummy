# Copyright The OpenTelemetry Authors
# SPDX-License-Identifier: Apache-2.0

FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:8.0 AS build

ARG TARGETARCH

WORKDIR /src

COPY ./src/cart/ ./cart/
COPY ./pb/ ./pb/

WORKDIR /src/cart

RUN dotnet restore ./src/cart.csproj \
    -r linux-$TARGETARCH

RUN dotnet publish ./src/cart.csproj \
    -c Release \
    -r linux-$TARGETARCH \
    --no-restore \
    --self-contained true \
    -p:PublishSingleFile=true \
    -p:PublishTrimmed=false \
    -p:PublishReadyToRun=true \
    -p:EnableCompressionInSingleFile=true \
    -o /app

FROM gcr.io/distroless/base-debian12:nonroot

WORKDIR /app

COPY --from=build --chown=nonroot:nonroot /app/ ./

ENV ASPNETCORE_URLS=http://+:8080
ENV CART_PORT=8080
ENV DOTNET_HOSTBUILDER__RELOADCONFIGONCHANGE=false

EXPOSE 8080

USER nonroot:nonroot

ENTRYPOINT ["/app/cart"]
