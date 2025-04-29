# Docker & Kubernetes Deployment and Monitoring Project

✨ 30-second Summary
-

In today's tech landscape, Docker, Kubernetes, Prometheus, and Grafana are critical skills for any SysAdmin II, DevOps Engineer, or Data Engineer. In this project, I containerized a Python Flask application, deployed it on a Kubernetes cluster, and implemented a full monitoring solution using Prometheus and Grafana. These technologies are fundamental for managing scalable, observable infrastructure in real-world production environments.

Skills and Tasks Covered:
-

🛠️ Docker Image Creation: Containerizing a Python Flask application.

📁 Kubernetes Deployment: Managing application lifecycle with YAML manifests.

🔢 Metrics Instrumentation: Exposing custom Prometheus metrics from the app.

🌐 Prometheus Setup: Deploying and configuring Prometheus in Kubernetes.

🔍 Grafana Dashboards: Visualizing application metrics with Grafana.

This project fits into a broader learning path around cloud-native operations, microservices, observability, and infrastructure automation. It mirrors the kind of full-stack system management you'd encounter in a SysAdmin or DevOps role.

Project Title

Docker & Kubernetes Deployment with Monitoring
-
Overview

In this project, I:

Built a simple Flask API application with monitoring endpoints.

Containerized the app using Docker.

Deployed the containerized app to Kubernetes.

Set up Prometheus to collect metrics from the app.

Deployed Grafana to visualize these metrics.

Primary Technologies:
-
Docker

Kubernetes (Minikube/kind)

Prometheus

Grafana

Flask (Python)

Deployment Platform:

Local Kubernetes Cluster (Minikube or kind)

Project Structure

project-directory/
├── app.py
├── requirements.txt
├── Dockerfile
├── deployment.yaml
├── prometheus-cm.yaml
├── prometheus-deploy.yaml
├── grafana-deploy.yaml

Screenshot Gallery
-
📸 📌 Screenshot Placeholder: Flask App running locally
<img width="450" alt="pyapp" src="https://github.com/user-attachments/assets/e9892445-831f-4da1-acac-b65a2dc20ff8" />
<img width="302" alt="dockerapp" src="https://github.com/user-attachments/assets/85e50e68-85e8-4e7a-b047-d5c0e39fe7e1" />


📸 📌 Screenshot Placeholder: Docker container running
<img width="450" alt="docker" src="https://github.com/user-attachments/assets/7f398e63-fa9e-4a75-904f-8739f5e03af3" />
<img width="302" alt="dockerapp" src="https://github.com/user-attachments/assets/b0015587-2164-466a-8d04-5e46bf0f5d6c" />

📸 📌 Screenshot Placeholder: Kubernetes pods and services
<img width="453" alt="getpods" src="https://github.com/user-attachments/assets/6b94f654-b621-4554-abc2-3bd164a03d1c" />

📸 📌 Screenshot Placeholder: Prometheus Target UP view
<img width="474" alt="prom" src="https://github.com/user-attachments/assets/11a7cb22-711f-4533-a1c0-fc11ab550a09" />

📸 📌 Screenshot Placeholder: Grafana Dashboard 
<img width="480" alt="grafana" src="https://github.com/user-attachments/assets/47181233-c41b-4c3f-a20a-6819b0b2901f" />

Detailed Project Sections

# 1. Build Flask Application

What
-
Write a simple app.py with /hello and /metrics endpoints.

from flask import Flask, request
from prometheus_client import Counter, generate_latest

app = Flask(__name__)
REQUEST_COUNT = Counter('app_requests_total', 'Total HTTP requests to the app')

@app.route('/hello')
def hello():
    REQUEST_COUNT.inc()
    name = request.args.get('name', 'world')
    return {"message": f"Hello, {name}!"}

@app.route('/metrics')
def metrics():
    return generate_latest(), 200, {'Content-Type': 'text/plain; charset=utf-8'}

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)

Why
-
Flask is lightweight and great for building simple APIs, common in internal tooling.

Real-World Relevance

Knowing how an application exposes metrics is crucial for effective monitoring.

Learning Moment: Understand HTTP endpoints and Prometheus instrumentation basics.

Take Screenshot Here: Flask app running locally at http://localhost:5000/hello

<img width="287" alt="appweb" src="https://github.com/user-attachments/assets/7d2591e7-7bc3-4b72-ace9-315721c83fce" />


# 2. Dockerize the Application

Create a Dockerfile to containerize the app.

FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt ./
RUN pip install -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]

Build and run:

docker build -t myflaskapp:1.0 .
docker run -d -p 5000:5000 myflaskapp:1.0

