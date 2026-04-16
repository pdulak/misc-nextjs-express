FROM node:20-alpine AS base
RUN apk add --no-cache python3 make g++
RUN npm install -g pnpm
WORKDIR /app

FROM base AS deps
COPY pnpm-lock.yaml pnpm-workspace.yaml package.json ./
COPY packages/backend/package.json ./packages/backend/package.json
COPY packages/frontend/package.json ./packages/frontend/package.json
RUN pnpm install --frozen-lockfile

FROM deps AS development
COPY packages/backend ./packages/backend
EXPOSE 3001
CMD ["pnpm", "--filter", "backend", "dev"]

FROM deps AS production
COPY packages/backend ./packages/backend
RUN pnpm --filter backend build
EXPOSE 3001
CMD ["node", "packages/backend/dist/index.js"]
