# Coworking Space Service Extension
The Coworking Space Service comprises APIs allowing users to request one-time tokens and administrators to grant access to a coworking space. This service adheres to a microservice architecture, with APIs divided into separate services for independent deployment and management.

## Getting Started

### Dependencies
#### Local Environment
1. Python Environment - run Python 3.6+ applications and install Python dependencies via `pip`
2. Docker CLI - build and run Docker images locally
3. `kubectl` - run commands against a Kubernetes cluster
4. `helm` - apply Helm Charts to a Kubernetes cluster

#### Remote Resources
1. AWS CodeBuild - build Docker images remotely
2. AWS ECR - host Docker images
3. Kubernetes Environment with AWS EKS - run applications in k8s
4. AWS CloudWatch - monitor activity and logs in EKS
5. GitHub - pull and clone code

### Setup
#### 1. Configure a Database
Set up a Postgres database using a Helm Chart.

1. Set up Bitnami Repo
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

2. Install PostgreSQL Helm Chart
```
helm install postgresql bitnami/postgresql
```

This should set up a Postgre deployment at `postgresql.default.svc.cluster.local` in your Kubernetes cluster. You can verify it by running `kubectl svc`

By default, it will create a username `postgres`. The password can be retrieved with the following command:
```bash
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

echo $POSTGRES_PASSWORD
```

3. Test Database Connection

* Connecting Via Port Forwarding
```bash
kubectl port-forward --namespace default svc/postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432
```

4. Run Seed Files
We will need to run the seed files in `db/` in order to create the tables and populate them with data.

```bash
kubectl port-forward --namespace default svc/<SERVICE_NAME>-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < 1_create_tables.sql
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < 2_seed_users.sql
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < 3_seed_tokens.sql
```

### 2. Running the Analytics Application Locally
In the `analytics/` directory:

1. Install dependencies
```bash
apt update
apt install build-essential libpq-dev
pip install --upgrade pip setuptools wheel
pip install -r requirements.txt
```
2. Run the application (see below regarding environment variables)
```bash
python app.py
```

There are multiple ways to set environment variables in a command. They can be set per session by running `export KEY=VAL` in the command line or they can be prepended into your command.

If we set the environment variables by exporting them, it would look like the following:
```bash
export DB_USERNAME=myuser
export DB_PASSWORD=${POSTGRES_PASSWORD}
export DB_HOST=127.0.0.1
export DB_PORT=5433
export DB_NAME=mydatabase
```

3. Verifying The Application
* Generate report for check-ins grouped by dates
`curl 127.0.0.1:5153/api/reports/daily_usage`

* Generate report for check-ins grouped by users
`curl 127.0.0.1:5153/api/reports/user_visits`

In this case, since the code is run directly on the local computer rather than through Docker, the BASE_URL is 127.0.0.1:5153. The port 5153 is specified in the app.py script.

## 3: Docker Image Creation and Local Deployment

*   Build the Docker image locally:

```bash 
docker build -t coworking .
```

## 4: AWS CodeBuild Pipeline

*  create an Amazon ECR repository on your AWS console.

*  create an Amazon CodeBuild project that is connected to your project's GitHub repository.

## 5: Kubernetes Deployment
* Deploy the application using the **coworking.yml** file, **configmap.yml** file, and **db-secret.yml** file.

## 6: CloudWatch
* Attach the CloudWatchAgentServerPolicy IAM policy to your worker nodes:
```bash
aws iam attach-role-policy \
--role-name eksctl-my-cluster-nodegroup-my-nod-NodeInstanceRole-GNNOOqrxHfGy  \
--policy-arn arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy 
```
* Use AWS CLI to install the Amazon CloudWatch Observability EKS add-on:
```bash 
aws eks create-addon --addon-name amazon-cloudwatch-observability --cluster-name my-cluster-name
```
* Trigger logging by accessing your application.

* Open up CloudWatch Log groups page. You should see aws/containerinsights/my-cluster-name/application there.

* Click on one of the log streams to see the logs.

### Deliverables
1. `Dockerfile`
2. Screenshot of AWS CodeBuild pipeline
3. Screenshot of AWS ECR repository for the application's repository
4. Screenshot of `kubectl get svc`
5. Screenshot of `kubectl get pods`
6. Screenshot of `kubectl describe svc postgresql`
7. Screenshot of `kubectl describe deployment coworking`
8. All Kubernetes config files used for deployment (ie YAML files)
9. Screenshot of AWS CloudWatch logs for the application
10. `README.md` 
