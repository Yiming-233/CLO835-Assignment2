# CLO835 Assignment 2 — report
| | |
|---|---|
| Name | Yiming Pan|
| Student ID | 101753267 |
| Language chosen | Python |
| Docker Hub image | `ypan69/a2-app` |
| Your repository | `https://github.com/Yiming-233/CLO835-Assignment2` |

## 1. Create Your Own GitHub Repository
### 1.1 GitHub Repository

I created a public GitHub repository and copied the Assignment 2 starter files into the repository root.

The GitHub Actions workflow is located at `.github/workflows/docker.yml`.

![GitHub Repository](<screenshot/1 git-repo.png>)

### 1.2 Git Commit History

![Git Log](<screenshot/1 git-log1.png>)

```text
# Displays the Git commit history
$ git log --oneline
4764046 (HEAD -> main, origin/main, origin/HEAD) Update Dockerfile and workflow
9a69483 Initial commit for CLO835 Assignment 2
```

### 1.3 Question
**Explain why the workflow does not run as required when the assignment files remain one directory level below the repository root.**

GitHub Actions only detects workflows in `.github/workflows` at the repository root, not inside nested folders.

## 2. Select a Programming Language and Implement Version 0.2

I selected Python and used the provided application starter.

The `MESSAGE` constant in `apps/python/app.py` is:

```python
MESSAGE = "Hello world from the CLO835 class!"
```

### 2.1 Run the Application Locally and Health Check

![Local Application](<screenshot/2 local host.png>)

```text
# Starts the Python application on port 8080
$ python app.py
CLO835 app v0.2 listening on :8080
```

```text
# Tests the application response on localhost
$ curl http://localhost:8080
Hello world from the CLO835 class!
```

```text
# Checks the application's health endpoint
$ curl -i http://localhost:8080/healthz
HTTP/1.0 200 OK
Server: BaseHTTP Python/3.15.2
Date: Thu, 29 Oct 2026 23:50:49 GMT

ok
```

### 2.2 Question 
**Explain the purpose of the /healthz and /readyz endpoints and how they differ.**

The `/healthz` endpoint checks whether the application is running, while `/readyz` checks whether it is ready to receive traffic. In this application, both return HTTP 200 because they use the same implementation.

## 3. Write and Test the Dockerfile
### 3.1 Dockerfile

I used `python:3.15` as the base image to run my Python application.

[View Dockerfile](./Dockerfile)

### 3.2 Build the Docker Image

![Docker Build](<screenshot/3 docker build.png>)

```text
# Builds the Docker image for the AMD64 architecture
$ docker build --platform linux/amd64 -t ypan69/a2-app:0.2 -f Dockerfile .
[+] Building 1.6s (9/9) FINISHED
...
=> [3/3] COPY apps/python/app.py /app/app.py
=> naming to docker.io/ypan69/a2-app:0.2
```

### 3.3 Run and Test the Container

![Docker Container](<screenshot/3 docker run.png>)

```text
# Runs the Docker container and maps port 8080
$ docker run -d --name a2-app -p 8080:8080 ypan69/a2-app:0.2
9c0a88843ab8bc765209a429084ca1f592a7a169a9dd9e1416cdeff4918d4093
```

```text
# Tests the application running inside the Docker container
$ curl http://localhost:8080
Hello world from the CLO835 class!
```

### 3.4 Questions

**Identify the base image you selected and explain why it is appropriate.**

I selected `python:3.15` because it includes the Python runtime required to run my application.

**Explain why the image must target linux/amd64 for this assignment.**

The Kubernetes nodes in this assignment use the AMD64 architecture, so the image must be compatible with `linux/amd64`.

**Explain what an exec format error means and identify a potential cause.**

An exec format error means the system cannot execute a file. One possible cause is running an ARM64 image on an AMD64 machine.

## 4. Configure the GitHub Actions Pipeline
### 4.1 GitHub Actions Workflow

I configured GitHub Actions to automatically build and push the Docker image to Docker Hub when changes are pushed to the main branch.

[View Docker Workflow](./.github/workflows/docker.yml)

### 4.2 GitHub Secrets

I created two repository secrets: `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`.

![GitHub Secrets](<screenshot/4 git secrets.png>)

### 4.3 GitHub Actions and Docker Hub

I committed and pushed the Dockerfile and GitHub Actions workflow to the main branch.

![Git Push](<screenshot/4 git update.png>)

