# CI/CD Pipeline Assignment – Flask Application

## 1. Project Overview

This project implements an end-to-end CI/CD pipeline for a Python Flask application using **Jenkins**, **Docker**, **Amazon ECR**, and **Amazon EC2**.

The pipeline automatically:

1. Pulls the latest code from GitHub.
2. Installs Python dependencies.
3. Runs the pytest test suite.
4. Builds a Docker image.
5. Tags the Docker image using the Git commit SHA.
6. Authenticates with Amazon ECR.
7. Pushes the image to ECR.
8. Connects to EC2 using SSH.
9. Pulls the new image from ECR.
10. Stops and removes the existing container.
11. Starts the new container.
12. Runs a deployment health check.
13. Sends a success/failure email.
14. Automatically starts when code is pushed to the `main` branch.

The assignment specifically requires the test stage to gate build/deployment, commit-SHA image tagging, ECR push, EC2 deployment, deployment verification, automatic `main`-branch triggering, customized notifications, secrets management, and documentation. fileciteturn28file0L46-L113

---

## 2. Architecture

```text
Developer
   |
   | git push
   v
GitHub - main branch
   |
   | GitHub Webhook
   v
ngrok
   |
   v
Jenkins
   |
   +--> Checkout Source
   |
   +--> Install Dependencies
   |
   +--> Pytest
   |
   +--> Docker Build
   |       |
   |       +--> Git Commit SHA tag
   |
   +--> ECR Login
   |
   +--> Docker Push
   |       |
   |       v
   |     Amazon ECR
   |
   +--> SSH to EC2
           |
           +--> Docker Login to ECR
           +--> Docker Pull
           +--> Stop old container
           +--> Remove old container
           +--> Run new container
           +--> /health verification
                    |
                    v
                Email
              SUCCESS / FAILURE
```

# 3. Technologies Used

| Technology | Purpose |
|---|---|
| GitHub | Source-code repository |
| Jenkins | CI/CD automation |
| Python / Flask | Application |
| pytest | Automated testing |
| Docker | Application containerization |
| Amazon ECR | Docker image registry |
| Amazon EC2 | Application deployment target |
| AWS IAM | Authentication and permissions |
| SSH | Jenkins-to-EC2 deployment connection |
| ngrok | Public tunnel for GitHub → local Jenkins webhook |
| Gmail SMTP | Jenkins email notification |

---

# 4. Application

The application is a Python Flask web application.

The repository contains:

```text
.
├── application files
├── requirements.txt
├── Dockerfile
├── pytest test suite
├── Jenkinsfile
└── README.md
```

The application exposes a health endpoint used by Jenkins after deployment:

```text
/health
```

The health endpoint is important because a Docker container can start and still fail to serve the application correctly. The assignment requires the deployment verification step to act as a gate. fileciteturn28file0L74-L80

<img width="1916" height="761" alt="image" src="https://github.com/user-attachments/assets/9ccbef6c-2b1e-420e-831a-03e55168673c" />

---

# 5. GitHub Repository

Repository:

```text
https://github.com/Ansarisjd/flask_Practice_CI-CD_Pipeline_Assignment
```

The GitHub repository contains the Flask application, tests, Dockerfile, Jenkinsfile, and README.

The assignment requires the GitHub repository to contain the application, Dockerfile, pytest suite, pipeline definition, and README. fileciteturn28file0L116-L120

---

# 6. AWS Infrastructure Setup

## 6.1 Amazon ECR

An Amazon ECR repository was created to store the Docker images.

Repository:

```text
student-registration-system-registry
```

Region:

```text
us-east-1
```

ECR registry:

```text
251523190381.dkr.ecr.us-east-1.amazonaws.com
```

The assignment requires an ECR repository for storing the built images. fileciteturn28file0L54-L55

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/038d8a65-e057-4d37-90ff-ca2ca9070ec0" />

---

## 6.2 EC2 Instance

An Ubuntu EC2 instance was configured as the deployment server.

The EC2 server has:

- Docker installed
- Docker service available
- IAM permissions for pulling images from ECR
- SSH access enabled
- Application port `5000` available as required
- Environment configuration stored on the EC2 instance

The assignment allows either SSH or SSM for deployment. SSH was selected for this project. fileciteturn28file0L56-L64


<img width="1518" height="465" alt="image" src="https://github.com/user-attachments/assets/1052ba79-f099-40b8-a85e-c295098b2258" />


---

# 7. IAM Configuration

