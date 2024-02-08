I have a strange situation:

I am tryning to docker-compose my app

ui : a svelte node app
api : a node-express app
mongodb : a mongodb container
nginx : nginx container

The mission is that the svelte app and the api both should be accessable from out side since when the svelte app makes an api call to the api (FROM THE USER BROWSER) it is considered an outside connection and thus we need to give api some outside access 

The problem : when docker-compose is run the api is not accessable

Here is the nginx defaut.config file
server {
    listen 80;
    location / {
        proxy_pass http://ui:3000;
    }
    location /api {
        proxy_pass http://api:5000;
    }
}

Here is dockerfile for api ui
FROM node:21.6-alpine3.18 AS build
WORKDIR /app
COPY package.json .
COPY package-lock.json ./
RUN npm install 
COPY . ./
RUN npm run build 
EXPOSE 3000
CMD ["node", "./build/index.js"]

Here is dockerfile for api app

version: '3'
services:

  db:
    image: mongo:latest
    volumes:
      - mongodbdata:/data/db
    ports:
      - 27017:27017
    
  api:
    build: ./
    depends_on:
      - db
    ports:
      - 5000:5000
    environment:
      JWT_SECRET: "frd6Sb14FDu810XzAJwQ09KcMwwIc39NDFGsdyt8"
      JWT_REFRESH_TOKEN: "S4FDu810GsdytXzAJwQ09Kb1cMwwIc39NDF8frd6"
      MONGO_DB_URL: "mongodb://db:27017/taleem_db"

volumes:
  mongodbdata:

here is docker-compose file
version: '3'
services:

  db:
    image: mongo:latest
    volumes:
      - mongodbdata:/data/db
    ports:
      - 27017:27017
    
  api:
    build: ./api
    ports:
      - 5000:5000
    depends_on:
      - db
    environment:
      JWT_SECRET: "frd6Sb14FDu810XzAJwQ09KcMwwIc39NDFGsdyt8"
      JWT_REFRESH_TOKEN: "S4FDu810GsdytXzAJwQ09Kb1cMwwIc39NDF8frd6"
      MONGO_DB_URL: "mongodb://db:27017/taleem_db"
      

  ui:
    build: ./ui
    depends_on:
      - api

  nginx:
    image: nginx:latest
    ports:
      - 80:80
    volumes:
      - ./nginx:/etc/nginx/conf.d
    depends_on:
      - ui
      - api

volumes:
  mongodbdata:

 --> i must add the api container is running but is not accessable