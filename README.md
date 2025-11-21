# Flask Monitor GitLab CI/CD Demo

This is a Python Flask web application that demonstrates GitLab CI/CD pipeline implementation with AWS CloudFormation deployment. The app provides system information and real-time monitoring screen with dials showing CPU, memory, IO and process information.

The app has been designed with cloud native demos & containers in mind, in order to provide a real working application for deployment, something more than "hello-world" but with the minimum of pre-reqs. It is not intended as a complete example of a fully functioning architecture or complex software design.

Typical uses would be deployment to Kubernetes, demos of Docker, GitLab CI/CD (build pipelines are provided), deployment to cloud (AWS) monitoring, auto-scaling

## Screenshot

![screen](https://user-images.githubusercontent.com/14982936/30533171-db17fccc-9c4f-11e7-8862-eb8c148fedea.png)

## Building & Running Locally

### Pre-reqs

- Be using Linux, WSL or MacOS, with bash, make etc
- [Python 3.8+](https://www.python.org/downloads/) - for running locally, linting, running tests etc
- [Docker](https://docs.docker.com/get-docker/) - for running as a container, or image build and push
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) - for deployment to AWS
- [GitLab account](https://gitlab.com) - for CI/CD pipeline execution

Clone the project to any directory where you do development work

```
git clone https://github.com/tkoppine/Flask-Monitor-GitLab-CI-CD-Demo.git
```

### Makefile

A standard GNU Make file is provided to help with running and building locally.

```text
help                 💬 This help message
lint                 🔎 Lint & format, will not fix but sets exit code on error
lint-fix             📜 Lint & format, will try to fix errors and modify code
image                🔨 Build container image from Dockerfile
push                 📤 Push container image to registry
run                  🏃 Run the server locally using Python & Flask
deploy               🚀 Deploy to AWS using CloudFormation
test                 🎯 Unit tests for Flask app
test-report          🎯 Unit tests for Flask app (with report output)
test-api             🚦 Run integration API tests, server must be running
clean                🧹 Clean up project
```

Make file variables and default values, pass these in when calling `make`, e.g. `make image IMAGE_REPO=your-repo/app-name`

| Makefile Variable | Default                      | Description               |
| ----------------- | ---------------------------- | ------------------------- |
| IMAGE_REG         | docker.io                    | Container registry URL    |
| IMAGE_REPO        | ${DOCKER_USERNAME}/flask-app | Docker repository name    |
| IMAGE_TAG         | latest                       | Container image tag       |
| AWS_REGION        | us-east-2                    | AWS deployment region     |
| STACK_NAME        | FlaskAppStack                | CloudFormation stack name |

The app runs under Flask and listens on port 5000 by default, this can be changed with the `PORT` environmental variable.

# Containers

Container images can be built and pushed to your preferred registry.

Run in a container with:

```bash
docker run --rm -it -p 5000:5000 ${IMAGE_REG}/${IMAGE_REPO}:${IMAGE_TAG}
```

Should you want to build your own container, use `make image` and the above variables to customise the name & tag.

## Kubernetes

The app can easily be deployed to Kubernetes using Helm, see [deploy/kubernetes/readme.md](deploy/kubernetes/readme.md) for details

# GitLab CI/CD

A working GitLab CI/CD pipeline is provided in `.gitlab-ci.yml`, automated builds and deployments are run in GitLab hosted runners.

### Pipeline Configuration

The pipeline includes the following stages:

- **Test** (commented): Run unit tests
- **Build** (commented): Build and push Docker image
- **Deploy**: Deploy to AWS using CloudFormation

### Required GitLab CI/CD Variables

Configure these variables in your GitLab project settings (`Settings > CI/CD > Variables`):

| Variable Name           | Description                  | Required |
| ----------------------- | ---------------------------- | -------- |
| `REGISTRY_USER`         | Docker registry username     | Yes      |
| `REGISTRY_PASS`         | Docker registry password     | Yes      |
| `AWS_ACCESS_KEY_ID`     | AWS access key               | Yes      |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key               | Yes      |
| `AWS_DEFAULT_REGION`    | AWS region (e.g., us-east-2) | Yes      |

## Deployment to AWS

The application deploys to AWS using CloudFormation. The CloudFormation template is located in `aws/cf_create_stack.json`.

### Manual Deployment

Ensure you have AWS CLI configured with appropriate credentials:

```bash
aws configure
```

Deploy using the CloudFormation template:

```bash
aws cloudformation deploy \
  --template-file aws/cf_create_stack.json \
  --stack-name ${STACK_NAME} \
  --region ${AWS_REGION} \
  --capabilities CAPABILITY_IAM
```

### GitLab CI/CD Deployment

The deployment happens automatically when code is pushed to the `main` branch. The pipeline will:

1. Delete existing stack (if any)
2. Wait for deletion to complete
3. Deploy new stack using CloudFormation
