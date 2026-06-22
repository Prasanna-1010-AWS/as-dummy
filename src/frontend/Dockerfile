# ===================== DEPENDENCIES =====================
FROM node:20-alpine AS deps

RUN apk add --no-cache libc6-compat

WORKDIR /app

COPY ./src/frontend/package*.json ./

# clean install (important for CI consistency)
RUN npm ci


# ===================== BUILDER =====================
FROM node:20-alpine AS builder

RUN apk add --no-cache libc6-compat curl unzip

WORKDIR /app

# install protobuf manually (avoids apk issues)
RUN curl -LO https://github.com/protocolbuffers/protobuf/releases/download/v25.3/protoc-25.3-linux-x86_64.zip && \
    unzip protoc-25.3-linux-x86_64.zip -d /usr/local

COPY --from=deps /app/node_modules ./node_modules

COPY ./pb ./pb
COPY ./src/frontend ./

# ensure protoc is available
ENV PATH="/usr/local/bin:$PATH"

RUN npm run grpc:generate
RUN npm run build


# ===================== RUNTIME =====================
FROM node:20-alpine AS runner

WORKDIR /app

RUN apk add --no-cache libc6-compat

ENV NODE_ENV=production
ENV PORT=8080

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

# copy standalone build
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static
COPY --from=builder /app/public ./public

USER nextjs

EXPOSE 8080

CMD ["node", "server.js"]