The workflow ran successfully after I pushed the changes to GitHub.

![GitHub Actions](<screenshot/4 git action.png>)

The Docker image was published with two tags: `0.2` and the corresponding commit SHA.

Docker Hub Image: https://hub.docker.com/r/ypan69/a2-app

![Docker Hub Tags](<screenshot/4 docker tag.png>)

### 4.4 Questions

**Explain why the Docker Hub token must be stored as a secret rather than written directly into the workflow file.**

To keep the token private and prevent it from being exposed in the workflow file or repository.

**Explain what the commit SHA tag provides that a version tag alone does not.**

The commit SHA tag identifies the exact Git commit used to build the image, while a version tag can be reused or overwritten.

## 5. Create the Kubernetes Cluster

### 5.1 Terraform Deployment

I used the Week 05 Terraform configuration to create a three-node Kubernetes cluster on AWS EC2.

![Terraform Apply and EC2 Instances](<screenshot/5 ec2 instance.png>)

### 5.2 SSH to the Control Plane

I connected to the control-plane node through SSH.

![SSH Session](<screenshot/5 SSH.png>)

### 5.3 Verify Kubernetes Nodes

I checked the Kubernetes node status. All three nodes were Ready.

![SSH Session](<screenshot/5 nodes.png>)

```text
# Displays all Kubernetes nodes and confirms they are Ready
ubuntu@masternode:~$ kubectl get nodes -o wide
NAME          STATUS   ROLES           AGE     VERSION    INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
masternode    Ready    control-plane   7m27s   v1.35.10   172.31.3.124   <none>        Ubuntu 26.04.1 LTS   7.0.0-1016-aws   containerd://2.2.2
workernode1   Ready    worker          7m16s   v1.35.10   172.31.0.140   <none>        Ubuntu 26.04.1 LTS   7.0.0-1016-aws   containerd://2.2.2
workernode2   Ready    worker          7m14s   v1.35.10   172.31.0.195   <none>        Ubuntu 26.04.1 LTS   7.0.0-1016-aws   containerd://2.2.2
```

### 5.4 Question
**Explain why the Kubernetes commands in this assignment must be executed on the control-plane machine.**

We run kubectl on the control-plane node because it has the configuration needed to manage the Kubernetes cluster.

## 6. Deploy Version 0.2 to Kubernetes

### 6.1 Create and Apply Kubernetes Configuration

[View Deployment](./k8s/deployment.yaml)

[View Service](./k8s/service.yaml)

I created the Deployment and Service on the master node and applied both configurations.

![Kubernetes Apply](<screenshot/6 apply.png>)
```text
# Creates the Deployment and NodePort Service
ubuntu@masternode:~$ kubectl apply -f deployment.yaml -f service.yaml
deployment.apps/a2-app created
service/a2-app-service created
```

### 6.2 Verify Kubernetes Resources

![Kubernetes Resources](<screenshot/6 get.png>)

```text
# Displays the Deployment, ReplicaSet, Pods, and Services
ubuntu@masternode:~$ kubectl get deploy,rs,pods,svc
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/a2-app   3/3     3            3           3m10s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/a2-app-6456748c47   3         3         3       3m10s

NAME                          READY   STATUS    RESTARTS   AGE
pod/a2-app-6456748c47-hhxvr   1/1     Running   0          3m10s
pod/a2-app-6456748c47-qcr4m   1/1     Running   0          3m10s
pod/a2-app-6456748c47-zxmvb   1/1     Running   0          3m10s

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/a2-app-service   NodePort    10.108.209.28   <none>        80:30000/TCP   3m10s
service/kubernetes       ClusterIP   10.96.0.1       <none>        443/TCP        19m
```

All three Pods are running, and the Service uses NodePort 30000.

### 6.3 Pod Distribution Across Nodes

![Pod Distribution](<screenshot/6 get pods.png>)

```text
# Shows which worker node is running each Pod
ubuntu@masternode:~$ kubectl get pods -o wide
NAME                      READY   STATUS    RESTARTS   AGE     IP           NODE          NOMINATED NODE   READINESS GATES
a2-app-6456748c47-hhxvr   1/1     Running   0          5m11s   10.244.2.3   workernode2   <none>           <none>
a2-app-6456748c47-qcr4m   1/1     Running   0          5m11s   10.244.1.2   workernode1   <none>           <none>
a2-app-6456748c47-zxmvb   1/1     Running   0          5m11s   10.244.2.2   workernode2   <none>           <none>
```

