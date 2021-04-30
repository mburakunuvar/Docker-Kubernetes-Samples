### deployment-03 test on remote host ECS => production

#### ECS Container Definition :

- `--name` : container name
- image : container image
  - for docker hub, repo name
  - for all others full url
- `-p 80:80` : publish port mapping

For Fargate, you don't need to specify two numbers here, because the container internal port will always be mapped to the same port outside of the container.

- Environment :

  - CMD within Dockerfile could be replaced
  - Entry Point within Dockerfile could be replaced
  - Working Directory `--workdir` could be replaced
  - Environment Variables : `--env`

- Network Settings :
- Storage and Logging:
  - `-v` bind volumes and mappings

#### ECS Task Definition :

Think of it like one remote machine which runs one or more containers

- Launch Type
- Memory , CPUD

"A task definition is a blueprint for your application, and describes one or more containers through attributes. Some attributes are configured at the task level but the majority of attributes are configured per container."

#### ECS Service :

how to execute task

- Load Balancer

"A service allows you to run and maintain a specified number (the "desired count") of simultaneous instances of a task definition in an ECS cluster."

#### ECS Cluster

overall network, in order to group multiple containers in one cluster

"The infrastructure in a Fargate cluster is fully managed by AWS. Your containers run without you managing and configuring individual Amazon EC2 instances."

# re-run updated image

- **Alternative 1 :** Create New Revision of task configuration and update service using that, then AWS ECS will automatically pull the updated image.

=> When you create a new task and then launch and use service using that task, AWS will use the latest image version

- **Alternative 2 :** Update Service and Force new Deployment