AWS credentials were configured in Jenkins rather than hardcoding AWS access credentials into the Jenkinsfile.

Jenkins uses the configured AWS credential:

```text
aws-ecr
```

The credential is used during the ECR login stage.

The EC2 instance also uses AWS permissions required to pull the Docker image from ECR.

The assignment explicitly requires sensitive AWS credentials and SSH credentials to be stored in the CI tool's credential store rather than committed to GitHub. fileciteturn28file0L104-L107

<img width="1889" height="917" alt="image" src="https://github.com/user-attachments/assets/aeaae57d-64bd-4b9a-a8c2-8d25588389c8" />

---

# 8. Jenkins Setup

Jenkins was installed and configured on a Windows machine.

Jenkins was accessed locally through:

```text
http://localhost:8080
```

The Jenkins job was configured as a Pipeline job using the Jenkinsfile stored in GitHub.

The Jenkins job uses the GitHub repository as its SCM source.

Jenkins Declarative Pipeline automatically performs the SCM checkout, which was visible in the Jenkins console output as:

```text
Declarative: Checkout SCM
```

The pipeline successfully checked out the `main` branch during execution. fileciteturn28file1L160-L165

### Screenshot

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/cfa9412c-4072-43a2-bdbf-a5718907c558" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/21a9a1b6-7934-49b8-980f-450f5e7e139e" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/6664ccd2-a51d-491f-b840-f7dda9260382" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d16045ed-0c24-4b7c-9183-b335922adf15" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c3f671b7-c4f4-42ac-934b-c7d6275dc7c9" />

---

# 9. Jenkins Credentials

The following credentials were configured in Jenkins.

## 9.1 MongoDB URI

Credential ID:

```text
MONGO_URI
```

Type:

```text
Secret Text
```

Used by the test stage.

Example usage:

```groovy
withCredentials([
    string(
        credentialsId: 'MONGO_URI',
        variable: 'MONGO_URI'
    )
]) {
    bat 'python -m pytest'
}
```

---

## 9.2 AWS ECR Credentials

Credential ID:

```text
aws-ecr
```

Used to authenticate Jenkins with Amazon ECR.

---

## 9.3 EC2 SSH Key

Credential ID:

```text
ec2-jenkins-ssh
```

The EC2 private key was stored in Jenkins rather than committed to GitHub.

The original EC2 key was available as a `.ppk` file. It was converted to an OpenSSH-compatible `.pem` key using PuTTY Key Generator and then stored in Jenkins.

### Screenshot

<img width="1380" height="449" alt="image" src="https://github.com/user-attachments/assets/3053303d-b438-4638-b6c1-0a0e27adf397" />


---

# 10. Jenkins Pipeline Stages

The final pipeline contains the following major stages.

## Stage 1 – Checkout

Jenkins automatically checks out the latest source code from GitHub.

The assignment requires the pipeline to begin by pulling the latest source code. fileciteturn28file0L65-L70

---

## Stage 2 – Install Dependencies

The pipeline runs:

```bat
python -m pip install -r requirements.txt
```

This installs the application's Python dependencies.

---

## Stage 3 – Test

The pipeline runs:

```bat
python -m pytest
```

The MongoDB URI is injected securely using Jenkins credentials.

The test stage must pass before Docker build and deployment can continue.

In one successful run, the test suite executed:

```text
4 passed
```

The assignment specifically requires the pipeline to stop if tests fail. fileciteturn28file0L68-L70


<img width="1122" height="404" alt="image" src="https://github.com/user-attachments/assets/76c39e6f-4428-4f8e-a456-07878107a0d6" />


---

# 11. Docker Build

The application is packaged into a Docker image.

The important improvement made during the assignment was moving away from only using:

```text
latest
```

and using the Git commit SHA as the image tag.

Example:

```text
student-registration-app:<GIT_COMMIT>
```

This makes every deployed image traceable to a specific Git commit.

The assignment explicitly requires commit-SHA image tagging. fileciteturn28file0L71-L72

### Screenshot

<img width="1362" height="554" alt="image" src="https://github.com/user-attachments/assets/bba9abfe-93a2-4e81-b396-5112b0dfcac3" />

---

# 12. ECR Login

Jenkins authenticates with ECR using:

```bat
aws ecr get-login-password --region us-east-1 |
docker login --username AWS --password-stdin \
251523190381.dkr.ecr.us-east-1.amazonaws.com
```

The AWS credentials are supplied through Jenkins credentials.

The pipeline does not store AWS secret values directly in the Jenkinsfile.

