---
layout: post
title: "How to deploy containerized Flask application to a Kubernetes with Kubeadm on Virtual Machines - Part 3"
date: 2023-05-22 00:00:00 +0800
excerpt_separator: <!-- excerpt -->
---
This is the third and final part of a series of hands-on tutorials for deploying containerized Flask application to a
Kubernetes with Kubeadm on Virtual Machines.
<!-- excerpt -->

# Series
1. [Part 1]({% post_url 2023-03-17-how-to-deploy-containerized-flask-application-to-a-kubernetes-with-kubeadm-on-virtual-machines-part-1%})
2. [Part 2]({% post_url 2023-03-17-how-to-deploy-containerized-flask-application-to-a-kubernetes-with-kubeadm-on-virtual-machines-part-2%})
2. [Part 3]({% post_url 2023-03-17-how-to-deploy-containerized-flask-application-to-a-kubernetes-with-kubeadm-on-virtual-machines-part-3%})


# Overview
In this tutorial, we are going to 
1. Install Docker
2. Write Flask application
3. Contanerize Flask application
4. Push Docker image to Docker Hub
5. Deploying the container on the cluster 

# Requirements
1. A Docker account. You can sign up [here](https://hub.docker.com/signup).

# 1. Docker installation (Master only)
First, we need to install `Docker`

## 1.1 Update the apt package index
```
sudo apt update
sudo apt-get install ca-certificates curl gnupg lsb-release
```
## 1.2 Add Docker’s official GPG key:

```
sudo mkdir -m 0755 -p /etc/apt/keyrings
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

## 1.3 setup repository
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
## 1.4 Install Docker Engine
```
sudo apt-get update
sudo apt-get install docker-ce
```
Select `N` when asking about `Configuration file '/etc/containerd/config.toml'` since we already setup in `containerd runtime` setup section in Part 2.

## 1.5 (Optional) Executing the Docker Command Without `sudo`
```
sudo usermod -aG docker ${USER}
su - ${USER}
```

# 2. Write a Flask application (Master only)
## 2.1 Create a new directory
Create a new directory named `helloworld` and change the directory into it
```
mkdir helloworld
cd helloworld
```
## 2.2 Write Flask API 
Create a file named `app.py` and paste the following code into it.

```py
from flask import Flask

app = Flask(__name__)


@app.route("/")
def hello_world():
    return "Hello Kubernetes"


if __name__ == "__main__":
    app.run(debug=True, host="0.0.0.0", port=8080)
```
### 2.3 Add Flask dependency
Create a file named `requirements.txt` and paste the following code into it.

```
Flask
```

# 3 Contanerize Flask Application (Master only)
## 3.1 Create a Docker file
Create a file named `Dockerfile` with the following contents.
``` Dockerfile
# Use the official lightweight Python image.
FROM python:3.10-slim

# Copy local code to the container image.
ENV APP_HOME /app
WORKDIR /app
COPY . ./

# Install production dependencies.
RUN pip install --no-cache-dir -r requirements.txt

# Run the web service on container startup.
CMD [ "flask", "run","--host","0.0.0.0","--port","8080"]
```
## 3.2 Build Docker image
Build a Docker image called `flask-backend`. Since we need to upload to the Docker hub, we need to use our login ID as a preface. So that will become `<YOUR-DOCKER-ID>/flask-backed`. For my case, I need to use `aknay/flask-backend`.
```
docker build --tag aknay/flask-backend .
```
You can list the docker that you just built by running `docker image ls` and you should see the following image listed.
```
REPOSITORY            TAG       IMAGE ID       CREATED          SIZE
aknay/flask-backend   latest    ebc930c236c8   21 minutes ago   136MB
```

> You can remove the docker image using `docker rmi -f <IMAGE ID>` 

## 3.3 Testing Docker image locally
You can run the image by running

```
docker run --detach --publish 8080:8080 <IMAGE ID>
```
> The command is `docker run --detach --publish <TRAFFIC-FROM-PORT>:<TO-CONTAINER-PORT> <IMAGE-ID>`

When you run `docker ps`, you should see an output similar to this.
```
CONTAINER ID   IMAGE                 COMMAND                  CREATED          STATUS          PORTS                                       NAMES
513453b7a0ad   aknay/flask-backend   "flask run --host 0.…"   18 seconds ago   Up 16 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   naughty_neumann8080/tcp, :::8080->8080/tcp   pedantic_almeida
```
If you use `curl localhost:8080` on the terminal, You should see the message `Hello Kubernetes` on the terminal (which is a reply from our Flask API).

## 3.4 Stop Docker image
We can now stop the running container with this command `docker stop <CONTAINER ID>` since we successfully tested the docker.


# 4. Push the Docker image to Docker Hub (Master only)
We need to push our Docker image to Docker Hub because the image will be later downloaded and deployed on the cluster. 

## 4.1 Login to Docker Hub
Log in to Docker Hub using your username and password
```
docker login
```

## 4.2 Push image to Docker Hub
```
docker push <YOUR-DOCKER-HUB-ID>/flask-backend
```

# 5. Deploying the container on the cluster (Master only)

## 5.1 Create a deployment
A Deployment provides declarative updates for Pods and ReplicaSets. You can learn more about it from [here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/). So we need a create a `deployment.yml` first. Note that you need to change the image name (in deployment.yml) based on your `docker hub id`. Mine is `aknay/flask-backend`.

```yml
kind: Service
apiVersion: v1
metadata:
  name: flask-app-service
  labels:
    app: flask-app
spec:
  selector:
    app: flask-app
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
  labels:
    app: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image: aknay/flask-backend
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          protocol: TCP
```
## 5.2 Deploying on cluster
Then create the deployment by `kubectl apply -f deployment.yml`.

Let's check the service first by `kubectl get service`. Now our `flask-app-service` is running.

```
NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
flask-app-service   ClusterIP   10.96.101.16   <none>        8080/TCP   12s
kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP    160m
```
## 5.3 Checking deployment
Now we check the deployment by `kubectl get deployment`. As you see below, our app is running with two replicas (based on what we defined in `deployments.yml`).
```
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
flask-app   2/2     2            2           107s
```
## 5.4 Testing our cluster
Let's test our app by `curl <CLUSTER-IP>:8080` (which is running `flash-app-service`) and you should see the expected message `Hello Kubernetes`.

> For me, I need to use my cluster IP address `curl 10.96.101.16:8080`

Now we successfully deploy our docker image to our cluster.


## 5.5 (Reference) Listing Pods with Nodes
You can list all pods on nodes using the following command
```
kubectl get pods --all-namespaces -o wide
```

You should see the following.
```
NAMESPACE      NAME                                  READY   STATUS    RESTARTS   AGE     IP           NODE          NOMINATED NODE   READINESS GATES
default        flask-app-6bdfd677c5-7gc98            1/1     Running   0          7m48s   10.244.1.4   worker-vbox   <none>           <none>
default        flask-app-6bdfd677c5-t4fjx            1/1     Running   0          7m48s   10.244.1.3   worker-vbox   <none>           <none>
kube-flannel   kube-flannel-ds-c4wfg                 1/1     Running   0          147m    10.0.2.15    master-vbox   <none>           <none>
kube-flannel   kube-flannel-ds-p775s                 1/1     Running   0          147m    10.0.2.5     worker-vbox   <none>           <none>
kube-system    coredns-5d78c9869d-f5dgw              1/1     Running   0          167m    10.244.1.2   worker-vbox   <none>           <none>
kube-system    coredns-5d78c9869d-f6wmn              1/1     Running   0          167m    10.244.0.2   master-vbox   <none>           <none>
kube-system    etcd-master-vbox                      1/1     Running   0          167m    10.0.2.15    master-vbox   <none>           <none>
kube-system    kube-apiserver-master-vbox            1/1     Running   0          167m    10.0.2.15    master-vbox   <none>           <none>
kube-system    kube-controller-manager-master-vbox   1/1     Running   0          167m    10.0.2.15    master-vbox   <none>           <none>
kube-system    kube-proxy-gzcgl                      1/1     Running   0          167m    10.0.2.15    master-vbox   <none>           <none>
kube-system    kube-proxy-v5lzg                      1/1     Running   0          154m    10.0.2.5     worker-vbox   <none>           <none>
kube-system    kube-scheduler-master-vbox            1/1     Running   0          167m    10.0.2.15    master-vbox   <none>           <none>
```

You can see that our two pods in running on the worker as expected.

## 5.6 (Reference) Cleaning up
1. You can delete the deployment with `kubectl kubectl delete -f deployment.yml` or you can deleted individually like so. `kubectl delete deployment <YOUR DEPLOYMENT NAME>`.
2. For the service, `kubectl delete service <YOUR SERVICE NAME>`

# Reference
1. https://docs.docker.com/engine/install/ubuntu/
2. https://cloud.google.com/run/docs/quickstarts/build-and-deploy/deploy-python-service