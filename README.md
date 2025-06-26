# How to Use These Kubernetes Files

These files provide a basic Kubernetes setup for a Node.js application connecting to a PostgreSQL database. Follow the steps below to deploy your application.

## Prerequisites:

1. Kubernetes Cluster: A running Kubernetes cluster (e.g., Minikube, Docker Desktop Kubernetes, GKE, EKS, AKS).

2. kubectl: The Kubernetes command-line tool configured to connect to your cluster.

3. Docker Image: A Docker image of your Node.js application. Make sure your Node.js application is configured to read database connection details from environment variables (e.g., DB_HOST, DB_NAME, DB_USER, DB_PASSWORD, DB_PORT).

## Step-by-Step Deployment:

1. Update 01-postgres-secret.yaml:
    - Crucial: Before applying, you must replace eW91cl9zZWN1cmVfcGFzc3dvcmQ= in 01-postgres-secret.yaml with the base64 encoded version of your actual, strong PostgreSQL password.
    - To do this, open your terminal and run: `echo -n "your_actual_strong_password" | base64`
    - Copy the output and paste it as the value for POSTGRES_PASSWORD in the 01-postgres-secret.yaml file.

2. Update 06-node-app-deployment.yaml:
    - Replace your-docker-registry/your-node-app-image:1.0.0 with the actual path to your Node.js application's Docker image.

3. Apply Kubernetes Resources in Order:

    - It's important to apply these files in a specific order to ensure dependencies are met (e.g., Secret and PVC before the database deployment).

    ``` 
        # Apply the PostgreSQL Secret
        kubectl apply -f 01-postgres-secret.yaml

        # Apply the PostgreSQL Persistent Volume Claim
        kubectl apply -f 02-postgres-pvc.yaml

        # Apply the PostgreSQL Deployment
        kubectl apply -f 03-postgres-deployment.yaml

        # Apply the PostgreSQL Service
        kubectl apply -f 04-postgres-service.yaml

        # Apply the Node.js Application ConfigMap
        kubectl apply -f 05-node-app-configmap.yaml

        # Apply the Node.js Application Deployment
        kubectl apply -f 06-node-app-deployment.yaml

        # Apply the Node.js Application Service
        kubectl apply -f 07-node-app-service.yaml
    ```

4. Verify Deployment:

    - Check the status of your pods:
        `kubectl get pods`
        You should see pods for postgres-deployment and node-app-deployment in a Running state.

5. Check the status of your services:
        `kubectl get services`
        Look for postgres-service (ClusterIP) and node-app-service (LoadBalancer). The node-app-service will show an EXTERNAL-IP once your cloud provider provisions it.

6. Access Your Application:
    - Once the node-app-service has an EXTERNAL-IP (if you're on a cloud provider with LoadBalancer support), you can access your Node.js application using that IP address and port 80.
    - If you're using minikube, you can get the service URL by running:
        `minikube service node-app-service --url`

## Important Considerations:

1. Database Migrations: This setup does not include a solution for running database migrations. For production environments, consider using a separate init container or a CI/CD pipeline to manage migrations.

2. Production Readiness: This is a basic setup. For a production environment, you would need to consider:
    - High availability for PostgreSQL (e.g., using a StatefulSet with multiple replicas, a highly available PostgreSQL operator).
    - Resource limits and requests for CPU/Memory.
    - Health checks (liveness and readiness probes).
    - Ingress for advanced routing and SSL termination.
    - Monitoring and logging.
    - Proper CI/CD integration for automated deployments.

3. Security: Ensure your POSTGRES_PASSWORD is truly secure and managed appropriately.

4. Node.js App Environment: Your Node.js application must be designed to pick up database credentials and other configurations from environment variables.

Good luck with your deployment!