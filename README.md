# 🚀 Trend Store – End-to-End DevOps CI/CD Project (Beginner Friendly)

This project demonstrates how to deploy a **React application** to a **production-ready Kubernetes (AWS EKS)** environment using **Docker, Jenkins, Terraform, GitHub, and AWS**.

⚠️ This README is written assuming **NO prior DevOps experience**.  
It includes **real mistakes I faced and how I fixed them**, so anyone can follow this end-to-end.

---

## 📌 Project Overview

**Goal:**  
Deploy a React application using a full DevOps CI/CD pipeline.

**Flow:**
GitHub → Jenkins → DockerHub → AWS EKS → LoadBalancer → Browser

---

## 🧰 Tech Stack Used

| Tool | Purpose |
|---|---|
| React (Vite) | Frontend application |
| Docker | Containerization |
| Jenkins | CI/CD pipeline |
| Terraform | Infrastructure as Code |
| AWS EC2 | Jenkins server |
| AWS EKS | Kubernetes cluster |
| DockerHub | Image registry |
| GitHub | Version control |
| Kubernetes Dashboard | Monitoring |

---

## 📂 Repository Structure
.
├── Dockerfile
├── Jenkinsfile
├── deployment.yaml
├── service.yaml
├── main.tf
├── dist/
├── .gitignore
└── README.md

---

## 🔹 STEP 1: Clone Application Repository

bash
git clone https://github.com/Vennilavan12/Trend.git
cd Trend

🔹 STEP 2: Build Production Files
npm install
npm run build

✔ This generates the dist/ folder.

❌ Common Mistake:
Forgetting to run npm run build.

✅ Fix:
Always ensure dist/ exists before Docker build.

🔹 STEP 3: Dockerize the Application

Dockerfile

FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY dist/ /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

Test Docker Image Locally

docker build -t h4meed/trend-prod .
docker run -p 3000:80 h4meed/trend-prod
✔ App should load in browser.

🔹 STEP 4: Push Image to DockerHub
docker login
docker push h4meed/trend-prod

❌ Error Faced: permission denied docker.sock
✅ Fix:sudo usermod -aG docker ubuntu
newgrp docker

🔹 STEP 5: Infrastructure Using Terraform

Terraform creates:
	•	VPC
	•	EC2 (Jenkins server)
	•	IAM roles
  terraform init
  terraform apply
❌ Mistake: Terraform state pushed to GitHub
✅ Fix: .gitignore
.terraform/
terraform.tfstate*
node_modules/
.DS_Store

🔹 STEP 6: Install Jenkins on EC2
sudo apt update
sudo apt install -y openjdk-17-jdk
sudo apt install -y jenkins
sudo systemctl start jenkins
http://<EC2-PUBLIC-IP>:8080


⸻

🔹 STEP 7: Jenkins Plugins Installed
	•	Git
	•	Docker
	•	Kubernetes
	•	Pipeline

⸻

🔹 STEP 8: Jenkins Credentials Setup

Add in Manage Jenkins → Credentials

Credential
Type
GitHub
Username + Token
DockerHub
Username + Token

❌ Mistake: Pipeline failed due to missing credentials
✅ Fix: Added credentials properly


⸻

🔹 STEP 9: AWS EKS Cluster Setup
eksctl create cluster --name trend-cluster --region ap-south-1
Verify:
aws eks list-clusters
kubectl get nodes

🔹 STEP 10: Give Jenkins Access to EKS
sudo cp /home/ubuntu/.kube/config /var/lib/jenkins/.kube/config
sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube
Verify : sudo -u jenkins kubectl get nodes

🔹 STEP 11: Kubernetes Manifests

deployment.yaml

Uses placeholder image:image: IMAGE_PLACEHOLDER
service.yaml type: LoadBalancer

🔹 STEP 12: Jenkins CI/CD Pipeline (Jenkinsfile)

pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "h4meed/trend-prod:${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Hamsab31/Trendstore-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh 'sed -i s|IMAGE_PLACEHOLDER|$DOCKER_IMAGE|g deployment.yaml'
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'
            }
        }
    }
}

🔹 STEP 13: Run Jenkins Pipeline

✔ Docker image built
✔ Image pushed to DockerHub
✔ Kubernetes deployment updated automatically

⸻

🔹 STEP 14: Verify Deployment
kubectl get pods
kubectl get svc
Access application using LoadBalancer URL:http://<ELB-DNS-NAME>

🔹 STEP 15: Monitoring – Kubernetes Dashboard 
kubectl proxy
Access locally:
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
Login using Service Account Token.

⸻

🧯 Common Errors & Fixes

| Issue                     | Fix                                      |
|---------------------------|------------------------------------------|
| Jenkins kubectl auth error | Copy kubeconfig to Jenkins               |
| Docker permission denied   | Add Jenkins to docker group              |
| Image not updating         | Use BUILD_NUMBER tag                     |
| dist folder missing        | Run `npm run build`                      |
| Jenkins deploy not working | Fix aws-auth & kubeconfig                |


⸻

📸 Screenshots to Attach (Submission)
	1.	Jenkins pipeline success
	2.	DockerHub image pushed
	3.	EKS nodes running
	4.	kubectl get pods & svc
	5.	LoadBalancer URL in browser
	6.	Kubernetes Dashboard
	7.	Terraform apply success

⸻

🎯 Final Outcome

✔ CI/CD fully automated
✔ Production-ready deployment
✔ Versioned Docker images
✔ Kubernetes-based scaling
✔ Monitoring enabled

⸻

🏁 Conclusion

This project demonstrates real-world DevOps practices with real issues and fixes.
Even a complete beginner can reproduce this project using this README.

⸻






