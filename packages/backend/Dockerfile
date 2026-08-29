FROM node:24-alpine AS base
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

FROM deps AS build
COPY packages/backend ./packages/backend
RUN pnpm --filter backend build
RUN pnpm --filter backend deploy --prod /app/deployed

FROM node:24-alpine AS production
WORKDIR /app
COPY --from=build /app/deployed ./
EXPOSE 3001
CMD ["node", "dist/index.js"]