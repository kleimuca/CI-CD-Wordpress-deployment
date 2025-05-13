# Jenkins CI/CD Pipeline Setup for WordPress Theme Deployment on K3s Cluster

This guide will walk you through the steps to successfully set up and run a Jenkins CI/CD pipeline to deploy a custom WordPress theme to a K3s Kubernetes cluster.

---

## 1. Install Jenkins

First, ensure Jenkins is installed and running in your Kubernetes environment. If you're using K3s, you can install Jenkins using Helm.

### Step-by-Step Jenkins Installation (via Helm):

1. **Create a Namespace for Jenkins**:
    ```bash
    kubectl create namespace jenkins
    ```

2. **Add the Jenkins Helm Repository**:
    ```bash
    helm repo add jenkins https://charts.jenkins.io
    helm repo update
    ```

3. **Create a Custom `jenkins-values.yaml` File**:
    Save this configuration to a file named `jenkins-values.yaml`:
    ```yaml
    controller:
      adminUser: admin
      adminPassword: admin123  # Change this securely!
      installPlugins:
        - kubernetes
        - workflow-aggregator
        - git
        - blueocean
        - credentials-binding
        - pipeline
      serviceType: ClusterIP

    persistence:
      enabled: true
      size: 8Gi
      storageClass: "standard"  # Ensure this matches your K3s storage class
    ```

4. **Install Jenkins with Helm**:
    ```bash
    helm install jenkins jenkins/jenkins -n jenkins -f jenkins-values.yaml
    ```

5. **Access Jenkins via Port Forwarding**:
    ```bash
    kubectl port-forward -n jenkins svc/jenkins 8080:8080
    ```
    Now, open [http://localhost:8080](http://localhost:8080) in your browser.

---

## 2. Install Necessary Jenkins Plugins

Make sure to install the following plugins in Jenkins:

- **GitHub Integration Plugin**: For Jenkins to interact with GitHub.
- **Kubernetes Plugin**: For Jenkins to communicate with your K3s cluster.
- **Pipeline Plugin**: To handle `Jenkinsfile` pipelines.
- **Docker Plugin**: If needed for Docker operations (though not used in this case).

Go to **Jenkins → Manage Jenkins → Manage Plugins** and install the plugins.

---

## 3. Add Jenkins Credentials

To interact with GitHub and Kubernetes, Jenkins needs to store credentials:

### 3.1 Add Kubeconfig Credential

1. Go to **Jenkins → Manage Jenkins → Manage Credentials**.
2. Add a **Secret Text** credential:
   - **ID**: `kubeconfig-secret`
   - **Secret**: Paste the contents of your `kubeconfig.yaml` file, which provides Jenkins access to your K3s cluster.

### 3.2 Add GitHub Credentials

1. Go to **Jenkins → Manage Jenkins → Manage Credentials**.
2. Add **GitHub** credentials:
   - **Username**: Your GitHub username.
   - **Password/Token**: Your Personal Access Token from GitHub.
   - **ID**: `github-credentials`

---

## 4. Create the Jenkins Pipeline Job

Now, create a pipeline job that will run the `Jenkinsfile`.

### 4.1 Create a New Pipeline Job

1. Go to **Jenkins → New Item**.
2. Select **Pipeline** and give the job a name, e.g., `wordpress-theme-cicd`.
3. Click **OK**.

### 4.2 Configure the Pipeline

1. In the **Pipeline** section, select **Pipeline script from SCM**.
2. For **SCM**, select **Git**.
3. In **Repository URL**, enter your GitHub repository URL (e.g., `https://github.com/your-org/wordpress-theme-cicd.git`).
4. For **Credentials**, choose the GitHub credentials (`github-credentials`).
5. For **Branch Specifier**, use `*/main` or `*/master` (depending on your branch).
6. In **Script Path**, specify `Jenkinsfile` (or wherever your Jenkinsfile is located).

Click **Save**.

---

## 5. Set Up GitHub Webhook

To automatically trigger Jenkins jobs on code changes in GitHub, set up a webhook.

### 5.1 Create GitHub Webhook

1. Go to your GitHub repository.
2. Navigate to **Settings → Webhooks → Add webhook**.
3. Set the **Payload URL** to your Jenkins server’s webhook URL:
   ```plaintext
   http://<your-jenkins-url>/github-webhook/