---

# 13. Docker Tag

The local image is tagged using the Git commit SHA:

```text
251523190381.dkr.ecr.us-east-1.amazonaws.com/
student-registration-system-registry:<GIT_COMMIT>
```

This provides traceability between:

```text
Git Commit
     ↓
Docker Image
     ↓
ECR
     ↓
EC2 Deployment
```

<img width="1480" height="775" alt="image" src="https://github.com/user-attachments/assets/a78df227-c8d2-42fc-ac41-72cc2be75e5a" />

---

# 14. Docker Push

The image is pushed to Amazon ECR:

```bat
docker push 251523190381.dkr.ecr.us-east-1.amazonaws.com/student-registration-system-registry:%GIT_COMMIT%
```

The assignment requires the tagged image to be pushed to ECR. fileciteturn28file0L71-L73

<img width="1480" height="775" alt="image" src="https://github.com/user-attachments/assets/e3cb8300-b828-4737-aa42-ea8bd0fc404d" />

---

# 15. Deployment to EC2

The deployment uses SSH.

Jenkins retrieves the SSH key from its credential store and connects to the Ubuntu EC2 instance.

The deployment process is:

```text
Jenkins
   |
   | SSH
   v
EC2
   |
   +--> Login to ECR
   |
   +--> Pull new Docker image
   |
   +--> Stop old container
   |
   +--> Remove old container
   |
   +--> Start new container
```

The container is started using:

```text
-p 5000:5000
```

and the EC2 environment file:

```text
/home/ubuntu/student-registration.env
```

The assignment specifically requires the deployment to pull the new image, stop/remove the existing container, run the new container, and map the application port. fileciteturn28file0L74-L78

<img width="1362" height="637" alt="image" src="https://github.com/user-attachments/assets/691ef9c4-b898-41d1-86d7-0f484e60173d" />

---

# 16. Deployment Verification

After deployment, Jenkins connects to EC2 and executes:

```bash
curl -f http://localhost:5000/health
```

The `-f` option makes curl return a failure status for HTTP errors.

Therefore, if the application is not healthy, the Jenkins pipeline fails instead of reporting a false successful deployment.

This satisfies the assignment's deploy-verification gate requirement. fileciteturn28file0L78-L80

<img width="1331" height="686" alt="image" src="https://github.com/user-attachments/assets/8482d155-b758-4277-8957-05371a9b8cd3" />


---

# 17. Email Notification

Email notification was implemented using Jenkins' native `mail` step.

The pipeline sends different messages for:

```text
SUCCESS
FAILURE
```

Example success subject:

```text
SUCCESS: Flask-CICD-Pipeline #50
```

The successful email confirms that the application was deployed to EC2 and includes the Git commit SHA and Jenkins build URL.

<img width="1899" height="928" alt="image" src="https://github.com/user-attachments/assets/de94b60b-0d5c-4763-8ea4-f0e299f801c7" />

---

# 18. Automatic GitHub Trigger

Initially, the pipeline was started manually using Jenkins **Build Now**.

The pipeline was then configured for automatic execution whenever code is pushed to the `main` branch.

The setup used:

```text
GitHub
   ↓
Webhook
   ↓
ngrok
   ↓
Jenkins
```

Because Jenkins was running locally, ngrok was used to expose Jenkins to GitHub.

The ngrok tunnel forwards:

```text
https://<ngrok-url>
        ↓
http://localhost:8080
```

The GitHub webhook uses:

```text
https://<ngrok-url>/github-webhook/
```

The Jenkins job was configured with:

```text
GitHub hook trigger for GITScm polling
```

The assignment requires automatic triggering on every push to `main`. fileciteturn28file0L84-L86

---

# 19. End-to-End Successful Execution

A complete pipeline execution successfully performed:

```text
GitHub Push
    ↓
Automatic Jenkins Trigger
    ↓
Checkout
    ↓
Install Dependencies
    ↓
Pytest
    ↓
Docker Build
    ↓
Commit SHA Tag
    ↓
ECR Login
    ↓
ECR Push
    ↓
SSH to EC2
    ↓
Docker Pull
    ↓
Stop Existing Container
    ↓
Remove Existing Container
    ↓
Run New Container
    ↓
Health Check
    ↓
Success Email
```

A successful build email was received for:

```text
Flask-CICD-Pipeline #50
```

<img width="1367" height="771" alt="image" src="https://github.com/user-attachments/assets/12171293-e056-4171-9602-339689bc3150" />


---

