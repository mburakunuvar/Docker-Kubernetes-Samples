## WE don't need to install tools globally on our host machine.

Int order to run below commands yo need to install Node on your local machine

```
$ npm init
$ npm install
```

But you can use docker environment instead

### One possible approach - Use Container Environment:

```bash
$ docker run -it -d node
$ docker container attach <CONTAINER_ID>
# besides the default command, additional commands
$ docker exec -it <CONTAINER_ID> sh
$ docker exec -it <CONTAINER_ID> npm init
# or replace the default command
$ docker run -it <IMAGE_NAME> sh
$ docker run -it <IMAGE_NAME> npm init
```

### Better Approach : Building First Utility Container using Dockerfile

```bash
$ docker build -t utility-container .
$ docker run -it utility-container npm init
## mirror to local
$ docker run -it -v $(pwd):/home/node/app utility-container npm init
>package name:
> ....
```

And now I could totally uninstall node and NPM on my host machine. And I could still create projects with help of NPM init with help of this utility container.

Note: For node there is not much but for Laravel it could be more of a challlenge.

### Using ENTRYPOINT within Dockerfile

CMD can be replaced/overwritten
ENTRYPOINT whatever written will be appended at the end

```bash
$ docker build -t utility-container-entry .
## mirror to local
$ docker run -it -v $(pwd):/home/node/app utility-container-entry init
>package name:
> ....
## other command to local
$ docker run -it -v $(pwd):/home/node/app utility-container-entry install
$ docker run -it -v $(pwd):/home/node/app utility-container-entry install express --save
```

### Docker-Compose

```bash
$ docker-compose up
## will not work because ENTRY includes just npm with no follow up.
$ docker-compose up init
## fails because docker-compose is for services running
## for utility containers
$ docker-compose exec <CONTAINER_NAME >...
# or another option, targeting a specific service within docker-compose
$ docker-compose run utility-container-entry init
# will work better

$ docker-compose down
# removes containers automatically but not in this case
$ docker-compose run -rm utility-container-entry init
# -rm will be necessary
```

[Utility Containers and Linux]
(https://www.udemy.com/course/docker-kubernetes-the-practical-guide/learn/lecture/22167138#questions/12977214/)
