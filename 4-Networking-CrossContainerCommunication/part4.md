# Docker Networking

### container to web

Out of the box, containers can communicate with web by default

### container to localhost : host.docker.internal

Imagine you've a mongo db running on your local machine

for local web server to local mongo db this will work :

- `"mongodb://mongodb:27017/swfavorites"`, => local

for containerized web server to local mongo db : :

- `"mongodb://host.docker.internal:27017/swfavorites"`, =>container to local

It's translated to the IP address off your host machine as seen from inside the Docker container.

### localhost(nodeapp) to container

in upcoming chapter

```bash
$ docker run --name mongodb --rm -d -p 27017:27017 mongo
```

### container to container (cross)

```bash
$ docker run -d --name mongodb mongo
# host.docker.internal will no more work
$ docker container inspect mongodb
## IP Address : 172.17.0.2
```

- `"mongodb://172.17.0.2:27017/swfavorites"` => container to container

# Introducing Docker Networks: Elegant Container to Container Communication

```bash
$ docker run -d --name mongodb --network mynetwork mongo
# this will stop since the run command will not create network by default
$ docker create network mynetwork
$ docker network ls
$ docker run -d --name mongodb --network mynetwork mongo
$ docker run -d --name webserver --network mynetwork -rm -p 3000:3000 image-name
```

- `"mongodb://172.17.0.2:27017/swfavorites"` => container to container
- `"mongodb://mongodb:27017/swfavorites"` => container to container

Two containers normally are not able to talk to each other, unless you create such a container network or you manually look up the container IP as you saw,

```js
mongoose.connect(
  // "mongodb://mongodb:27017/swfavorites", => local
  // "mongodb://host.docker.internal:27017/swfavorites", =>container to local
  // $ docker container inspect mongodb
  // "mongodb://172.17.0.2:27017/swfavorites", => container to container
  // easier approach for the above line :
  // => container to container, after launching all in the same networks
  "mongodb://mongodbcontainer:27017/swfavorites",
  { useNewUrlParser: true },
  (err) => {
    if (err) {
      console.log(err);
    } else {
      app.listen(3000);
    }
  }
);
```

Docker will not, important, not go ahead and start replacing your source code under the hood.

That is not what it's doing. It's not replacing your source code. So it will not read your source code, see a container name and plug in the IP address of that container into your code.

That would be way too complex and it's not what it's doing. Instead Docker owns the environment in which your application runs. And if your application sends an HTTP request
or a MongoDB request or any other kind of request that leaves the container, then Docker is aware of that. And it's that point of time where Docker then is able to resolve the address,
or the container name or host Docker internal and replace it with the actual IP address,
since it is aware of the surrounding containers and the host machine. So it's only when a request leaves the container.

If a request does not leave a container, or if a request is generated somewhere else,
for example, in the browser. If users are visiting your web app and JavaScript code is running in their browser and a request is sent from there, then Docker is not doing anything
because it's not replacing the source code.

# Docker Network Drivers

Docker Networks actually support different kinds of "Drivers" which influence the behavior of the Network.

The default driver is the "bridge" driver - it provides the behavior shown in this module (i.e. Containers can find each other by name if they are in the same Network).

The driver can be set when a Network is created, simply by adding the --driver option.

docker network create --driver bridge my-net
Of course, if you want to use the "bridge" driver, you can simply omit the entire option since "bridge" is the default anyways.

Docker also supports these alternative drivers - though you will use the "bridge" driver in most cases:

host: For standalone containers, isolation between container and host system is removed (i.e. they share localhost as a network)

overlay: Multiple Docker daemons (i.e. Docker running on different machines) are able to connect with each other. Only works in "Swarm" mode which is a dated / almost deprecated way of connecting multiple containers

macvlan: You can set a custom MAC address to a container - this address can then be used for communication with that container

none: All networking is disabled.

Third-party plugins: You can install third-party plugins which then may add all kinds of behaviors and functionalities

As mentioned, the "bridge" driver makes most sense in the vast majority of scenarios.
