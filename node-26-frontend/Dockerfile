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

#----------------- FINAL (PRODUCTION) -----------------
FROM nginxinc/nginx-unprivileged:stable-alpine AS final
EXPOSE 8080

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /src/dist /usr/share/nginx/html

CMD ["nginx", "-g", "daemon off;"]
