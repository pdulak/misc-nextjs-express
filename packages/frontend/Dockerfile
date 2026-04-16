FROM node:20-alpine AS base
RUN apk add --no-cache python3 make g++
RUN npm install -g pnpm
WORKDIR /app

FROM base AS deps
COPY pnpm-lock.yaml pnpm-workspace.yaml package.json ./
COPY packages/backend/package.json ./packages/backend/package.json
COPY packages/frontend/package.json ./packages/frontend/package.json
RUN pnpm install --frozen-lockfile
COPY packages/frontend ./packages/frontend

FROM deps AS development
EXPOSE 3000
CMD ["pnpm", "--filter", "frontend", "dev"]

FROM deps AS production
ARG API_URL=http://localhost:3001
ENV API_URL=$API_URL
RUN pnpm --filter frontend build
EXPOSE 3000
CMD ["pnpm", "--filter", "frontend", "start"]
