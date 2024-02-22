# End-to-end-DevOps | Nodejs-Postgres App

## Prerequisites

- AWS CLI
- Docker
- kubectl
- Terraform
- Helm
- GitHub_CLI
- Beekeeper-Studio

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