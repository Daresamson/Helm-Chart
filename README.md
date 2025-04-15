## **Project 5: Kubernetes Deployment and Helm Chart for Web Application**
![Screenshot (219)](https://github.com/user-attachments/assets/5f067a89-e84f-4902-b178-c5cf0204c07d)

![Screenshot (227)](https://github.com/user-attachments/assets/f8d172e7-01f1-4ebd-a767-0a0dcbd42f57)


### **1. Create Kubernetes Manifests**

**Objective:** Write Kubernetes manifests for deployment, service, and ingress to deploy your web application.

1. **Create Deployment Manifest**:
   - Define the deployment in `deployment.yaml` to specify the Docker image, replicas, and other deployment configurations for the web app.
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: web-app
     labels:
       app: web-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: web-app
     template:
       metadata:
         labels:
           app: web-app
       spec:
         containers:
         - name: web-app
           image: yourdockerimage:latest
           ports:
           - containerPort: 80
   ```

![Screenshot (228)](https://github.com/user-attachments/assets/22dd5c17-b88f-4271-9fa6-dbe881fec8ac)

2. **Create Service Manifest**:
   - Define the service to expose the web app in `service.yaml`.
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: web-app-service
   spec:
     selector:
       app: web-app
     ports:
       - protocol: TCP
         port: 80
         targetPort: 80
     type: LoadBalancer
   ```

![Screenshot (229)](https://github.com/user-attachments/assets/fa694400-5a41-45ff-a4b7-615ef9be02f5)

3. **Create Ingress Manifest**:
   - Define ingress rules in `ingress.yaml` to manage external HTTP(S) access via NGINX Ingress Controller.
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: Ingress
   metadata:
     name: web-app-ingress
   spec:
     rules:
     - host: web-app.example.com
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: web-app-service
               port:
                 number: 80
   ```

---

![Screenshot (230)](https://github.com/user-attachments/assets/0b9cc1e9-43cd-4dca-99ed-dc051b78ceb8)
![Screenshot (231)](https://github.com/user-attachments/assets/576fdc48-a6a2-4b05-9d48-f15b9987959d)
![Screenshot (233)](https://github.com/user-attachments/assets/f2c0469f-ec86-4a26-be60-034d8b7901f2)


### **2. Convert Manifests to Helm Chart**

**Objective:** Convert the Kubernetes manifests into a Helm chart for easier management and deployment.
![Screenshot (237)](https://github.com/user-attachments/assets/5ff348e5-aec5-4cec-9046-ff244b944c77)

1. **Create a Helm Chart**:
   - Initialize a new Helm chart in the `web-app` directory using the following command:
   ```bash
   helm create web-app
   ```
   This will generate the structure for the Helm chart, including `Chart.yaml`, `values.yaml`, and the `templates` folder.

![Screenshot (247)](https://github.com/user-attachments/assets/1921e2c3-842b-4e6c-9d1c-f791fe42f6b1)

2. **Modify `Chart.yaml`**:
   - In the `Chart.yaml`, set the application name, description, and version:
   ```yaml
   apiVersion: v2
   name: web-app
   description: A Helm chart for deploying the web app
   version: 0.1.0
   ```

3. **Modify Templates**:
   - Replace the default templates in the `templates/` directory with your custom Kubernetes manifests (`deployment.yaml`, `service.yaml`, `ingress.yaml`) from step 1.
![Screenshot (234)](https://github.com/user-attachments/assets/e51c8a13-5954-40b4-bf1e-5ea0284c6d44)
![Screenshot (235)](https://github.com/user-attachments/assets/ed00340a-d172-4110-a094-dfbbbcad06e0)
![Screenshot (224)](https://github.com/user-attachments/assets/f94dd76e-43c4-47c0-808e-9bab4d5df08e)

4. **Configure `values.yaml`**:
![Screenshot (248)](https://github.com/user-attachments/assets/7a1dbd39-f528-444c-b415-2bf7c033bede)

    - Add customizable values to `values.yaml` so you can manage configurations like Docker image, replicas, and more.
   ```yaml
   replicaCount: 2
   image:
     repository: yourdockerimage
     pullPolicy: IfNotPresent
     tag: latest
   ingress:
     enabled: true
     hostname: web-app.example.com
   ```

![Screenshot (225)](https://github.com/user-attachments/assets/ce2f7d46-15e1-4f44-8bf4-2adf17aa2163)
![Screenshot (233)](https://github.com/user-attachments/assets/e421a4df-04cf-4158-b20d-62c30d736d55)


5. **Test the Helm Chart**:
   - Run the following command to test the Helm chart locally with Minikube:
   ```bash
   helm install web-app ./web-app --namespace default
   ```

---

![Screenshot (236)](https://github.com/user-attachments/assets/0ea9aaa8-0e90-41cb-8cbb-99456188c59f)

### **3. Set Up CI/CD Pipeline**

**Objective:** Configure a CI/CD pipeline to automate the deployment of the Helm chart to Kubernetes, including integrations with tools like Cert-Manager, Nginx Ingress Controller, Vault, and Cluster Issuer.

1. **Create GitHub Personal Access Token**:
   - Generate a GitHub personal access token (PAT) with the required permissions (`repo`, `workflow`) to authenticate for CI/CD actions:
   - Navigate to GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Generate new token**.
   - Add this token as a secret (`GITHUB_TOKEN`) in your GitHub repository.

![Screenshot (238)](https://github.com/user-attachments/assets/f6c44b09-436a-4f54-a375-1ea1b646afd5)

2. **Create GitHub Actions Workflow**:
   - Create a `.github/workflows/deploy.yml` file in your repository to define the CI/CD pipeline.

```yaml
name: Deploy Web App

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: self-hosted  # 🧠 This tells GitHub to use the self-hosted runner

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Helm
        run: |
          curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
          helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
          helm repo update

      - name: Deploy to Kubernetes (Minikube)
        run: |
          helm upgrade --install web-app ./web-app --namespace default
```
![Screenshot (239)](https://github.com/user-attachments/assets/b459f497-bf7f-4968-be10-3c22f643ab58)

3. **Push Code to GitHub**:
   - After creating or updating your Helm chart and GitHub Actions workflow file, push the changes to the `main` branch:
   ```bash
   git add .
   git commit -m "Add CI/CD pipeline and Helm chart for web-app"
   git push origin main
   ```
![Screenshot (246)](https://github.com/user-attachments/assets/f7b766cf-87bc-464a-9e13-76299fb59ab6)
![Screenshot (242)](https://github.com/user-attachments/assets/db758922-a5d6-4f62-aa35-100daabecc9e)

4. **GitHub Actions CI/CD Workflow Execution**:
   - Upon pushing to the `main` branch, the GitHub Actions pipeline will trigger.
   - The workflow will:
     1. Checkout the code from GitHub.
     2. Install Helm.
     3. Deploy the application using the Helm chart to the Kubernetes cluster.
![Screenshot (241)](https://github.com/user-attachments/assets/9076eddd-c53f-4f47-b5e6-7f27912e4c77)
![Screenshot (240)](https://github.com/user-attachments/assets/91a3052a-28e0-4c52-a924-95c742a2476b)

5. **Secure Deployment with Vault and Cert-Manager**:
   - If required, set up **Vault** and **Cert-Manager** to manage secrets and certificates.
   - You can configure the `values.yaml` to point to a Vault secret backend and use Cert-Manager for automatic certificate management.

---

### **4. Troubleshooting and Iterations**

During the process, you encountered several issues and made adjustments to improve the workflow:

- **Helm Chart Path Issues**:
  - At one point,  I received errors like `Error: path "./web-app" not found`. This was due to incorrect paths or working directories.
  - I resolved this by adjusting the Helm chart path and ensuring that the directory structure was correct.

- **Helm Deployment Issues**:
  - There were issues with `helm upgrade --install` commands not being recognized. These were resolved by checking the correct file paths and ensuring that the Helm chart was properly configured and structured.

- **CI/CD Token Errors**:
  - Tokens were generated and stored securely in GitHub secrets to allow seamless integration with GitHub Actions and Kubernetes.

---

### **5. Final Steps**

1. **Verify Deployment**:
   - After successful deployment, verify that your application is accessible by running the following commands:
   ```bash
   kubectl get pods --namespace default
   kubectl get svc --namespace default
   kubectl get ingress --namespace default
   ```
![Screenshot (245)](https://github.com/user-attachments/assets/32b1168d-bb80-4c3b-a42b-bf95066f88dc)

2. **Access the Web App**:
   - Use `kubectl port-forward` to access your web app locally or configure DNS for your ingress.

---
![Screenshot (244)](https://github.com/user-attachments/assets/b510167f-6aa8-4f85-92c3-36b0a89909f8)

### **Conclusion**

By following this step-by-step process, I have successfully deployed a web application to Kubernetes using Helm, set up a CI/CD pipeline with GitHub Actions, and integrated security tools like Vault and Cert-Manager for secure management. This process demonstrated key aspects of Kubernetes, Helm, CI/CD, and automated deployments, giving me hands-on experience in deploying and managing modern applications in Kubernetes.

