# ===================== BASE =====================
FROM node:20-slim AS base
WORKDIR /app/src/frontend

# ===================== DEPENDENCIES =====================
FROM base AS deps
COPY src/frontend/package*.json ./
RUN npm ci

# ===================== BUILDER =====================
FROM base AS builder

RUN apt-get update && apt-get install -y \
    curl \
    unzip \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# install protoc
RUN curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v25.3/protoc-25.3-linux-x86_64.zip && \
    unzip protoc-25.3-linux-x86_64.zip -d /usr/local

ENV PATH="/usr/local/bin:$PATH"

WORKDIR /app

# copy full repo
COPY pb ./pb
COPY src/frontend ./src/frontend

WORKDIR /app/src/frontend

# install deps AGAIN (important for deterministic builds)
COPY src/frontend/package*.json ./
RUN npm ci

# ===================== GRPC GENERATION =====================
RUN mkdir -p ./protos && \
    protoc -I /app/pb \
      --plugin=./node_modules/.bin/protoc-gen-ts_proto \
      --ts_proto_out=./protos \
      --ts_proto_opt=esModuleInterop=true,outputServices=grpc-js \
      /app/pb/demo.proto

# ===================== NEXT BUILD =====================
RUN npm run build


# ===================== RUNTIME =====================
FROM gcr.io/distroless/nodejs20-debian12 AS runner

WORKDIR /app

ENV NODE_ENV=production
ENV PORT=8080

COPY --from=builder /app/src/frontend/.next/standalone ./
COPY --from=builder /app/src/frontend/.next/static ./.next/static
COPY --from=builder /app/src/frontend/public ./public

EXPOSE 8080

CMD ["server.js"]