### 6.4 Question
**Identify which node is running each pod.**

- a2-app-6456748c47-hhxvr — workernode2
- a2-app-6456748c47-qcr4m — workernode1
- a2-app-6456748c47-zxmvb — workernode2

**Explain why the Service selector must match the pod labels specified in the Deployment's pod template.**

Both the ReplicaSet and Service use label selectors to find matching Pods. The ReplicaSet maintains the number of Pods, while the Service forwards traffic to them.


## 7. Test NodePort Access

### 7.1 Test NodePort from Master Node

![NodePort Curl Test](<screenshot/7 curl.png>)

```text
# Tests the application through NodePort 30000
ubuntu@masternode:~$ curl http://localhost:30000
Hello world from the CLO835 class!
```

### 7.2 Access NodePort from All Three Nodes

![NodePort Access](<screenshot/7 access node.png>)

All three nodes returned the same response through NodePort 30000.

### 7.3 Question

**Explain why a node without a locally running application pod can still serve requests through the NodePort Service.**

NodePort forwards traffic to matching Pods across the cluster, even when the Pod runs on another node.

## 8. Release Version 0.3

### 8.1 Update Application and Push to GitHub

I updated the application to v0.3 and added my student ID to the message.

```python
VERSION = "0.3"
MESSAGE = "Hello world from the CLO835 class and 101753267!"
```

![Git Log](<screenshot/8 git-log.png>)

```text
# Displays the Git commit history
$ git log --oneline
14e21bd (HEAD -> main, origin/main, origin/HEAD) Update Docker image tag to v0.3
1621652 Release v0.3 and update Kubernetes configs and report
2decfc7 Update Assignment 2 report sections 1-4
4764046 Update Dockerfile and workflow
9a69483 Initial commit for CLO835 Assignment 2
```
### 8.2 GitHub Actions

After pushing the changes, GitHub Actions successfully built and published the updated Docker image.

![GitHub Actions](<screenshot/8 git action.png>)

### 8.3 Docker Hub Image

The Docker image is available on Docker Hub with the `0.3` tag.

![Docker Hub Tags](<screenshot/8 docker tag.png>)

