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
**Why does the workflow not run as required when the assignment files remain one directory level below the repository root?**

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

**Why did you select this base image?**

I selected `python:3.15` because it includes the Python runtime required to run my application.

**Why must the image target linux/amd64?**

The Kubernetes nodes in this assignment use the AMD64 architecture, so the image must be compatible with `linux/amd64`.

**What does an exec format error mean?**

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

**Why must the Docker Hub token be stored as a secret?**

To keep the token private and prevent it from being exposed in the workflow file or repository.

**What does the commit SHA tag provide?**

The commit SHA tag identifies the exact Git commit used to build the image, while a version tag can be reused or overwritten.

## 5. Create the Kubernetes Cluster

<!-- TODO: Add Terraform init/apply commands and outputs -->
<!-- TODO: Add screenshot of three Ready Kubernetes nodes -->
<!-- TODO: Explain why kubectl runs on the control-plane node -->


## 6. Deploy Version 0.2 to Kubernetes

<!-- TODO: Add links to deployment.yaml and service.yaml -->
<!-- TODO: Add kubectl apply commands and outputs -->
<!-- TODO: Add deployment, ReplicaSet, Pod, and Service outputs -->
<!-- TODO: Add kubectl get pods -o wide output -->
<!-- TODO: Explain how the Service label selector works -->


## 7. Test NodePort Access

<!-- TODO: Add curl http://localhost:30000 output -->
<!-- TODO: Add browser screenshot from control-plane public IP:30000 -->
<!-- TODO: Add browser screenshot from worker1 public IP:30000 -->
<!-- TODO: Add browser screenshot from worker2 public IP:30000 -->
<!-- TODO: Explain why NodePort works on nodes without a local Pod -->


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
