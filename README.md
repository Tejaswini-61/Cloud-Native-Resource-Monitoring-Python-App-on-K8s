## Cloud Native Resource Monitoring Python App on K8s! ##

## 🌐 Cloud Native Resource Monitoring Python App on Kubernetes ##

This project monitors CPU and Memory usage in real time using Python Flask.
It is containerized using Docker, pushed to AWS ECR, and deployed on Kubernetes (EKS).
The app uses psutil to fetch system metrics and Plotly to display interactive gauges on a web dashboard. Alerts are displayed when CPU or memory exceeds 80%.

## ⚙️ Prerequisites ##

Before starting, ensure the following tools are installed and configured:

Python 3 – required to run the Flask application.

Docker – required to build and run containers so the app can run anywhere.

AWS CLI configured – required to push Docker images to AWS ECR.

kubectl installed – required to deploy and manage applications on Kubernetes.

Code editor (like VS Code) – to view and edit the code.

## 🚀 Step 1: Run Flask App Locally ##

## Step 1: Clone the project repository ##

This copies the project from GitHub to your local machine and navigates into the project directory so all commands run in the correct location.

`git clone <repository_url>`
`cd cloud-native-monitoring-app `


## Step 2: Install dependencies ##
Installs all Python libraries needed for the app, including Flask, psutil, Plotly, boto3, and Kubernetes client. This ensures the app can run without missing packages.

`pip install -r requirements.txt`


## Step 3: Run the Flask application ##
Starts the Flask web server. The app will read CPU and Memory metrics and serve the dashboard at localhost:5000.

`python app.py`


## Step 4: Open the dashboard ##
Open this URL in your browser to see real-time CPU and Memory metrics along with alerts for high usage.

`http://localhost:5000`

## 🐳 Step 2: Dockerize the Flask Application ##

## Step 1: Create a Dockerfile ##

The Dockerfile defines how the Docker image is built. It sets the base Python image, installs dependencies, copies your application code, exposes port 5000, and runs Flask.

`FROM python:3.9-slim-buster`
`WORKDIR /app`
`COPY requirements.txt .`
`RUN pip install --no-cache-dir -r requirements.txt`
`COPY . .`
`ENV FLASK_RUN_HOST=0.0.0.0`
`EXPOSE 5000`
`CMD ["flask", "run"]`


## Step 2: Build the Docker image ##

Builds a Docker image named my-flask-app based on the Dockerfile. This creates a containerized version of your application that can run anywhere Docker is installed.

`docker build -t my-flask-app .`


## Step 3: Run the Docker container ##

Runs the Flask app inside a Docker container and maps port 5000 from the container to your local machine.

`docker run -p 5000:5000 my-flask-app`


## Step 4: Open the dashboard in a browser##

Open this URL to verify the app is running inside Docker.

`http://localhost:5000`

## ☁️ Step 3: Push Docker Image to AWS ECR ##

## Step 1: Create an ECR repository using Python ##

This Python script creates a repository in AWS ECR to store your Docker image and prints the repository URI for later use.

`import boto3`
`ecr_client = boto3.client('ecr')`
`repository_name = 'my-monitoring-app'`
`response = ecr_client.create_repository(repositoryName=repository_name)`
`print(response['repository']['repositoryUri'])`


## Step 2: Authenticate Docker with ECR ##

Allows Docker to securely push images to AWS ECR by logging in with your AWS credentials.

`aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account_id>.dkr.ecr.<region>.amazonaws.com`


## Step 3: Tag the Docker image ##

Tags the local Docker image with the ECR repository URI so it can be pushed.

`docker tag my-flask-app:latest <ecr_repo_uri>:latest`


## Step 4: Push the Docker image to ECR ##

Uploads your Docker image to AWS ECR so it can be deployed to Kubernetes.

`docker push <ecr_repo_uri>:latest`

## ☸️ Step 4: Deploy to Kubernetes (EKS) ##

## Step 1: Ensure Kubernetes cluster is ready ##

Make sure an EKS cluster exists, node groups are created, and kubectl is configured.

## Step 2: Create Deployment and Service using Python ##

This Python script defines a Deployment to run the Docker container in Kubernetes Pods and a Service to expose it on port 5000.

from kubernetes import client, config

config.load_kube_config()
api_client = client.ApiClient()

deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="my-flask-app"),
    spec=client.V1DeploymentSpec(
        replicas=1,
        selector=client.V1LabelSelector(match_labels={"app": "my-flask-app"}),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(labels={"app": "my-flask-app"}),
            spec=client.V1PodSpec(containers=[
                client.V1Container(
                    name="my-flask-container",
                    image="<ecr_repo_uri>:latest",
                    ports=[client.V1ContainerPort(container_port=5000)]
                )
            ])
        )
    )
)

api_instance = client.AppsV1Api(api_client)
api_instance.create_namespaced_deployment(namespace="default", body=deployment)

service = client.V1Service(
    metadata=client.V1ObjectMeta(name="my-flask-service"),
    spec=client.V1ServiceSpec(
        selector={"app": "my-flask-app"},
        ports=[client.V1ServicePort(port=5000)]
    )
)

api_instance = client.CoreV1Api(api_client)
api_instance.create_namespaced_service(namespace="default", body=service)


## Step 3: Run the deployment script ##

Executes the script to create the Deployment and Service in Kubernetes.

`python eks.py`


## Step 4: Verify resources ##

Checks that the Deployment, Service, and Pods are running correctly.

`kubectl get deployments -n default`
`kubectl get services -n default`
`kubectl get pods -n default`

## Step 5: Access the application ##

Forwards the service to your local machine so you can access the dashboard.

`kubectl port-forward service/my-flask-service 5000:5000`

Open in browser:

http://localhost:5000