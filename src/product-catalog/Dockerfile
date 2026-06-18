FROM golang:1.22-alpine AS builder

WORKDIR /usr/src/app

# Use Go build cache for dependencies
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    mkdir -p /root/.cache/go-build

# Copy only go.mod first (safe)
COPY go.mod go.sum ./

RUN go mod download

# Copy full source (IMPORTANT FIX)
COPY . .

RUN go build -o product-catalog .

####################################

FROM alpine AS release

WORKDIR /usr/src/app/

# copy products from builder stage (IMPORTANT FIX)
COPY --from=builder /usr/src/app/products ./products

# copy binary correctly (IMPORTANT FIX)
COPY --from=builder /usr/src/app/product-catalog ./product-catalog

ENV PRODUCT_CATALOG_PORT=8088

ENTRYPOINT ["./product-catalog"]
