# syntax=docker/dockerfile:1

FROM node:20-alpine AS base
WORKDIR /app

FROM base AS builder

RUN apk add --no-cache git
ARG REPO_URL
ARG GIT_BRANCH
RUN git clone --branch ${GIT_BRANCH} --depth 1 ${REPO_URL} .

RUN --mount=type=cache,target=/root/.npm \
    npm ci
# Do not use --omit=dev here: it prunes devDependencies required for `npm run build`.
RUN --mount=type=cache,target=/root/.npm \
    npm install sharp

ENV NEXT_TELEMETRY_DISABLED=1
RUN --mount=type=cache,target=/app/.next/cache \
    npm run build

FROM base AS runner

ENV NODE_ENV=production
ENV PORT=3000
ENV HOSTNAME=0.0.0.0
ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup --system --gid 1001 nodejs \
    && adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
COPY --from=builder --chown=nextjs:nodejs /app/node_modules/sharp ./node_modules/sharp
COPY --from=builder --chown=nextjs:nodejs /app/node_modules/@img ./node_modules/@img

USER nextjs

EXPOSE 3000

CMD ["node", "server.js"]