Why
-
Containers allow consistent deployment across environments.

Real-World Relevance

Containerization is a must-know skill for any modern IT professional.

Learning Moment: Layered Docker images, best practices (like using slim base images).

Take Screenshot Here: Docker container running and app accessible.
<img width="450" alt="docker" src="https://github.com/user-attachments/assets/59e01d94-2362-4ba5-8ed4-7764539d7874" />

<img width="302" alt="dockerapp" src="https://github.com/user-attachments/assets/7e939ba2-62ca-422a-9d78-67a78c9e2eac" />

Troubleshooting Tip: If the container fails, use docker logs <container-id>.

# 3. Deploy to Kubernetes

What
-
Write a deployment.yaml manifest.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myflaskapp-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myflaskapp
  template:
    metadata:
      labels:
        app: myflaskapp
    spec:
      containers:
      - name: myflaskapp
        image: myflaskapp:1.0
        ports:
        - containerPort: 5000
---
apiVersion: v1
kind: Service
metadata:
  name: myflaskapp-service
spec:
  selector:
    app: myflaskapp
  ports:
  - port: 80
    targetPort: 5000
  type: NodePort

Deploy:

kubectl apply -f deployment.yaml

Why
-
Kubernetes handles scaling, rolling updates, and service discovery.

Real-World Relevance

SysAdmins often maintain Kubernetes applications, even if they don't write them.

Learning Moment: Understand Deployments vs. Services.

Take Screenshot Here: kubectl get pods and service status output.
<img width="453" alt="getpods" src="https://github.com/user-attachments/assets/57333126-1ea6-43b4-9ecd-7efa299f3c39" />


Troubleshooting Tip: Use kubectl describe pod <pod-name> for pod errors.

# 4. Set Up Prometheus Monitoring

Write Prometheus config:

apiVersion: v1
kind: ConfigMap
metadata:
  name: prom-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
    - job_name: 'flaskapp'
      static_configs:
      - targets: ['myflaskapp-service:80']

Deploy Prometheus:

kubectl apply -f prometheus-cm.yaml
kubectl apply -f prometheus-deploy.yaml

Why
-
Prometheus collects time-series metrics crucial for diagnosing performance issues.

Real-World Relevance

Prometheus is heavily used in production environments for SRE and DevOps tasks.

Learning Moment: Configuration management via ConfigMaps.

Take Screenshot Here: Prometheus UI Target screen.
<img width="474" alt="prom" src="https://github.com/user-attachments/assets/b5ba9df2-e240-4130-9a6b-9da84c1a0143" />


Troubleshooting Tip: If target is DOWN, check service DNS resolution and access.

# 5. Visualize with Grafana

Deploy Grafana:

kubectl apply -f grafana-deploy.yaml

Configure Prometheus as a data source inside Grafana (URL: http://prom-service:9090).
Create a dashboard using rate(app_requests_total[1m]) query.

Why
-
Grafana makes metrics easy to understand through rich visualizations.

Real-World Relevance

Dashboards and alerts are critical for maintaining service health.

Learning Moment: PromQL basics (Prometheus Query Language).

Take Screenshot Here: Grafana Dashboard
<img width="480" alt="grafana" src="https://github.com/user-attachments/assets/06bd75f8-a9a1-4a8b-a683-d49d5a20228a" />



Troubleshooting Tip: Confirm correct service URL for Prometheus source in Grafana.

Challenges & Solutions

Challenge

Solution

Docker image could not connect on cluster

Built image directly inside Minikube docker-env

Prometheus target was DOWN

Corrected service port in Prometheus config

Grafana could not find Prometheus

Verified internal service DNS resolution

Summary
-
📈 Built and containerized a Python Flask API.

🌍 Deployed the containerized app to a Kubernetes cluster.

🧬 Configured and deployed Prometheus to scrape app metrics.

🔢 Connected Grafana to Prometheus and built dashboards.

🎉 Mastered container orchestration and observability fundamentals.

Congratulations, you've completed the project! 🚀
-
References
-
Docker Official Documentation

Kubernetes Official Documentation

Prometheus Official Documentation

Grafana Official Documentation

Flask Official Documentation

Environment Cleanup
-
Delete all Kubernetes resources:

kubectl delete -f deployment.yaml
kubectl delete -f prometheus-cm.yaml
kubectl delete -f prometheus-deploy.yaml
kubectl delete -f grafana-deploy.yaml

Stop Minikube (if used):

minikube stop

Remove Docker images (optional):

docker rmi myflaskapp:1.0

Fresh start ready for the next project! 🚀

