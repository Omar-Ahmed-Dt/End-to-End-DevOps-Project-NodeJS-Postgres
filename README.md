# End-to-end-DevOps | Nodejs-Postgres App

## Prerequisites

- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) configured with appropriate permissions
- [Docker](https://docs.docker.com/engine/install/) installed and configured
- [kubectl](https://kubernetes.io/docs/tasks/tools/) installed and configured to interact with your Kubernetes cluster
- [Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) installed
- [Helm](https://helm.sh/docs/intro/install/) installed
- [GitHub_CLI](https://github.com/cli/cli) installed
- [Beekeeper-Studio](https://www.beekeeperstudio.io/) `For Database Access`

## Before you run the script

### Update Script Variables:

The `build.sh` script contains a set of variables that you need to customize according to your AWS environment and deployment requirements. Here's how you can update them:

1. Open the `build.sh` script in a text editor of your choice.

2. Update the variables at the top of the script with your specific configurations:

   ```bash
   cluster_name="YOUR_EKS_CLUSTER_NAME"
   region="YOUR_AWS_REGION"
   aws_id="YOUR_AWS_ACCOUNT_ID"
   repo_name="YOUR_REPO_NAME"
   domain="YOUR_DOMAIN.com"
   dbsecret="POSTGRES_K8S_SECRET"
   namespace="nodejs-app"
   ```

3. Save the changes to the `build.sh` script.

### Important Notes:

- `cluster_name`: This is the name of your Amazon EKS cluster.
- `region`: This is the AWS region where your resources are located.
- `aws_id`: Your AWS account ID which is required to construct the ECR image name.
- `domain`: The domain that your application will use.
- `dbsecret`: If you update this variable make sure to update it in `k8s/database.yaml` in the Postgres environment variable `name: db-password-secret`
- `namespace`: The Kubernetes namespace where your application will be deployed. If you change the namespace, ensure to also update it in your Kubernetes manifests located in the `k8s` directory.

After updating these variables, the script will use them to deploy your application, ensuring that the correct resources are targeted and that your application is accessible via your specified domain.

### Update application domain

1. Navigate to `terraform/monitoring.tf`
2. Update applications `host` with your domain
3. Do the same in `k8s/app.yaml`

```
    grafana:
      adminUser: admin
      adminPassword: admin
      enabled: true
      ingress:
        enabled: true
        ingressClassName: nginx
        annotations:
          cert-manager.io/cluster-issuer: letsencrypt-production
        hosts:
          - grafana.[YOUR_DOMAIN]
        tls:
          - secretName: grafana-tls
            hosts:
              - grafana.[YOUR_DOMAIN]
```

## How to Run

1. Clone the repository and navigate to the root directory.

2. Make the deployment script executable:

   ```bash
   chmod +x build.sh
   ```

3. Run the deployment script:

   ```bash
   ./build.sh
   ```

4. Follow the post-deployment steps printed by the script to update your DNS records with the provided Ingress URL.

## Accessing Your Services

After DNS configuration, you should be able to access the following services:

- **Nodejs App**: `http://nodeapp.[your-domain]`
- **Alertmanager**: `http://alertmanager.[your-domain]`
- **Prometheus**: `http://prometheus.[your-domain]`
- **Grafana**: `http://grafana.[your-domain]`

Replace `[your-domain]` with your actual domain name.

## CI/CD Workflows

This project is equipped with GitHub Actions workflows to automate the Continuous Integration (CI) and Continuous Deployment (CD) processes.

### Continuous Integration Workflow

The CI workflow is triggered on pushes to the `main` branch. It performs the following tasks:

- Checks out the code from the repository.
- Configures AWS credentials using secrets stored in the GitHub repository.
- Logs in to Amazon ECR.
- Builds the Docker image for the Nodejs app.
- Tags the image and pushes it to the Amazon ECR repository.

### Continuous Deployment Workflow

The CD workflow is triggered upon the successful completion of the CI workflow. It performs the following tasks:

- Checks out the code from the repository.
- Configures AWS credentials using secrets stored in the GitHub repository.
- Sets up `kubectl` with the required Kubernetes version.
- Deploys the Kubernetes manifests found in the `k8s` directory to the EKS cluster.

## GitHub Actions Secrets

The following secrets need to be set in your GitHub repository for the workflows to function correctly:

- `AWS_ACCESS_KEY_ID`: Your AWS Access Key ID.
- `AWS_SECRET_ACCESS_KEY`: Your AWS Secret Access Key.
- `KUBECONFIG_SECRET`: Your Kubernetes config file encoded in base64.

### Setting Up GitHub Secrets for AWS

Before using the GitHub Actions workflows, you need to set up the AWS credentials as secrets in your GitHub repository. The included `github_secrets.sh` script automates the process of adding your AWS credentials to GitHub Secrets, which are then used by the workflows. To use this script:

1. Ensure you have the GitHub CLI (`gh`) installed and authenticated.
2. Run the script with the following command:

   ```bash
   ./github_secrets.sh
   ```

This script will:

- Extract your AWS Access Key ID and Secret Access Key from your local AWS configuration.
- Use the GitHub CLI to set these as secrets in your GitHub repository.

**Note**: It's crucial to handle AWS credentials securely. The provided script is for demonstration purposes, and in a production environment, you should use a secure method to inject these credentials into your CI/CD pipeline.

These secrets are consumed by the GitHub Actions workflows to access your AWS resources and manage your Kubernetes cluster.

## Adding KUBECONFIG to GitHub Secrets

For the Continuous Deployment workflow to function properly, it requires access to your Kubernetes cluster. This access is granted through the `KUBECONFIG` file. You need to add this file manually to your GitHub repository's secrets to ensure secure and proper deployment.

To add your `KUBECONFIG` to GitHub Secrets, follow these steps:

1. Encode your `KUBECONFIG` file to a base64 string:

   ```bash
   cat ~/.kube/config | base64
   ```

2. Copy the encoded output to your clipboard.

3. Navigate to your GitHub repository on the web.

4. Go to `Settings` > `Secrets` > `New repository secret`.

5. Name the secret `KUBECONFIG_SECRET`.

6. Paste the base64-encoded `KUBECONFIG` data into the secret's value field.

7. Click `Add secret` to save the new secret.

This `KUBECONFIG_SECRET` is then used by the CD workflow to authenticate with your Kubernetes cluster and apply the required configurations.

**Important**: Be cautious with your `KUBECONFIG` data as it provides administrative access to your Kubernetes cluster. Only store it in secure locations, and never expose it in logs or to unauthorized users.

### Adding Data to the Database with Beekeeper

Create a table in the database and insert data.

1. Run the following query to create table.

```
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title TEXT NOT NULL,
    body TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

2. Run the following query to insert data into the table.

```
INSERT INTO posts (title, body) VALUES ('First Post', 'This is the first post.');
INSERT INTO posts (title, body) VALUES ('Second Post', 'This is the second post.');
INSERT INTO posts (title, body) VALUES ('Third Post', 'This is the third post.');
```

4. Check the application webpage to see the updates.

<img src=imgs/app.png>

After running the queries, you should be able to verify that the new entries have been added to the database and showed on the webpage.

## Destroying the Infrastructure

In case you need to tear down the infrastructure and services that you have deployed, a script named `destroy.sh` is provided in the repository. This script will:

- Log in to Amazon ECR.
- Delete the specified Docker image from the ECR repository.
- Delete the Kubernetes deployment and associated resources.
- Delete the Kubernetes namespace.
- Destroy the AWS resources created by Terraform.

### Before you run

1. Open the `destroy.sh` script.
2. Ensure that the variables at the top of the script match your AWS and Kubernetes settings:

   ```bash
   REGION="REGION"
   REPOSITORY_NAME="ECR_REPOSITORY_NAME"
   ```

### How to Run the Destroy Script

1. Save the script and make it executable:

   ```bash
   chmod +x destroy.sh
   ```

2. Run the script:

   ```bash
   ./destroy.sh
   ```

This script will execute another script `ecr-img-delete.sh` which will delete all the images on the `ECR` to make sure the `ECR` is empty then `terraform` commands to destroy all resources related to your deployment. It is essential to verify that the script has completed successfully to ensure that all resources have been cleaned up and no unexpected costs are incurred.
