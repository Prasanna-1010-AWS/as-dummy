# ===================== DEPENDENCIES =====================
FROM node:20-alpine AS deps

WORKDIR /app/src/frontend

COPY src/frontend/package*.json ./
RUN npm ci


# ===================== BUILDER =====================
FROM node:20-alpine AS builder

RUN apk add --no-cache curl unzip libc6-compat

WORKDIR /app

# install protoc
RUN curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v25.3/protoc-25.3-linux-x86_64.zip && \
    unzip protoc-25.3-linux-x86_64.zip -d /usr/local

ENV PATH="/usr/local/bin:$PATH"

# copy dependencies
COPY --from=deps /app/src/frontend/node_modules ./src/frontend/node_modules

# copy full source (IMPORTANT: root context)
COPY pb ./pb
COPY src/frontend ./src/frontend

WORKDIR /app/src/frontend

# ===================== GRPC GENERATION =====================
RUN mkdir -p ./protos && \
    protoc -I /app/pb \
      --plugin=./node_modules/.bin/protoc-gen-ts_proto \
      --ts_proto_out=./protos \
      --ts_proto_opt=esModuleInterop=true,outputServices=grpc-js \
      /app/pb/demo.proto


# ===================== NEXT.JS BUILD =====================
RUN npm run build


# ===================== RUNTIME =====================
FROM gcr.io/distroless/nodejs20-debian12 AS runner

WORKDIR /app

ENV NODE_ENV=production
ENV PORT=8080

# Next.js standalone output
COPY --from=builder /app/src/frontend/.next/standalone ./
COPY --from=builder /app/src/frontend/.next/static ./.next/static
COPY --from=builder /app/src/frontend/public ./public

EXPOSE 8080

CMD ["server.js"]