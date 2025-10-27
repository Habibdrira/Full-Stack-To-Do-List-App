# Étape 1 : build Angular
FROM node:18-alpine as build

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build --prod

# Étape 2 : servir avec NGINX
FROM nginx:alpine
COPY --from=build /app/dist/to-do-list-app-angular /usr/share/nginx/html
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