[View Docker Hub Image](https://hub.docker.com/r/ypan69/a2-app/tags)


### 8.4 Kubernetes Rolling Update

![Rolling Update](<screenshot/8 set image.png>)

```text
# Updates the Deployment image from v0.2 to v0.3
ubuntu@masternode:~$ kubectl set image deployment/a2-app a2-app=ypan69/a2-app:0.3
deployment.apps/a2-app image updated
```

```text
# Confirms that the rolling update completed successfully
ubuntu@masternode:~$ kubectl rollout status deployment/a2-app
deployment "a2-app" successfully rolled out
```

### 8.5 Verify Rolling Update

![Rolling Update Verification](<screenshot/8 rollout.png>)

```text
# Shows that the new v0.3 ReplicaSet has 3 Pods and the old one has 0
ubuntu@masternode:~$ kubectl get rs -o wide
NAME                DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES              SELECTOR
a2-app-585658c64    3         3         3       2m42s   a2-app       ypan69/a2-app:0.3   app=a2-app,pod-template-hash=585658c64
a2-app-6456748c47   0         0         0       52m     a2-app       ypan69/a2-app:0.2   app=a2-app,pod-template-hash=6456748c47
```

```text
# Displays the Deployment revision history
ubuntu@masternode:~$ kubectl rollout history deployment/a2-app
deployment.apps/a2-app
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

```text
# Confirms that all three v0.3 Pods are running on worker nodes
ubuntu@masternode:~$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP           NODE          NOMINATED NODE   READINESS GATES
a2-app-585658c64-7pdqq   1/1     Running   0          6m11s   10.244.1.3   workernode1   <none>           <none>
a2-app-585658c64-ff7dj   1/1     Running   0          6m7s    10.244.1.4   workernode1   <none>           <none>
a2-app-585658c64-zgkcq   1/1     Running   0          6m9s    10.244.2.4   workernode2   <none>           <none>
```

The new ReplicaSet has three running Pods, while the old ReplicaSet has zero.

### 8.6 Verify Version 0.3

![Version 0.3 NodePort Access](<screenshot/8 access node.png>)

All three nodes returned the updated message with my student ID, confirming that v0.3 was successfully deployed.

### 8.7 Question

**Explain how the rolling update replaces the old pods with new pods and how the Deployment's update strategy helps maintain application availability during the rollout.**

Kubernetes gradually replaces old Pods with new ones while keeping some Pods running to maintain availability.

## 9. Roll Back to Version 0.2

### 9.1 Rollback to Version 0.2

![Kubernetes Rollback](<screenshot/9 undo.png>)

```text
# Displays the available Deployment revisions before rollback
ubuntu@masternode:~$ kubectl rollout history deployment/a2-app
deployment.apps/a2-app
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

```text
# Rolls back the Deployment from v0.3 to v0.2
ubuntu@masternode:~$ kubectl rollout undo deployment/a2-app
deployment.apps/a2-app rolled back
```

```text
# Confirms that the rollback completed successfully
ubuntu@masternode:~$ kubectl rollout status deployment/a2-app
deployment "a2-app" successfully rolled out
```

```text
# Shows v0.3 Pods terminating while v0.2 Pods are running
ubuntu@masternode:~$ kubectl get pods -o wide
NAME                      READY   STATUS        RESTARTS   AGE   IP           NODE          NOMINATED NODE   READINESS GATES
a2-app-585658c64-7pdqq    1/1     Terminating   0          12m   10.244.1.3   workernode1   <none>           <none>
a2-app-585658c64-ff7dj    1/1     Terminating   0          12m   10.244.1.4   workernode1   <none>           <none>
a2-app-585658c64-zgkcq    1/1     Terminating   0          12m   10.244.2.4   workernode2   <none>           <none>
a2-app-6456748c47-7shl7   1/1     Running       0          23s   10.244.2.6   workernode2   <none>           <none>
a2-app-6456748c47-8glsw   1/1     Running       0          24s   10.244.1.5   workernode1   <none>           <none>
a2-app-6456748c47-lmmnd   1/1     Running       0          25s   10.244.2.5   workernode2   <none>           <none>
```

The v0.3 Pods were terminating, while the three v0.2 Pods were running.

### 9.2 Verify Rollback to Version 0.2

![Rollback Verification](<screenshot/9 access node.png>)

```text
# Confirms that the v0.2 ReplicaSet has 3 Pods and v0.3 has 0
ubuntu@masternode:~$ kubectl get rs -o wide
NAME                DESIRED   CURRENT   READY   IMAGES
a2-app-585658c64    0         0         0       ypan69/a2-app:0.3
a2-app-6456748c47   3         3         3       ypan69/a2-app:0.2
```

After rollback, the v0.2 ReplicaSet manages three Pods, while the v0.3 ReplicaSet has zero Pods. All three nodes return the original message.

### 9.3 Questions

**Explain why rolling back to the previous Deployment revision does not require building and publishing a new Docker image.**

Kubernetes reuses the previous ReplicaSet and existing Docker image, so no new image needs to be built or published.

**Explain what changed after `kubectl rollout undo`. Say which ReplicaSet held the Pods before, and which one holds them after.**

Before rollback, v0.3 had 3 Pods. After rollback, v0.2 has 3 Pods, while the v0.3 ReplicaSet has 0.

## 10. Prepare Your Report

### 10.1 Proof

The following screenshots show the application running on the Kubernetes nodes before and after the update.

| Version | Address | Screenshot |
|---|---|---|
| 0.2 | `http://54.236.55.81:30000` | [View Screenshot](<screenshot/7 access node.png>) |
| 0.3 | `http://54.236.55.81:30000` | [View Screenshot](<screenshot/8 access node.png>) |


### 10.2 The problems that you met

1. **Incorrect Docker Image Tag**

    After updating the application to v0.3, I forgot to change the Docker image tag in docker.yml, so GitHub Actions overwrote the 0.2 image with v0.3. I fixed the workflow to publish 0.3, then restored the original 0.2 image using its previous commit SHA tag and pushed it back to Docker Hub.

## 11. Delete the Kubernetes Resources and Destroy the Cluster

I used Terraform to destroy the Kubernetes cluster and its AWS resources after completing the assignment.

![Terraform Destroy](<screenshot/11 terraform destroy.png>)

**Explain why stopping an EC2 instance does not necessarily eliminate all AWS charges. For example, attached EBS volumes and other billable resources may continue to incur costs.**

AWS resources such as EC2 instances, EBS volumes, and public IPv4 addresses may incur charges. Destroying unused resources helps avoid unnecessary costs.

