################Containerize & Deploy a Multi-Tier Spring Boot Application on Kubernetes#########################

Objective: Containerize the application, push to Docker Hub, and deploy the Application in Kubernetes using Docker Hub images.
 Here, we have frontend, backend, and PostgreSQL images stored in Docker Hub.
Technical Stack:
   Frontend:  Nodejs
   Backend: Java Spring boot
   Database: Postgres sql
   Version control : github
   Containerization: Docker
   Orchestration: Kubernetes
 Prerequisites:
   Install the following tool:
Java JDK 17
Docker desktop
Minikube
Kubectl
Verify the installation:
 Run the commands in the command prompt
     Java -version
     Docker version
     Minikube version
     Kubectl version – client

Create one directory and clone the code into created directory
Create directory :   
mkdir  bankingapplication
Change the Directory: 
cd bankingapplication
Clone the application using github url:
   Git clone github.com/pallamraju/sourcecode.git
For Kubernetes manifest files, create one directory inside bankingapplication directory

Create directory for manifest files
	Mkdir k8s
Project Structure:
 
Inside k8s, write the Kubernetes manifest yaml files like below
 
Build the Application in Docker:
Step1:
# Build backend
docker build -t banking-backend:v1 ./backend

# Build frontend
docker build -t banking-frontend:v1 ./frontend

# Check image sizes
docker images | grep banking
# backend should be ~250MB (JRE Alpine)
# frontend should be ~180MB (Node Alpine)

Push images to Docker Hub:
# Tag and push to Docker Hub
docker login
docker tag banking-backend:v1 YOUR_USERNAME/banking-backend:v1
docker tag banking-frontend:v1 YOUR_USERNAME/banking-frontend:v1
docker tag banking-backend:v1 YOUR_USERNAME/banking-backend:latest
docker tag banking-frontend:v1 YOUR_USERNAME/banking-frontend:latest
docker push YOUR_USERNAME/banking-backend:v1
docker push YOUR_USERNAME/banking-backend:latest
docker push YOUR_USERNAME/banking-frontend:v1
docker push YOUR_USERNAME/banking-frontend:latest

      Deploy All Kubernetes Resources in order:
# 1. Namespace
kubectl apply -f k8s/namespace.yaml

# 2. Secrets and ConfigMaps
kubectl apply -f k8s/postgres-secret.yaml
kubectl apply -f k8s/backend-configmap.yaml
kubectl apply -f k8s/frontend-configmap.yaml

# 3. Persistent storage
kubectl apply -f k8s/postgres-pvc.yaml

# 4. PostgreSQL
kubectl apply -f k8s/postgres-deployment.yaml
kubectl apply -f k8s/postgres-service.yaml
kubectl wait --for=condition=ready pod -l app=postgres -n banking-app --timeout=120s



# 5. Backend (wait longer for Spring Boot startup). 
minikube image load banking-backend:v1     
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/backend-service.yaml
kubectl wait --for=condition=ready pod -l app=backend -n banking-app --timeout=180s
Note:
As the Spring Boot application requires approximately 20–30 seconds to start, we use kubectl wait to confirm pod readiness before moving to the next deployment step.

# 6. Frontend.
minikube image load banking-frontend:v1  
kubectl apply -f k8s/frontend-deployment.yaml
kubectl apply -f k8s/frontend-service.yaml

Verify pods created or not  in our namespace:
   Kubectl get pods -n banking-app
Verify deployments created or not in our namespace
   Kubectl get deployment -n banking-app
Verify services created or not in our namespace
    Kubectl get service -n banking-app
Verify run on locally for testing, accessing the application
   Kubectl port-forward svc/backend 8080:8080 – n banking-app // backend testing
   Check in browser, localhost:8080/accurator/health //system is up and probe serve the traffic 
   Kubectl prot-forward svc/frontend 3000:3000 – n baning-app // frontend testing 
   Check in brower, localhost:3000  to access the application  
