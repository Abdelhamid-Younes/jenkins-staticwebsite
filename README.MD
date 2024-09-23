# Jenkins Pipeline for Dockerized Application
This Jenkins pipeline builds, tests, and deploys a Dockerized application. It uses a multistage process to automate building a Docker image, running tests, and deploying the app to staging and production environments.

## Prerequisites

### Local Machine Setup:
- **Git**: Ensure that Git is installed on your local machine.
- **Ngrok**: To make the Jenkins server accessible publicly for GitHub webhook integration, use Ngrok to create a tunnel and expose Jenkins over the internet with a public IP address.

### Jenkins Setup:

#### Environment Variables:
Ensure you have set the required environment variables in Jenkins:
- `PARAM_IMAGE_NAME`: The name of the Docker image (e.g., `staticwebsite`).
- `PARAM_APP_NAME`: The name of your app (e.g., `younesabdh`).
- `PARAM_IMAGE_TAG`: The tag for your Docker image (e.g., `latest`).
- `PARAM_DOCKERHUB_ID`: Your DockerHub username.
- `PARAM_PORT_EXPOSED`: The port to expose the app (default is `80`).
- `PARAM_INTERNAL_PORT`: The internal port of the application (default is `5000`).

#### Secrets:
- DockerHub credentials should be securely stored in Jenkins' credentials store. Ensure you have a credential with the ID `dockerhub` containing your DockerHub password.

#### Plugins:
Install the following Jenkins plugins:
- Docker Pipeline
- GitHub Integration
- Slack Notification (optional for alerts)

### GitHub Setup:
1. Create a GitHub repository to host your application's code.
2. Set up a GitHub **Webhook** pointing to your Jenkins server, allowing automated builds to be triggered by push events to the repository.
