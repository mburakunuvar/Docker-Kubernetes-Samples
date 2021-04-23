### deployment-04 multiple containers on ECS => production

###

- docker-compose will no more be used
- this will require some change on code ( in terms of networking )
- replace the container names defined by yaml
  - localhost if all containers are sharing the same task
  - other url if hosted outside (Atlas )

update code

```js
  `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@mongodb:27017/course-goals?authSource=admin`,

  `mongodb://${process.env.MONGODB_USERNAME}:${process.env.MONGODB_PASSWORD}@${process.env.MONGODB_URL}:27017/course-goals?authSource=admin`,
```

update default environment variables

```env
MONGODB_URL=mongodb
```

```bash
$ docker build -t logic-tier-image ./backend/
$ docker tag logic-tier-image burakunuvar/logic-tier-image
$ docker push burakunuvar/logic-tier-image
```

#### ECS Container Definition for logic tier:

- `--name` : container name
- image : container image
  - for docker hub, repo name
  - for all others full url
- `-p 80:80` : publish port mapping

For Fargate, you don't need to specify two numbers here, because the container internal port will always be mapped to the same port outside of the container.

- Environment :

  - CMD within Dockerfile could be replaced => COMMAND : node,app.js (should be comma separated list)

  - Entry Point within Dockerfile could be replaced
  - Working Directory `--workdir` could be replaced
  - Environment Variables : `--env` or `--env-file ./.env`

ENV MONGODB_USERNAME=max
ENV MONGODB_PASSWORD=secret
ENV MONGODB_URL=localhost (thanks to ECS feature)

- Network Settings :
- Storage and Logging:
  - `-v` bind volumes and mappings

#### ECS Container Definition for db tier:

- `--name` : container name
- image : mongo (for default on dockerhub)
  - for docker hub, repo name
  - for all others full url
- port : 27017

- Environment Variables : `--env` or `--env-file ./.env`

MONGO_INITDB_ROOT_USERNAME=max
MONGO_INITDB_ROOT_PASSWORD=secret

note: there's no dockerfile for mongo , you aren't building anything so if you don't make changes above, default ones of public image will be used for container

- Storage and Logging:
  - `-v` bind volumes and mappings

### Load Balancer

Create ALB with target group using IPs (for fargate)

- Target Group health check part should be /goals (as only this path is handled in node code)
- update SG of ALB should let port 80
