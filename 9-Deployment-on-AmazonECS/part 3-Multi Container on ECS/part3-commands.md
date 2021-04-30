### deployment-04 multiple containers on ECS => production

###

- docker-compose will no more be used
- this will require some change on code ( in terms of networking )
- replace the container names defined by yaml
  - localhost if all containers are sharing the same task
  - other url if hosted outside (Atlas )

```js
  `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals?authSource=admin`,

  `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@${process.env.MONGODB_URL}:27017/course-goals?authSource=admin`,
```

"if your containers are added in the same task in AWS ECS case, then they are guaranteed to run on the same machine. Still then AWS ECS will not create a Docker network
for them instead it allows you to use localhost as an address inside of your container application code. So we can use `localhost` instead of `mongodb` here when deploying this with AWS ECS, because these two containers, which we wanna connect the node app container
and Mongo DB will be part of the same task, and therefore will be guaranteed to run on the same machine. And then AWS ECS gives us access to the network on that machine and to the other containers, which are part of the task, through the local host address."

update default environment variables

```env
MONGODB_URL=mongodb
```

```bash
$ docker build -t goals-node-backend ./backend/
$ docker tag goals-node-backend burakunuvar/goals-node-backend
#or directly tag the image with repo name
$ docker build -t burakunuvar/goals-node-backend .
$ docker push burakunuvar/goals-node-backend
```

#### Test Images on EC2

```bash
# test simple app
$ docker run --name dockerized-node-app -p 80:80 burakunuvar/node-deployment-sample-image

# create networks
$ docker network create fullstack-webapp-network
# multi container part 1 - MONGODB

$ docker run --name mongodb \
-v data:/data/db \
--rm -d \
--network fullstack-webapp-network \
-e MONGO_INITDB_ROOT_USERNAME=buraku \
-e MONGO_INITDB_ROOT_PASSWORD=password \
mongo

# multi container part 2 - NODE
docker run --name node-backend \
-e MONGODB_USERNAME=buraku \
-e MONGODB_PASSWORD=password \
-e MONGODB_URL=mongodb \
--rm -d \
--network fullstack-webapp-network \
-p 80:80 \
node-app-image

$ docker tag node-app-image burakunuvar/goals-node-backend
$ docker push burakunuvar/goals-node-backend

docker run --name node-backend \
-e MONGODB_USERNAME=buraku \
-e MONGODB_PASSWORD=password \
-e MONGODB_URL=mongodb \
--rm -d \
--network fullstack-webapp-network \
-p 80:80 \
burakunuvar/goals-node-backend

# part 3 - REACT with NGINX

$ docker run --name react-frontend \
--rm -d \
-p 3000:80 \
react-app-image

$ docker tag react-app-image burakunuvar/goals-react-frontend
$ docker push burakunuvar/goals-react-frontend

$ docker run --name react-frontend \
--rm -d \
-p 3000:80 \
burakunuvar/goals-react-frontend

```

#### ECS Container Definition for logic tier:

- `--name` : container name
- image : container image
  - for docker hub, repo name
  - for all others full url
- `-p 80:80` : publish port mapping

For Fargate, you don't need to specify two numbers here, because the container internal port will always be mapped to the same port outside of the container.

- Environment :

  - CMD within Dockerfile could be replaced =>

```yaml
CMD ["npm", "start"]
```

COMMAND : node,app.js (should be comma separated list)

- Entry Point within Dockerfile could be replaced
- Working Directory `--workdir` could be replaced
- Environment Variables : `--env` or `--env-file ./.env`

ENV MONGODB_USERNAME=buraku
ENV MONGODB_PASSWORD=password
ENV MONGODB_URL=localhost (thanks to ECS feature)

- Network Settings :
- Storage and Logging:
  - `-v` bind volumes and mappings

#### ECS Container Definition for DB Tier:

So to add MongoDB, we simply add another container to this task definition on which we are working here and give this a name of our choice, for example, MongoDB.

- `--name` : container name
- `image` : mongo (for default on dockerhub)
  - for docker hub, repo name
  - for all others full url
- port : 27017

- Environment Variables : `--env` or `--env-file ./.env`

MONGO_INITDB_ROOT_USERNAME=buraku
MONGO_INITDB_ROOT_PASSWORD=password

note: there's no dockerfile for mongo , you aren't building anything so if you don't make changes above, default ones of public image will be used for container

- Storage and Logging:
  - `-v` bind volumes and mappings

### Load Balancer

Create ALB with target group using IPs (for fargate)

- Target Group health check part should be /goals (as only this path is handled in node code)
- update SG of ALB should let port 80

### MongoDB Atlas

- update connection string in your app code as

```js
// node app running locally
  `mongodb+srv://USERNAME:PASSWORD@mongodb-cluster.qjisd.mongodb.net/goalsdb?retryWrites=true&w=majority`,

  `mongodb+srv://buraku:mbu6993bB!@mongodb-cluster.qjisd.mongodb.net/goalsdb?retryWrites=true&w=majority`

// dockerized node app
 `mongodb+srv://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@${process.env.MONGODB_URL}/${MONGODB_DBNAME}?retryWrites=true&w=majority`,
```

- update Dockerfile for new environment variable and rebuild the image `MONGODB_DBNAME`

#### EC2 to Atlas

```bash
$ docker build -t goals-node-backend .
# $ docker push burakunuvar/goals-node-backend

$ docker run --name node-backend \
-e MONGODB_USERNAME=***** \
-e MONGODB_PASSWORD=***** \
-e MONGODB_URL=***** \
-e MONGODB_DBNAME=***** \
--rm -d \
-p 80:80 \
goals-node-backend
# burakunuvar/goals-node-backend

$ docker tag goals-node-backend burakunuvar/goals-node-backend
$ docker push burakunuvar/goals-node-backend
```

# USING MULTI-STAGE BUILDS

So far we've used the development server of react test environment which could be replaced with an NGINX server in production.

- Update Dockerfile for react-frontend (replace previous as Dockerfile-dev)

```Dockerfile
FROM node:alpine as builder
...
FROM nginx
COPY --from=builder /home/node/app/build /usr/share/nginx/html
```

- If you'll run react-frontend and node-backend on the same task, update the code

```js
//const response = await fetch("http://54.170.114.35/goals");
const response = await fetch("/goals");
```

- But it might not be possible to publish port 80 twice ( for node and react ) thus best practice is to have two different tasks and using environment variables.

```js
const backendUrl =
  process.env.NODE_ENV === "development"
    ? "http://localhost"
    : "http://containerized-fullstack-alb-1716459932.eu-west-1.elb.amazonaws.com";
```

=> NODE_ENV is "development" if we use react start script
=> NODE_ENV is "production" if we use react build script

```bash
# part 3 - REACT with NGINX
# DEVELOPMENT
$ docker build -t react-app-image -f Dockerfile-dev .

$ docker run --name react-frontend \
--rm -d \
-p 3000:3000 \
-it \
react-app-image

# PRODUCTION
$ docker build -t react-app-image .
$ docker tag react-app-image burakunuvar/goals-react-frontend
$ docker push burakunuvar/goals-react-frontend

$ docker run --name react-frontend \
--rm -d \
-p 3000:80 \
burakunuvar/goals-react-frontend

```