# 20. Manual Deployment Procedure

If Jenkins is unavailable, the deployment can be reproduced manually from a machine with AWS CLI, Docker, and SSH configured.

## Login to ECR

```bash
aws ecr get-login-password --region us-east-1 |
docker login --username AWS --password-stdin \
251523190381.dkr.ecr.us-east-1.amazonaws.com
```

## Pull the image

```bash
docker pull \
251523190381.dkr.ecr.us-east-1.amazonaws.com/student-registration-system-registry:<GIT_COMMIT>
```

## Stop existing container

```bash
docker stop student-registration-app
```

## Remove existing container

```bash
docker rm student-registration-app
```

## Start new container

```bash
docker run -d \
  --name student-registration-app \
  --env-file /home/ubuntu/student-registration.env \
  -p 5000:5000 \
  251523190381.dkr.ecr.us-east-1.amazonaws.com/student-registration-system-registry:<GIT_COMMIT>
```

## Verify

```bash
curl -f http://localhost:5000/health
```

---

# 21. Secrets Management

Sensitive values were kept outside the Git repository.

The following were stored/configured through Jenkins credentials:

```text
MONGO_URI
AWS credentials
EC2 SSH private key
```

The SSH private key was never committed to GitHub.

The assignment explicitly prohibits committing sensitive credentials to the repository. fileciteturn28file0L104-L107

---

# 22. Problems Encountered and How They Were Resolved

This assignment involved several real troubleshooting scenarios.

## 22.1 `.ppk` vs `.pem` SSH key

### Problem

The EC2 key was originally available as a PuTTY `.ppk` file.

Using it directly with OpenSSH caused authentication problems.

### Resolution

The key was loaded into PuTTY Key Generator and exported as an OpenSSH-compatible private key.

The converted key was then stored in Jenkins credentials.

---

## 22.2 Incorrect SSH command

An early SSH test used an incorrect identity-file path:

```text
ssh -i "Documents" ...
```

This resulted in:

```text
Identity file Documents not accessible
```

### Resolution

The correct private-key file path was supplied.

---

## 22.3 SSH `Permission denied (publickey)`

Initially EC2 returned:

```text
Permission denied (publickey)
```

### Resolution

The SSH key, username, key format, and EC2 configuration were checked.

A direct SSH test was eventually successful:

```text
ubuntu@ip-10-0-1-41:~$
```

This confirmed that the SSH credentials were valid before integrating them into Jenkins.

---

## 22.4 Jenkins SSH credential configuration problems

There was confusion between different Jenkins credential bindings.

The deployment eventually used the Jenkins credential as a file:

```groovy
file(
    credentialsId: 'ec2-jenkins-ssh',
    variable: 'SSH_KEY'
)
```

This allowed the Jenkins agent to use the private key file during the SSH commands.

---

## 22.5 Jenkinsfile brace/structure error

At one point the `post` block was outside the correct Declarative Pipeline structure.

Jenkins reported:

```text
Expected a stage @ line 98
```

### Resolution

The pipeline structure was corrected so that:

```text
pipeline
 ├── agent
 ├── stages
 │    └── stage
 └── post
```

was correctly nested.

---

## 22.6 Gmail SMTP authentication failure

Jenkins initially returned:

```text
530-5.7.0 Authentication Required
```

The SMTP authentication configuration was incomplete.

### Resolution

SMTP authentication and Gmail security settings were reviewed and corrected until Jenkins was able to send the test email successfully.

The working solution used Jenkins' email notification capability rather than hardcoding Gmail credentials in the Jenkinsfile.

---

## 22.7 Attempt to use Email Extension unnecessarily

The Email Extension / Extended Email configuration introduced additional configuration complexity.

The pipeline ultimately used the simpler native Jenkins `mail` step.

This was sufficient for sending customized success/failure messages.

---

## 22.8 Health endpoint returned 404

The first deployment verification attempt used:

```bash
curl -f http://localhost:5000/health
```

but the application returned:

```text
404
```

This correctly caused the deployment verification to fail.

### Resolution

The Flask application's health endpoint was corrected.

After the correction:

```bash
curl http://localhost:5000/health
```

worked successfully.

This demonstrated why a health-check gate is important.

---

## 22.9 Docker image initially used `latest`

The first version of the pipeline used:

```text
student-registration-app:latest
```

and pushed:

```text
.../student-registration-system-registry:latest
```

This did not satisfy the assignment's requirement for commit traceability.

### Resolution

The pipeline was changed to use:

