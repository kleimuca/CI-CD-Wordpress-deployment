
# 🚀 Jenkins Installation on K3s Kubernetes Cluster (via Helm)

This guide walks you through installing Jenkins inside a K3s cluster using Helm.

---

## 📦 1. Create Jenkins Namespace

```bash
kubectl create namespace jenkins
```

---

## 📥 2. Add the Jenkins Helm Repository

```bash
helm repo add jenkins https://charts.jenkins.io
helm repo update
```

---

## ⚙️ 3. Create `jenkins-values.yaml`

Save this configuration in a file named `jenkins-values.yaml`:

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
  storageClass: "standard"  # Make sure this matches your K3s storage class
```

---

## 🚀 4. Install Jenkins Using Helm

```bash
helm install jenkins jenkins/jenkins -n jenkins -f jenkins-values.yaml
```

---

## 🌐 5. Access Jenkins (Port Forwarding)

```bash
kubectl port-forward -n jenkins svc/jenkins 8080:8080
```

Visit: [http://localhost:8080](http://localhost:8080)

---

## 🔑 6. Jenkins Login

- **Username**: `admin`
- **Password**: `admin123` (or whatever you set in the values file)

---

## 🔐 7. Configure Credentials

In Jenkins UI:

- Navigate to: `Manage Jenkins → Credentials`
- Add a new **Secret Text** for kubeconfig:
  - ID: `kubeconfig-secret`
  - Value: contents of your `kubeconfig.yaml`
- Add GitHub or Docker credentials if needed

---

You're now ready to build CI/CD pipelines that interact with your K3s workloads using Jenkins!
