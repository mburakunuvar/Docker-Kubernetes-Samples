# PART1- GETTING STARTED

## Dockerizing the MongoDB Service

### step1

https://hub.docker.com/_/mongo

```bash
# connect nodeapp running on locally to containerized mongodb
$ docker run --name mongodb --rm -d -p 27017:27017 mongo
```

with this `mongodb://localhost:27017/course-goals` will be mapped to our container. So a node app running locally can access to containerized mongodb

## Dockerizing the Node App

### step 1

A node app running locally can access to containerized mongodb with this :

- `'mongodb://localhost:27017/course-goals',`

```bash
$ docker build -t node-app-image .
$ docker run --name node-backend --rm -d -p 80:80 node-app-image

```

But for a containerized node app to access containerized mongodb:

- `'mongodb://host.docker.internal:27017/course-goals',`

## Dockerizing the React App

## step 1

First enable a react app running locally connect to containerized node app :

```bash
$ docker run --name node-backend --rm -d -p 80:80 node-app-image
```

Second, containerize react app

```bash
$ docker build -t react-app-image .
$ docker run --name goals-frontend --rm -d -p 3000:3000 react-app-image
## REACT Apps will require -it flag
$ docker run --name goals-frontend --rm -d -p 3000:3000 -it react-app-image
```

We're done, but so far all communication is possible through localhost machine by publishing ports, there's no cross container communication yet

# PART2- ADDING DOCKER NETWORKS

```bash
$ docker network ls
$ docker network create web-app-network
```

## Dockerizing the MongoDB Service

### step2

use the docker networking this we'll no more need to publish the port

```bash
$ docker run --name mongodb --rm -d --network web-app-network mongo
```

## Dockerizing the Node App

### step 2

```bash
$ docker run --name node-backend  --network web-app-network --rm -d node-app-image
```

instead of going through localhost to access containerized mongodb which required port mapping to be published on step1 :

- `'mongodb://host.docker.internal:27017/course-goals',`

we could use docker networks now , which will simply require container name

- `'mongodb://mongodb:27017/course-goals',`

```bash
$ docker build -t node-app-image .
$ docker run --name node-backend  --network web-app-network --rm -d node-app-image
```

## Dockerizing the React App

### part 2

```bash
$ docker run --name goals-frontend --rm -d -p 3000:3000 -it react-app-image
$ docker run --name node-backend --rm -d -p 80:80 node-app-image
```

instead of going through localhost to access containerized node-app which required port mapping to be published on step1 :

- `fetch('http://localhost/goals`

we could use docker networks now , which will simply require container name

- `fetch('http://node-backend/goals`

```bash
$ docker build -t react-app-image .
$ docker run --name goals-frontend --rm -d --network web-app-network   -p 3000:3000 -it react-app-image
```

# STH WENT WRONG !

If you keep in mind and understand that all the front end react code here, this JavaScript code, is actually running in the browser, not on some server. And that's a key difference compared to this node code.

- This is executed by the node runtime on the server, directly in the container.

- For react, that's different, there, we just run NPM start, which does only one thing, it starts a development server,which serves this basic react application. The react code however, is not executed inside of the container. This always runs in the browser.

And that means that our nice code here where we reach out to goals backend, does not run in the container where docker would be able to translate this, it runs in the browser and the browser has no idea what goals backend should be. And that is not a bug or anything like that.
This is just related to what react is and where it helps us and that with it you build applications that run in the browser, not on the server. And therefore using the container names here, is simply not an option because this code here is not executed in a docker container, it's running in a browser. The only thing which is running in a docker container here, is the development server, serving this application,
but that's not enough.

thus we should go back to

- `fetch('http://localhost/goals`

this will also require port 80 of node app to be published :

```bash
$ docker run --name node-backend --rm -d -p 80:80 node-app-image
```

also it doesn't make anymore sense to have react container in the network
because the part which runs in the container, the development server, doesn't care about the network, it doesn't interact with the node API or the database. And the part that would interact with the API is not executed in the docker environment.Instead we run this front end container as we did before.

```bash
$ docker run --name goals-frontend --rm -d --network web-app-network   -p 3000:3000 -it react-app-image

```

#### USING NGINX FOR ABOVE PROBLEM ?

# PART3- ADDING DOCKER VOLUMES

## Dockerizing the MongoDB Service

### step3

Check https://hub.docker.com/_/mongo for the location of :data/db in mongo image as well as env variables

```bash
$ docker run --name mongodb \
-v data:/data/db \
--rm -d \
--network web-app-network \
-e MONGO_INITDB_ROOT_USERNAME=burak \
-e MONGO_INITDB_ROOT_PASSWORD=unuvar \
mongo
```

we'll also need to update app code for connection

https://docs.mongodb.com/manual/reference/connection-string/

```js
//   'mongodb://mondodb:27017/course-goals',
//   'mongodb://username:password@mongodb:27017/course-goals',

 `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals?authSource=admin`,
```

```bash
$ docker build -t node-app-image .
$ docker run --name node-backend --rm -d -p 80:80 --network web-app-network node-app-image
```

## Dockerizing the Node App

### step 3

```bash
$ docker run --name node-backend \
-v logs:/app/logs \
-v $(pwd):/app \
-v /app/node_modules \
-e MONGODB_USERNAME=burak \
-e MONGODB_PASSWORD=unuvar \
--rm \
-d \
--network goals-net \
-p 80:80 \
goals-node
```

- Longer container internal paths have precedents and overwrite shorter paths.
  So in the example of logs here, we already do ensure that logs written by the container
  are not overwritten by the local logs folder. Thus, the logs in the container will survive, regardless
  of any local change.

- use nodemon for development environment

```json

  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "nodemon app.js"
  },
  ...
  ...
  "devDependencies": {
    "nodemon": "^2.0.4"
  }
}
```

```Dockerfile
# CMD [ "node", "app.js" ]
CMD ["npm", "start"]
```

- use dockerignore

## Dockerizing the React App

### part 3

```bash
$ docker build -t react-app-image .

$ docker run --name goals-frontend \
 -v $(pwd)/src:/app/src \
 --rm \
 -d \
 -p 3000:3000 \
 -it \
react-app-image
```

- In this code as we're only binding source folder, no need to exculde ignore node_modules by anonymous volumes. They're not included in mapping thus by default container will use its own node_modules
- no need for nodemon as react development environment already responds to updates (watch the files and reload by default)

- use dockerignore
