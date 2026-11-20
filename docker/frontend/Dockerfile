# Build context: repository root (needs ./frontend and ./docker/frontend)

# ---- Build stage: Angular production build ----
FROM node:18-alpine AS build
WORKDIR /src
COPY frontend/package.json frontend/package-lock.json ./
RUN npm ci --no-audit --no-fund
COPY frontend/ .
# The upstream services call http://localhost:8084/skillExchange/... directly.
# Rewrite those to relative URLs so the same image works behind any host:
# NGINX below proxies /skillExchange/ to the backend.
# The upstream component styles exceed Angular's default size budgets, which
# fail the production build: budgets are dropped here (build-time only).
RUN grep -rl "http://localhost:8084" src | xargs sed -i "s#http://localhost:8084##g" \
 && node -e "const f='angular.json',a=require('./'+f);for(const p of Object.values(a.projects))delete p.architect.build.configurations.production.budgets;require('fs').writeFileSync(f,JSON.stringify(a,null,2))" \
 && npx ng build --configuration production

# ---- Runtime stage: unprivileged NGINX ----
FROM nginxinc/nginx-unprivileged:1.27-alpine
COPY docker/frontend/nginx.conf.template /etc/nginx/templates/default.conf.template
COPY --from=build /src/dist/skill-xchange /usr/share/nginx/html
ENV BACKEND_URL=http://backend:8084
EXPOSE 8080
