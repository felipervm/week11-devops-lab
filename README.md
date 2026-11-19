# Week 11 Containerization Lab

Felipe Mattos

This lab builds a Python Flask image with Docker, runs it on host port 4000, and deploys three Nginx replicas with a NodePort Service on Minikube.

## Contents

- lab11-Docker/: Flask application, Dockerfile and Python requirements.
- k8/: Nginx Deployment and NodePort Service from the class repository.
- evidence/: real console screenshots and command transcripts.
- report/Week11_Containerization_Report.pdf: combined submission report.

## Requirements

Docker Desktop with Linux containers, an existing Docker Hub account, Git, kubectl, and Minikube. Tested on Windows 11 with Docker 29.8.2, Python 3.11, Minikube 1.39.0, and Kubernetes 1.37.0. The image build resolved Flask 3.1.3 and Redis 8.1.0.

## Part I Docker

Run in PowerShell from the repository root:

~~~powershell
docker run --rm hello-world
Set-Location lab11-Docker
docker build --tag=hellodocker .
docker image ls hellodocker
docker run -d --name week11-flask -p 4000:80 hellodocker
curl.exe --fail --silent --show-error http://localhost:4000
docker logs --tail 8 week11-flask
docker container stop week11-flask
docker container ls --filter name=week11-flask
docker container ls -a --filter name=week11-flask
Set-Location ..
~~~

The response contains Hello Felipe!, the container hostname, and cannot connect to Redis, counter disabled. That fallback is expected: the lab installs the Redis client library but does not start a Redis server. The first request can take time while the client attempts the connection.

The Dockerfile uses the supplied Python 3.11 base, caches dependency installation separately from application code, avoids disabling pip TLS verification, and defines NAME=Felipe using current ENV syntax. Detached mode (-d) permits HTTP testing in the same terminal; the handout port mapping 4000:80 is retained.

## Part II Kubernetes

~~~powershell
minikube start -p week11-lab --driver=docker --cpus=2 --memory=2200mb --keep-context
minikube status -p week11-lab
Set-Location k8
kubectl --context week11-lab get nodes
kubectl --context week11-lab apply -f nginx-deploy.yaml
kubectl --context week11-lab apply -f nginx-service.yaml
kubectl --context week11-lab rollout status deployment/nginx --timeout=180s
kubectl --context week11-lab get pods -l app=nginx -o wide
kubectl --context week11-lab get svc nginx-nodeport
kubectl --context week11-lab get deployment nginx
minikube -p week11-lab service nginx-nodeport --url
~~~

Keep the last command running. In a second terminal, use curl.exe with the actual URL printed by Minikube. On Windows with the Docker driver, a temporary localhost tunnel is used and its port changes each run. The real URL and HTTP 200 result are recorded in the evidence and PDF.

The Service exposes port 80 as NodePort 30080 and routes traffic to pods labelled app=nginx. Three replicas must be Ready. The supplied readiness and liveness probes and resource requests/limits are preserved.

## Cleanup

Stop the Minikube service tunnel with Ctrl+C, then run from k8/:

~~~powershell
kubectl --context week11-lab delete -f nginx-deploy.yaml
kubectl --context week11-lab delete -f nginx-service.yaml
kubectl --context week11-lab get svc nginx-nodeport --ignore-not-found
minikube stop -p week11-lab
~~~

The original Kubernetes context is preserved by --keep-context and explicit --context week11-lab on every kubectl command. No unrelated cluster resources are changed.

## Source

Forked from https://github.com/pritamworld/DevOps_Kubernetes_exercise9 . The supplied lab archive provided the Flask starter and Docker instructions. The Nginx manifests retain the class repository image nginx:1.25-alpine. This is a classroom exercise, not a production deployment.
