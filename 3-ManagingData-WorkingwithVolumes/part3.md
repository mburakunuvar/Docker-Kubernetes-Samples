### anonymous volumes

exists as long as container exists, attached to a container

And that of course means that docker doesn't have to store all the data inside of the container and doesn't have to manage all the data inside of this container read write layer.
But that instead it can outsource certain data to your host machine file system.

```Dockerfile
VOLUME ["app/feedback"]
```

or

```bash
-v /app/node_modules
```

Anonymous volumes are removed automatically, when a container is removed. But this only happens when you start / run a container with the --rm option.

If you start a container without that option, the anonymous volume would NOT be removed, even if you remove the container (with `docker rm` ...).

Still, if you then re-create and re-run the container (i.e. you run `docker run` ... again), a new anonymous volume will be created. So even though the anonymous volume wasn't removed automatically, it'll also not be helpful because a different anonymous volume is attached the next time the container starts (i.e. you removed the old container and run a new one).

Now you just start piling up a bunch of unused anonymous volumes - you can clear them via `docker volume rm VOL_NAME` or `docker volume prune`.

=> to define parts that shouldn't be overwritten, we'll use anonymous volumes

And this can also help with performance and efficiency.

```yaml
VOLUME [ "/temp" ]
```

### named volumess

are necessary for permanent data

```bash
$ docker run -d -p 3000:80 --rm --name my-container -v feedback:app/feedback my-image
```

will not be deleted when container is shut down, not attached to a container

### bind mounts

managed by you, mainly for development purposes in order to reflect source code changes directly. (you define folder/path )

```bash
$ docker run -d -p 3000:80 --rm --name my-container -v feedback:app/feedback
-v "$(pwd):/app" my-image
# will not work
# we'll also  need anonymous module here for packages
# packages should be on container
$ docker run -d -p 3000:80 --rm --name my-container -v feedback:app/feedback my-image
-v "$(pwd):/app"
# to define parts that shouldn't be overwritten, we'll use anonymous volumes
$ docker run -d -p 3000:80 --rm --name my-container -v feedback:app/feedback
-v "$(pwd):/app" -v /app/node_modules my-image
```

### nodemon

```json
  "scripts": {
    "start": "nodemon server.js"
  },
```

```Dockerfile
# CMD [ "node", "server.js" ]
CMD [ "npm", "start" ]
```

### read-only volumes

to prevent docker from writing into your local

```bash
$ docker run -d -p 3000:80 --rm --name my-container -v feedback:app/feedback
-v "$(pwd):/app:ro" -v /app/node_modules my-image
```

#### IN ORDER TO MAKE WRITING POSSIBLE INTO SOME FOLDERS: SPECIFY ANOTHER SPECIFIC SUB VOLUME TO OVERWRITE MAIN VOLUME

- just like `-v /app/node_modules` replacing bind mount

- For instnace `data:app/feedback` by named volume is longer path than /app:ro; so feedback folder will be writable

- so let's add similar longer path, additional volume

```bash
$ docker run -d -p 3000:80 --rm --name my-container -v feedback:app/feedback
-v "$(pwd):/app:ro" -v /app/node_modules -v /app/temp my-image
```

So above setup will only let container to write into temp folder, nothing else

### DOCKERFILE VS COMMAND LINE FOR ANONYMOUS VOLUMES

CLI will overwrite bond monts, but yaml will not
So if we need to use packages(express) or let some specific folders to be written , ANONYMOUS VOLUMES should be defined through CLI

### managing docker volumes

```bash
$ docker volumes ls
# bind mount will not be listed
# docker run command will create in advance, but manual way is also possible.
$ docker volume create NAME
$ docker volume inspect NAME
$ docker volume rm NAME
$ docker volume prune
```
