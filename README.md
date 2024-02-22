# End-to-end-DevOps | Nodejs-Postgres App

## Prerequisites

- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) configured with appropriate permissions
- [Docker](https://docs.docker.com/engine/install/) installed and configured
- [kubectl](https://kubernetes.io/docs/tasks/tools/) installed and configured to interact with your Kubernetes cluster
- [Terraform](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli) installed
- [Helm](https://helm.sh/docs/intro/install/) installed
- [GitHub_CLI](https://github.com/cli/cli) installed
- [Beekeeper-Studio](https://www.beekeeperstudio.io/) `For Database Access`

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
