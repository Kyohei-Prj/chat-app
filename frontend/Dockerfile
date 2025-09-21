# ---------- Build Stage ----------
FROM node:22.19.0-alpine3.21 AS build
WORKDIR /app

COPY package*.json ./
RUN npm ci --silent --omit=dev

COPY . .

RUN npm run build


# ---------- Production Stage ----------
FROM nginx:stable-alpine

# Copy our custom site config and static build output
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html

# set non-root user
RUN touch /var/run/nginx.pid
RUN chown -R nginx:nginx /var/run/nginx.pid /usr/share/nginx/html /var/cache/nginx /var/log/nginx /etc/nginx/conf.d
USER nginx

EXPOSE 8081
CMD ["nginx", "-g", "daemon off;"]
