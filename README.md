# docker-Java-kubernetes-project
Deploying Java Applications with Docker and Kubernetes

Credit: https://github.com/danielbryantuk/oreilly-docker-java-shopping/

# Microservices CI/CD Pipeline

This project implements a complete CI/CD pipeline for a microservices architecture using Jenkins, Docker, and Kubernetes.

## Project Structure

- `stockmanager/`: Stock management service
- `shopfront/`: Frontend service
- `productcatalogue/`: Product catalog service
- `kubernetes/`: Kubernetes deployment configurations

## Prerequisites

1. Jenkins Server with the following plugins:
   - Pipeline
   - Docker Pipeline
   - Kubernetes
   - Credentials Binding

2. Docker Registry (configure your registry URL in Jenkinsfile)

3. Kubernetes Cluster with kubectl configured

4. Required Jenkins Credentials:
   - `docker-registry-creds`: Docker registry username/password
   - `kubeconfig`: Kubernetes configuration file

## CI/CD Pipeline Stages

1. **Checkout**: Clones the repository
2. **Build and Test**: Builds and tests all services in parallel
3. **Build Docker Images**: Creates Docker images for all services
4. **Push Docker Images**: Pushes images to the Docker registry
5. **Deploy to Kubernetes**: Deploys services to Kubernetes cluster
6. **Verify Deployment**: Ensures successful deployment

## Configuration

1. Update the `DOCKER_REGISTRY` variable in the Jenkinsfile with your Docker registry URL
2. Configure the following Jenkins credentials:
   - Docker registry credentials
   - Kubernetes configuration

## Running the Pipeline

1. Create a new Pipeline job in Jenkins
2. Point it to this repository
3. Set the Pipeline script path to `Jenkinsfile`
4. Run the pipeline

## Monitoring

- Jenkins provides detailed logs for each stage
- Kubernetes dashboard can be used to monitor deployments
- Each service has health endpoints for monitoring

## Troubleshooting

1. Check Jenkins logs for build failures
2. Verify Docker registry access
3. Ensure Kubernetes cluster is accessible
4. Check service health endpoints:
   - StockManager: `http://<host>:8030/health`
   - ShopFront: `http://<host>:8080/health`
   - ProductCatalogue: `http://<host>:8020/health`
