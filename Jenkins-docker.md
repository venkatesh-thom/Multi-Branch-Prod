# Jenkins Kubernetes & Docker Setup

## Overview

Jenkins runs as a **separate Linux user (`jenkins`)**, not as `root`.  
To allow Jenkins pipelines to:
- Deploy applications to Kubernetes
- Run `kubectl` and `helm`
- Build and push Docker images

we must configure **kubeconfig** and **Docker access** for the Jenkins user.  

Without these steps, Jenkins jobs will fail with **permission denied** errors when accessing Kubernetes or Docker.

---

## 1. Verify Jenkins Process

Check that Jenkins is running:

```bash
ps aux | grep jenkins
```

Example output:

```text
jenkins 20425 1.8 10.3 4867872 840552 ? Ssl 06:27 0:52 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=8080
root 37820 0.0 0.0 7076 2072 pts/1 S+ 07:13 0:00 grep --color=auto jenkins
```

**Explanation:**  
- Jenkins runs as user `jenkins`.  
- Pipeline jobs execute as this user, not `root`.  
- Jenkins cannot access root-owned files like `/root/.kube/config`.

---

## 2. Why Update kubeconfig

Kubernetes commands (`kubectl`) require a configuration file (`kubeconfig`) located at:

```text
$HOME/.kube/config
```

- For root → `/root/.kube/config`  
- For Jenkins → `/home/jenkins/.kube/config`  

**Reason:** Jenkins cannot read root’s kubeconfig, so we must copy it to the Jenkins home directory and give proper permissions.

---

## 3. Create kubeconfig Directory for Jenkins

```bash
sudo mkdir -p /home/jenkins/.kube
```

**Explanation:**  
- `kubectl` looks for `$HOME/.kube/config` by default.  
- Jenkins home directory is `/home/jenkins`.  
- Ensures the directory exists before copying the kubeconfig.

---

## 4. Copy kubeconfig from root to Jenkins

```bash
sudo cp /root/.kube/config /home/jenkins/.kube/config
```

**Explanation:**  
- Root already has valid Kubernetes credentials.  
- Copying allows Jenkins to use the same cluster access without reconfiguring.  

⚠️ In production, using root kubeconfig is **not recommended**. Prefer:
- Kubernetes ServiceAccounts with RBAC  
- Cloud IAM roles (EKS/GKE/AKS)

---

## 5. Change Ownership of kubeconfig

```bash
sudo chown -R jenkins:jenkins /home/jenkins/.kube
```

**Explanation:**  
- Without correct ownership, Jenkins will get `permission denied` errors when running `kubectl`.  
- Ensures the configuration is only accessible by the Jenkins user.

---

## 6. Add Jenkins User to Docker Group

```bash
sudo usermod -aG docker jenkins
```

**Explanation:**  
- Jenkins pipelines often build Docker images or run containers.  
- Adding Jenkins to the Docker group allows it to access the Docker daemon **without sudo**.  
- Without this, jobs fail with:  
  ```
  permission denied while trying to connect to the Docker daemon
  ```

⚠️ Group changes only take effect after Jenkins restarts.

---

## 7. Restart Jenkins

```bash
sudo systemctl restart jenkins
```

Or reboot the server:

```bash
sudo reboot
```

**Explanation:**  
- New group membership (`docker`) requires a restart for Jenkins to apply.  
- Ensures Docker commands in pipelines work correctly.

---

## 8. Verify Everything Works

- Check Kubernetes access:

```bash
sudo -u jenkins kubectl get nodes
```

- Check Docker access:

```bash
sudo -u jenkins docker ps
```

Both commands should succeed **without errors**. If either fails, double-check ownership and group membership.

---

## 9. Copy-Paste Ready Commands

```bash
# Create kubeconfig directory for Jenkins
sudo mkdir -p /home/jenkins/.kube

# Copy kubeconfig from root
sudo cp /root/.kube/config /home/jenkins/.kube/config

# Set ownership
sudo chown -R jenkins:jenkins /home/jenkins/.kube

# Add Jenkins to Docker group
sudo usermod -aG docker jenkins

# Restart Jenkins
sudo systemctl restart jenkins

# Verify Jenkins Kubernetes access
sudo -u jenkins kubectl get nodes

# Verify Jenkins Docker access
sudo -u jenkins docker ps
```

---

## Summary

1. Jenkins runs as its own user → cannot use root files.  
2. Copy root kubeconfig to `/home/jenkins/.kube/config`.  
3. Set correct ownership (`jenkins:jenkins`).  
4. Add Jenkins to the Docker group.  
5. Restart Jenkins.  

After this, Jenkins pipelines can:
- Deploy to Kubernetes  
- Build and push Docker images  
- Run Helm charts  

---

## Best Practices

- Avoid using root kubeconfig in production.  
- Use Kubernetes ServiceAccounts with RBAC.  
- Use cloud IAM roles for cluster access (EKS/GKE/AKS).  
- Store sensitive kubeconfig or credentials in Jenkins Credentials.
