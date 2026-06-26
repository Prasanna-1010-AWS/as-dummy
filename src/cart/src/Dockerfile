# -----------------------------------------------------------------------------
# Build Stage
# -----------------------------------------------------------------------------
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:8.0 AS builder

ARG TARGETARCH

WORKDIR /usr/src/app

COPY ./src/cart/ ./
COPY ./pb/ ./pb/

RUN dotnet restore ./src/cart.csproj -r linux-musl-$TARGETARCH

RUN dotnet publish ./src/cart.csproj \
    -c Release \
    -r linux-musl-$TARGETARCH \
    --no-restore \
    -o /cart

# -----------------------------------------------------------------------------
# Runtime Stage (Distroless)
# -----------------------------------------------------------------------------
FROM mcr.microsoft.com/dotnet/runtime-deps:8.0-alpine3.20-distroless

WORKDIR /usr/src/app

COPY --from=builder /cart/ ./

ENV DOTNET_HOSTBUILDER__RELOADCONFIGONCHANGE=false

EXPOSE ${CART_PORT}

ENTRYPOINT ["./cart"]