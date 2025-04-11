
# 🚀 Deploying Demo App on AWS EKS with Docker & ECR

This guide walks you through containerizing a demo app, pushing it to AWS ECR, and deploying it on an Amazon EKS cluster.

---

## 🐳 Docker Setup

### 1. Build the Docker Image
```bash
docker build -t demo-app .
```

### 2. Run the Image Locally (Optional)
```bash
docker run -d -p 8080:8080 demo-app:latest
```

---

## 🔐 AWS Configuration

### 3. Configure AWS CLI
```bash
aws configure
```
Fill in:
- Access Key ID
- Secret Access Key
- Region: `us-east-1`

> Optional: Set session token  
```bash
aws configure set aws_session_token "<Your session token>"
```

---

## 📦 AWS ECR (Elastic Container Registry)

### 4. Authenticate Docker with ECR
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <your-account-id>.dkr.ecr.us-east-1.amazonaws.com
```

### 5. Create an ECR Repository
```bash
aws ecr create-repository --repository-name demo-app
```

### 6. Tag the Image for ECR
```bash
docker tag demo-app:latest <your-account-id>.dkr.ecr.us-east-1.amazonaws.com/demo-app:latest
```

### 7. Push Image to ECR
```bash
docker push <your-account-id>.dkr.ecr.us-east-1.amazonaws.com/demo-app:latest
```

---

## ☸️ Amazon EKS (Elastic Kubernetes Service)

### 8. Create EKS Cluster
```bash
eksctl create cluster --name demo-cluster --region us-east-1 --nodegroup-name demo-nodes --nodes 2
```

---

## 📄 Kubernetes Deployment

### 9. `demo-app-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo-app
  template:
    metadata:
      labels:
        app: demo-app
    spec:
      containers:
      - name: demo-app
        image: <your-account-id>.dkr.ecr.us-east-1.amazonaws.com/demo-app:latest
        ports:
        - containerPort: 8080
```

### 10. `demo-app-service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: demo-app-service
spec:
  type: LoadBalancer
  selector:
    app: demo-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### 11. Apply Kubernetes Configs
```bash
kubectl apply -f demo-app-deployment.yaml
kubectl apply -f demo-app-service.yaml
```

---

## 🌐 Access the App

Run:
```bash
kubectl get service demo-app-service
```

Your app will be accessible via the **EXTERNAL-IP** of the LoadBalancer:

🔗 Example:  
[http://a72999a730f7444ba9c0f6fe5e8aad6b-13004719.us-east-1.elb.amazonaws.com/](http://a72999a730f7444ba9c0f6fe5e8aad6b-13004719.us-east-1.elb.amazonaws.com/)

---

## ✅ To Do Next: Secure via HTTPS

- Use AWS ACM for SSL Certificates
- Set up Ingress + ALB Controller
- Use a domain like `myapp.example.com`

---

## 📌 Notes

- Replace `<your-account-id>` with your AWS Account ID.
- Make sure EKS nodes have proper IAM roles to pull from ECR.

---
