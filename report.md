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

Command:

```bash
git log --oneline
```

Output:

```text
4764046 (HEAD -> main, origin/main, origin/HEAD) Update Dockerfile and workflow
9a69483 Initial commit for CLO835 Assignment 2
```

### 1.3 Question
**Explain why the workflow does not run as required when the assignment files remain one directory level below the repository root.**

GitHub Actions only detects workflows in .github/workflows at the repository root, not inside nested folders.


## 2. Select a Programming Language and Implement Version 0.2

I selected Python and used the provided application starter.

The `MESSAGE` constant in `apps/python/app.py` is:

```python
MESSAGE = "Hello world from the CLO835 class!"
```

### 2.1 Run the Application Locally and Health Check

![Local Application](<screenshot/2 local host.png>)

Command:
```bash
python app.py
```

Output:

```text
CLO835 app v0.2 listening on :8080
```

Command:
```bash
curl http://localhost:8080
```

Output:

```text
Hello world from the CLO835 class!
```

Command:
```bash
curl -i http://localhost:8080/healthz
```

Output:

```text
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

Command:

```bash
docker build --platform linux/amd64 -t ypan69/a2-app:0.2 -f Dockerfile .
```

Output:

```text
[+] Building 1.6s (9/9) FINISHED
...
=> [3/3] COPY apps/python/app.py /app/app.py
=> naming to docker.io/ypan69/a2-app:0.2
```

### 3.3 Run and Test the Container

![Docker Container](<screenshot/3 docker run.png>)

Command:

```bash
docker run -d --name a2-app -p 8080:8080 ypan69/a2-app:0.2
```

Output:

```text
9c0a88843ab8bc765209a429084ca1f592a7a169a9dd9e1416cdeff4918d4093
```

Test the application:

```bash
curl http://localhost:8080
```

Output:

```text
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

### 5.2 SSH to controal plane

I connected to my control plane thourgh SSH session.

![SSH Session](<screenshot/5 SSH.png>)

### 5.2 Verify Kubernetes Nodes

Show the kubernets Nodes status using the command below.

![SSH Session](<screenshot/5 nodes.png>)

Command:

```bash
kubectl get nodes -o wide
```

Output:

```text
NAME          STATUS   ROLES           AGE     VERSION    INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
masternode    Ready    control-plane   7m27s   v1.35.10   172.31.3.124   <none>        Ubuntu 26.04.1 LTS   7.0.0-1016-aws   containerd://2.2.2
workernode1   Ready    worker          7m16s   v1.35.10   172.31.0.140   <none>        Ubuntu 26.04.1 LTS   7.0.0-1016-aws   containerd://2.2.2
workernode2   Ready    worker          7m14s   v1.35.10   172.31.0.195   <none>        Ubuntu 26.04.1 LTS   7.0.0-1016-aws   containerd://2.2.2
```

### 5.3 Questions
**Explain why the Kubernetes commands in this assignment must be executed on the control-plane machine.**

We run kubectl on the control-plane node because it has the configuration needed to manage the Kubernetes cluster.

## 6. Deploy Version 0.2 to Kubernetes

### 6.1 Create and Apply Kubernetes Configuration

[View Deployment](./k8s/deployment.yaml)

[View Service](./k8s/service.yaml)

I created the Deployment and Service on the master node and applied both configurations.

![Kubernetes Apply](<screenshot/6 apply.png>)
Command:

```bash
kubectl apply -f deployment.yaml -f service.yaml
```

Output:

```text
deployment.apps/a2-app created
service/a2-app-service created
```

### 6.2 Verify Kubernetes Resources


![Kubernetes Resources](<screenshot/6 get.png>)

Command:

```bash
kubectl get deploy,rs,pods,svc
```

Output:

```text
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

Command:

```bash
kubectl get pods -o wide
```

Output:

```text
NAME                      READY   STATUS    RESTARTS   AGE     IP           NODE          NOMINATED NODE   READINESS GATES
a2-app-6456748c47-hhxvr   1/1     Running   0          5m11s   10.244.2.3   workernode2   <none>           <none>
a2-app-6456748c47-qcr4m   1/1     Running   0          5m11s   10.244.1.2   workernode1   <none>           <none>
a2-app-6456748c47-zxmvb   1/1     Running   0          5m11s   10.244.2.2   workernode2   <none>           <none>
```

### Question
**Identify which node is running each pod.**

- a2-app-6456748c47-hhxvr — workernode2
- a2-app-6456748c47-qcr4m — workernode1
- a2-app-6456748c47-zxmvb — workernode2

**Explain why the Service selector must match the pod labels specified in the Deployment's pod template.**

Both the ReplicaSet and Service use label selectors to find matching Pods. The ReplicaSet maintains the number of Pods, while the Service forwards traffic to them.


## 7. Test NodePort Access

### 7.1 Test NodePort from Master Node

![NodePort Curl Test](<screenshot/7 curl.png>)

### 7.2 Access NodePort from All Three Nodes

![NodePort Access](<screenshot/7 access node.png>)

All three nodes returned the same response through NodePort 30000.

### 7.3 Question

**Explain why a node without a locally running application pod can still serve requests through the NodePort Service.**

NodePort forwards traffic to matching Pods across the cluster, even when the Pod runs on another node.

## 8. Release Version 0.3

<!-- TODO: Update MESSAGE with student ID and commit/push -->
<!-- TODO: Add screenshot of successful GitHub Actions run -->
<!-- TODO: Add Docker Hub screenshot showing the 0.3 tag -->
<!-- TODO: Add kubectl set image and rollout status commands/outputs -->
<!-- TODO: Add ReplicaSet outputs before and after rollout -->
<!-- TODO: Add rollout history, kubectl get rs, and kubectl get pods -o wide outputs -->
<!-- TODO: Add browser screenshot showing version 0.3 -->
<!-- TODO: Explain how rolling updates work -->

Give the output of these three commands, after you released version 0.3.

```
kubectl rollout history deployment/<name>
kubectl get rs
kubectl get pods -o wide
```

## 9. Roll Back to Version 0.2

<!-- TODO: Add rollout history and rollout undo commands/outputs -->
<!-- TODO: Add rollout status and ReplicaSet outputs -->
<!-- TODO: Identify which ReplicaSet held Pods before and after rollback -->
<!-- TODO: Add browser screenshot showing version 0.2 again -->
<!-- TODO: Explain why rollback does not require rebuilding the image -->

Explain what changed after `kubectl rollout undo`. Say which ReplicaSet held
the Pods before, and which one holds them after.

## 10. Prepare Your Report

### 10.1 Proof

<!-- TODO: Add browser screenshots and public IP addresses for versions 0.2 and 0.3 -->

Add a screenshot of the browser, on a node public address, for both versions.

| Version | Address | Screenshot |
|---|---|---|
| 0.2 | `http://<ip>:30000` | |
| 0.3 | `http://<ip>:30000` | |

### 10.2 The problems that you met
<!-- TODO: Describe Problem 1 ~3 and how it was solved -->
List each problem, then how you solved it. One short paragraph for each.

1.
2.
3.

## 11. Delete the Kubernetes Resources and Destroy the Cluster
<!-- TODO: Add kubectl delete commands and outputs -->
<!-- TODO: Add terraform destroy command and screenshot of Destroy complete! -->
<!-- TODO: Explain why AWS resources must be destroyed to avoid charges -->
