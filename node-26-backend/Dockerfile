#----------------- DEVELOPMENT ------------------------
FROM mcr.microsoft.com/devcontainers/typescript-node:26 AS development
RUN npm install -g pnpm
WORKDIR /app
CMD ["sleep", "infinity"]

#----------------- BUILD ------------------------------
FROM node:26-slim AS build
RUN npm install -g pnpm
WORKDIR /src

COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm run build
RUN pnpm prune --prod

#----------------- FINAL (PRODUCTION) -----------------
FROM node:26-slim AS final
ENV NODE_ENV=production
EXPOSE 3000
WORKDIR /app

COPY --chown=node:node --from=build /src/package.json ./
COPY --chown=node:node --from=build /src/node_modules ./node_modules
COPY --chown=node:node --from=build /src/dist ./dist

USER node
CMD ["node", "dist/main.js"]
