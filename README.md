# Multi-Container Application
This project is part of [roadmap.sh](https://roadmap.sh/projects/multi-container-service) DevOps projects.


## Test the Project Locally

### Endpoints

- `GET /todos` — Get all todos
- `POST /todos` — Create a new todo
- `GET /todos/:id` — Get a todo by ID
- `PUT /todos/:id` — Update a todo by ID
- `DELETE /todos/:id` — Delete a todo by ID

### Setup
1. Install dependencies:

```bash
npm install
```

2. Set up a local mongodb database
I've used docker image.

```bash
docker run -d -p 27017:27017 mongo
```
3. Change this line in the index.js
```
mongoose.connect('mongodb://mongo:27017/todos', {
mongoose.connect('mongodb://localhost:27017/todos', {
```

4. Start the server:
```bash
npm start
```
Go to [localhost:3000/todos](http://localhost:3000/todos)

5. Add some data to database

```bash
curl -X POST http://localhost:3000/todos \
     -H "Content-Type: application/json" \
     -d '{"title": "Buy milk"}'
```

## Dockerize the Project

1. Create a dockerfile:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD [ "npm", "start" ]
```
2. Create a docker compose file:
```compose.yaml
version: '3'
services:
  node-service:
    build: .
    container_name: node-service
    restart: always
    ports:
      - 3000:3000
    networks:
      - node-network

  mongo:
    image: mongo:latest 
    container_name: mongo
    restart: always
    volumes:
      - ./data:/data/db
    networks:
      - node-network

networks:
  node-network:
    driver: bridge
```

3. Create a .dockerignore file:

With this configuration you will save mongodb files to ./data but we don't want to put this file into node-service docker image. So create a .dockerignore file:
```
data/
node_modules/
compose.yaml
.gitignore
```

4. Change the source code to use docker dns:
In index.js file change this line to this:
```
mongoose.connect('mongodb://localhost:27017/todos', {
mongoose.connect('mongodb://mongo:27017/todos', {
```

## Setup a reverse proxy
1. Choose the right software for reverse proxy
I recommend nginx.

2. Configure your compose.yaml:
- Add this lines to your compose.yaml:

```compose.yaml
  nginx-proxy:
    image: nginx:latest
    container_name: nginx-proxy
    restart: always
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
       - "80:80"
       - "443:443"
    networks: 
      - node-network
```
- Delete the `node-service` open ports from compose.yaml
- Create a nginx directory
- Inside this directory create a nginx.conf file:
```conf
worker_processes auto;

events {
	worker_connections 1024;
}

http {
	server {
		listen 80;
		location / {
			proxy_pass http://node-service:3000;
		}
	}
}

```
Don't forget this is not production ready configuration.

## Setup a remote Linux Server
Setup your linux server and add your SSH keys into your server.

## Setup a CI/CD pipeline
1. You need to manage Github Secrets first:
 - `REMOTE_HOST`:IP of the server
 - `REMOTE_USER`: User of the server
 - `SSH_KEY`: Your private SSH key
 - `SSH_PORT`: Your SSH port 
Create a github workflow:
```yaml
name: Deploy Service

on:
  push:
    branches:
      - main  # Trigger workflow on push to the main branch
  pull_request:
    branches:
      - main  # Trigger on pull request
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Execute remote SSH commands using password
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.REMOTE_HOST }}
          username: ${{ secrets.REMOTE_USER }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script: cd ~/multi-container-service && git pull && docker compose down && docker compose up -d --build 
```