```text
$GIT_COMMIT
```

as the Docker image tag.

Example:

```text
student-registration-app:<commit-sha>
```

and:

```text
student-registration-system-registry:<commit-sha>
```

---

## 22.10 Confusion about manual Build Now vs automatic trigger

Initially, builds were manually started using:

```text
Build Now
```

### Resolution

A GitHub webhook was configured so that pushes to `main` automatically trigger Jenkins.

The final workflow no longer requires manually clicking **Build Now** for normal deployments.

---

## 22.11 ngrok authentication failure

When ngrok was first started, it returned:

```text
ERR_NGROK_4018
```

because the ngrok session was not authenticated.

### Resolution

An ngrok account was configured and the authtoken was added.

ngrok then successfully showed:

```text
Session Status    online
Forwarding       https://<ngrok-url> -> http://localhost:8080
```

---

## 22.12 Jenkins initially not publicly reachable

Because Jenkins was running locally on Windows:

```text
http://localhost:8080
```

GitHub could not directly access it.

### Resolution

ngrok was used to expose Jenkins temporarily through a public HTTPS URL.

---

# 23. Important Lessons Learned

### CI/CD

A CI/CD pipeline is not simply "build a Docker image."

A complete pipeline should establish a controlled flow:

```text
Code
 → Test
 → Build
 → Registry
 → Deployment
 → Verification
 → Notification
```

### Commit SHA tagging

Using the Git commit SHA makes deployments traceable.

For example:

```text
Commit:
5095429a124d6bf3b659e4f3f89caed29e5bfb1f

        ↓

Docker:
.../student-registration-system-registry:
5095429a124d6bf3b659e4f3f89caed29e5bfb1f

        ↓

EC2:
Running container from that exact image
```

### Deployment verification

A successful `docker run` command does not necessarily mean the application is healthy.

The health check provides the final verification gate.

### Secrets

Credentials should never be placed directly in:

```text
Jenkinsfile
GitHub repository
Dockerfile
```

They should be stored securely in Jenkins credentials or another secrets-management system.

---

# 24. Assignment Compliance Checklist

| Requirement | Status |
|---|---|
| Flask application | ✅ |
| `requirements.txt` | ✅ |
| pytest test suite | ✅ |
| Dockerfile | ✅ |
| ECR repository | ✅ |
| EC2 deployment target | ✅ |
| Docker installed on EC2 | ✅ |
| EC2 ECR permissions | ✅ |
| Jenkins pipeline | ✅ |
| Checkout | ✅ |
| Install dependencies | ✅ |
| Pytest gate | ✅ |
| Docker build | ✅ |
| Git commit SHA tagging | ✅ |
| ECR push | ✅ |
| SSH deployment | ✅ |
| Stop/remove existing container | ✅ |
| Run new container | ✅ |
| `/health` verification | ✅ |
| Automatic `main` push trigger | ✅ |
| Success email | ✅ |
| Failure email implementation | ⚠️ Verify with intentionally broken run |
| Failure email identifies failed stage | ⚠️ Should be improved/verified |
| Success email includes branch | ⚠️ Add/verify |
| Success email includes Docker image tag | ⚠️ Add/verify |
| Success email includes EC2 target | ⚠️ Add/verify |
| README | ✅ |
| Screenshots | ⏳ Add before submission |
| Intentionally broken pipeline screenshot | ⏳ Required before submission |

The assignment explicitly requires the success email to include the commit SHA, branch, Docker image tag, EC2 target, and pipeline link. It also requires a failure email identifying the failed stage, commit SHA, and pipeline/log link. fileciteturn28file0L87-L103

---


The assignment specifically asks for a successful full pipeline, success email, and an intentionally broken run demonstrating early termination and failure notification. fileciteturn28file0L121-L126

---

# 26. Final Result

The project demonstrates an end-to-end Jenkins CI/CD implementation for a Flask application:

```text
GitHub
   ↓
Automatic Webhook
   ↓
Jenkins
   ↓
Automated Tests
   ↓
Docker Build
   ↓
Commit SHA Image
   ↓
Amazon ECR
   ↓
SSH
   ↓
Amazon EC2
   ↓
Docker Deployment
   ↓
Health Check
   ↓
Email Notification
```

The pipeline successfully automates application delivery from a GitHub push through testing, containerization, registry publishing, EC2 deployment, verification, and notification.

---

## Repository

```text
https://github.com/Ansarisjd/flask_Practice_CI-CD_Pipeline_Assignment
```

---

## Submission Note


