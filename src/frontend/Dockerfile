# =========================
# 1. Base image
# =========================
FROM node:20-alpine AS base

WORKDIR /app

# install system deps
RUN apk add --no-cache libc6-compat python3 make g++

# =========================
# 2. Dependencies stage
# =========================
FROM base AS deps

WORKDIR /app

# copy only package files first (better caching)
COPY ./src/frontend/package*.json ./

# 🚨 CRITICAL: disable Cypress download during build
ENV CYPRESS_INSTALL_BINARY=0
ENV npm_config_loglevel=verbose

# stabilize npm for bad networks
RUN npm config set registry https://registry.npmjs.org/ && \
    npm config set fetch-retries 10 && \
    npm config set fetch-retry-mintimeout 20000 && \
    npm config set fetch-retry-maxtimeout 300000 && \
    npm config set fetch-timeout 600000 && \
    npm config set maxsockets 1 && \
    npm config set strict-ssl true

# install dependencies
RUN npm ci --no-audit --no-fund

# =========================
# 3. Builder stage
# =========================
FROM base AS builder

WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY ./src/frontend .

# build app
RUN npm run build

# =========================
# 4. Runner stage
# =========================
FROM node:20-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production

# create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

# copy build output
COPY --from=builder /app ./

USER nextjs

EXPOSE 3000

CMD ["npm", "start"]