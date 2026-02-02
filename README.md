# aws-tf-cicd-platform
**An AWS-native, Terraform-driven CI/CD platform for deploying containerized applications using Jenkins, Docker, and Ansible**

---


## 1. Repository Structure

```text
aws-tf-cicd-platform/
├── infra/                     # Terraform (infrastructure only)
│   ├── modules/
│   │   ├── ec2/
│   │   ├── security-group/
│   │   ├── iam/
│   │   └── ecr/
│   ├── jenkins-master.tf
│   ├── jenkins-worker.tf
│   ├── app-server.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── backend.tf
│
├── ansible/                   # Ansible (configuration only)
│   ├── inventories/
│   │   ├── dev/
│   │   └── prod/
│   ├── roles/
│   │   ├── common/
│   │   ├── docker/
│   │   ├── docker-compose/
│   │   ├── jenkins-master/
│   │   ├── jenkins-worker/
│   │   ├── sonarqube/
│   │   └── app-runtime/
│   └── playbooks/
│       ├── jenkins-master.yml
│       ├── jenkins-worker.yml
│       └── app-server.yml
│
├── jenkins/
│   ├── Jenkinsfile             # Platform pipeline
│   └── job-dsl/                # Optional: seed jobs
│
├── jenkins-shared-libs/
│   ├── vars/
│   │   ├── cloneApp.groovy
│   │   ├── buildImage.groovy
│   │   ├── sonarScan.groovy
│   │   ├── pushDockerHub.groovy
│   │   ├── pushECR.groovy
│   │   └── deployContainer.groovy
│   ├── src/com/platform/
│   │   └── PipelineUtils.groovy
│   └── resources/
│
└── README.md

```

**Separation of concerns is strict:**

* **Terraform** creates machines.
* **Ansible** prepares machines.
* **Jenkins** delivers software.

---

## 2. Infrastructure Layer (Terraform)

### Responsibilities

* Jenkins Master & Worker EC2 Instances
* Application EC2 Instances
* Security Groups & IAM Roles/Policies
* ECR Repositories

### Rules

* Terraform **never** installs software.
* Terraform **never** deploys applications.
* Terraform outputs only metadata (IPs, ARNs) to feed into Ansible.

---

## 3. Configuration Layer (Ansible)

### Target Nodes

* Jenkins Master / Worker
* Application Server

### Responsibilities

* OS hardening & base packages
* Docker & Docker Compose installation
* Jenkins installation & Worker registration
* SonarQube installation (containerized)
* App runtime preparation

### Execution Order

1. **Common role:** All nodes
2. **Jenkins Master:** Orchestration setup
3. **Jenkins Worker:** Build environment
4. **Application Server:** Deployment environment

---

## 4. Jenkins Architecture

| Component | Role | Key Tools |
| --- | --- | --- |
| **Jenkins Master** | UI & Orchestration | Credentials, Plugins, Shared Libs |
| **Jenkins Worker** | Build Engine | Docker, SonarQube, Build Tools |
| **App Server** | Runtime | Docker, Docker Compose |

> [!IMPORTANT]
> Builds **never** run on the Master. The App Server **never** builds images; it only runs them.

---

## 5. Application Integration Contract

Each deployment job is parameter-driven to keep the platform app-agnostic.

### Parameters

* **Required:** `APP_GIT_URL`, `APP_GIT_BRANCH`, `APP_NAME`, `DEPLOY_ENV`
* **Optional:** `DOCKERFILE_PATH`, `COMPOSE_FILE_PATH`, `APP_PORT`

---

## 6. Jenkins Shared Library

The shared library acts as the **Platform API**. It abstracts complex logic away from the end-user.

* **Logic:** Clones repos, detects build types, handles ECR/DockerHub auth, and manages remote deployment via SSH/Docker.
* **Maintenance:** Changes to logic are made in the library, automatically updating all pipelines.

---

---

## 8. End-to-End Execution Flow

1. **Provision:** Terraform creates the AWS infrastructure.
2. **Configure:** Ansible installs Docker and configures Jenkins.
3. **Initialize:** Jenkins loads the Shared Library.
4. **Trigger:** A deployment job is triggered (Manual/Webhook).
5. **Build:** The Worker clones the app, builds the image, and runs SonarQube.
6. **Store:** The image is pushed to AWS ECR.
7. **Run:** The App Server pulls the latest image and restarts the container.

---

**Final Note:** This is a **delivery platform**. Treat it like infrastructure to ensure scalability and prevent technical decay.

