# Copyright The OpenTelemetry Authors
# SPDX-License-Identifier: Apache-2.0


FROM node:20-alpine AS deps
RUN apk add --no-cache libc6-compat

WORKDIR /app/src/frontend

COPY ./src/frontend/package*.json ./
RUN npm ci

FROM node:20-alpine AS builder
RUN apk add --no-cache libc6-compat protobuf-dev protoc
WORKDIR /app/src/frontend
COPY --from=deps /app/src/frontend/node_modules ./node_modules
COPY ./pb /app/pb
COPY ./src/frontend ./

RUN npm run grpc:generate
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
RUN apk add --no-cache protobuf-dev protoc

ENV NODE_ENV=production

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

COPY --from=builder /app/src/frontend/next.config.js ./
COPY --from=builder /app/src/frontend/utils/telemetry/Instrumentation.js ./
COPY --from=builder /app/src/frontend/public ./public
COPY --from=builder /app/src/frontend/package.json ./package.json
COPY --from=deps /app/src/frontend/node_modules ./node_modules

COPY --from=builder --chown=nextjs:nodejs /app/src/frontend/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/src/frontend/.next/static ./.next/static

USER nextjs

ENV PORT=8080
EXPOSE ${PORT}

CMD ["node", "--require", "./Instrumentation.js", "server.js"]

