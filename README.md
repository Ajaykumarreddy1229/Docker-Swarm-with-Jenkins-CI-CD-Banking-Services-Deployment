# Docker Swarm with Jenkins CI/CD | Banking Services Deployment

## Project Overview

This project demonstrates a DevOps CI/CD workflow using GitHub, Jenkins, Docker, Docker Hub, Docker Compose, and Docker Swarm to build container images and deploy banking-related web services across multiple nodes.

The application includes an HTML-based banking login interface and a multi-service deployment configuration for Internet Banking, Mobile Banking, Insurance, and Loans.

The objective is to understand source code management, automated image building, image publishing, container orchestration, service scaling, and deployment management.

## Architecture Diagram

![Docker Swarm with Jenkins Project Architecture](ProjectArchitecture.png)

## Technologies Used

- Git and GitHub — source code management
- Jenkins — CI/CD pipeline automation
- Docker — application containerization
- Docker Hub — container image registry
- Docker Compose — multi-service configuration
- Docker Swarm — container orchestration
- Linux / Amazon EC2 — infrastructure
- HTML — banking login interface
- Apache HTTP Server — web server inside the Docker image

## Project Architecture

Developer → GitHub → Jenkins Pipeline → Docker Image Build → Docker Hub → Docker Swarm Manager → Worker Nodes

### Architecture Components

**1. GitHub**

Stores the project files, including the HTML application, Dockerfile, Docker Compose configuration, pipeline script, and project documentation.

**2. Jenkins**

Checks out the source code, builds Docker images, tags and pushes images to Docker Hub, and runs the stack deployment command.

**3. Docker**

Packages the HTML application into an Ubuntu-based container image with Apache installed.

**4. Docker Hub**

Stores versioned images that can be pulled by the deployment infrastructure.

**5. Docker Swarm**

Coordinates services across manager and worker nodes and maintains the desired number of service replicas when sufficient resources are available.

**6. Docker Compose and Docker Stack**

The Compose file defines four services. Docker Stack uses the Compose configuration to deploy and manage those services in the Swarm cluster.

## Repository Structure

```text
dockerproject/
│
├── Dockerfile
├── JenkinsPipeline
├── PROCESS
├── ProjectArchitecture.png
├── docker-compose.yml
├── index.html
└── README.md
```

## Application Details

The project contains an HTML banking login interface and a Compose configuration for four services.

| Service | Docker Hub image | Host port | Configured replicas |
|---|---|---:|---:|
| Internet Banking | trainerreyaz/ib-image:latest | 81 | 3 |
| Mobile Banking | trainerreyaz/mb-image:latest | 82 | 3 |
| Insurance | trainerreyaz/insurance-image:latest | 83 | 3 |
| Loans | trainerreyaz/loans-image:latest | 84 | 3 |

The replica counts represent the desired service configuration, not proof that all replicas are currently running.

## Dockerfile

The existing Dockerfile uses Ubuntu, installs Apache, copies `index.html` into the Apache web directory, and starts Apache in the foreground.

```dockerfile
FROM ubuntu

RUN apt-get update -y

RUN apt-get install apache2 -y

COPY index.html /var/www/html/

CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]
```

### Dockerfile Explanation

- `FROM ubuntu` — selects the Ubuntu base image.
- `RUN apt-get update -y` — updates package information.
- `RUN apt-get install apache2 -y` — installs Apache HTTP Server.
- `COPY index.html /var/www/html/` — copies the application page into Apache's document root.
- `CMD` — starts Apache in the foreground so the container remains running.

## Docker Compose Configuration

The existing `docker-compose.yml` defines four services:

- `internetbanking`
- `mobilebanking`
- `insurance`
- `loan`

Each service references a Docker Hub image, maps a host port to container port 80, and declares three replicas.

The Compose file also declares service volumes under `/var/lib` paths.

For the exact image names, ports, replica settings, and volume declarations, see the [docker-compose.yml file](https://github.com/Ajaykumarreddy1229/dockerproject/blob/master/docker-compose.yml).

## Jenkins CI/CD Workflow

The repository includes a pipeline script named `JenkinsPipeline`.

The intended pipeline stages are:

1. **Checkout:** Retrieve the project source code from GitHub.
2. **Build:** Build a Docker image using the Dockerfile.
3. **Tag:** Assign a registry image tag.
4. **Push:** Authenticate and publish the image to Docker Hub.
5. **Deploy:** Deploy the banking services using Docker Stack.

The pipeline must use the correct repository URL, configured Jenkins credentials, valid image variables, and a deployment agent with access to the Swarm manager.

See the [JenkinsPipeline file](https://github.com/Ajaykumarreddy1229/dockerproject/blob/master/JenkinsPipeline).

## Docker Swarm Setup

Install Docker on the manager and worker nodes and start the Docker service.

Initialize the Swarm cluster on the manager:

```bash
docker swarm init --advertise-addr MANAGER_PRIVATE_IP
```

Generate the worker join command:

```bash
docker swarm join-token worker
```

Run the generated join command on each worker node.

Verify cluster membership:

```bash
docker node ls
```

## Build and Push a Docker Image

From the project directory, build the image:

```bash
docker build -t YOUR_DOCKERHUB_USERNAME/banking-web:v1 .
```

Authenticate securely to Docker Hub:

```bash
docker login
```

Push the image:

```bash
docker push YOUR_DOCKERHUB_USERNAME/banking-web:v1
```

The image name above is an example for the build process. The existing Compose file uses four separate images, so update the corresponding Compose image references if you intend to use this newly built image for a service.

## Deploy the Banking Services

From a manager node, with the required images available:

```bash
docker stack deploy -c docker-compose.yml bank
```

Check the deployed services:

```bash
docker stack services bank
```

Inspect service tasks:

```bash
docker service ps bank_internetbanking
docker service ps bank_mobilebanking
docker service ps bank_insurance
docker service ps bank_loan
```

Check the Swarm cluster:

```bash
docker node ls
```

## Application Access

If the services are deployed successfully and the required ports are allowed by the firewall or EC2 security groups, access them using:

- Internet Banking: `http://SWARM_NODE_PUBLIC_IP:81`
- Mobile Banking: `http://SWARM_NODE_PUBLIC_IP:82`
- Insurance: `http://SWARM_NODE_PUBLIC_IP:83`
- Loans: `http://SWARM_NODE_PUBLIC_IP:84`

Replace `SWARM_NODE_PUBLIC_IP` with the public IP of a suitable Swarm node.

## Key Learning Outcomes

- GitHub repository management
- Dockerfile creation and image building
- Docker Hub image tagging and publishing
- Jenkins pipeline stages
- Docker Compose multi-service configuration
- Docker Swarm cluster initialization
- Service replicas and scaling
- Docker Stack deployment
- Container and deployment troubleshooting

## Improvements to Explore

- Configure GitHub webhooks to trigger Jenkins builds.
- Use Jenkins Credentials for Docker Hub authentication.
- Use unique image tags instead of relying only on `latest`.
- Add pipeline failure handling and deployment verification.
- Configure HTTPS and secure access to the banking interface.
- Add monitoring, logging, and deployment rollback procedures.

## Project Repository

GitHub: https://github.com/Ajaykumarreddy1229/dockerproject

## Disclaimer

This is an educational DevOps project. The banking-themed application is for learning and demonstration purposes only. It is not a production banking system and must not be used to collect real banking credentials or sensitive financial information.
