# DevOps k3s Assessment – Synthlane
**Candidate:** Rakesh Chary  
**Role:** DevOps Engineer Intern  
**Environment:** Hetzner Cloud VM (Ubuntu 24)

## 1. VM & Cluster Setup

Docker Installation
docker version
docker ps
Docker Engine was installed and verified. The Docker daemon is running and accessible by the user.

k3s Installation
kubectl get nodes
kubectl get pods -A
k3s was installed as a single-node Kubernetes cluster.
This setup is suitable for an early-stage startup because it is lightweight, fast to deploy,
cost-efficient, and Kubernetes-compliant. However, it is not ideal for high availability or
large-scale production workloads.

# 2. Helm Deployment (Open WebUI)
Add Helm Repository
helm repo add open-webui https://helm.openwebui.com/
helm repo update
Dry Run
helm install webui open-webui/open-webui \
  -n openwebui \
  --create-namespace \
  --dry-run
Install Application
helm install webui open-webui/open-webui \
  -n openwebui \
  --create-namespace
Validation
kubectl get all -n openwebui

# 3. OIDC Authentication Configuration
values-oidc.yaml

oidc:
  clientId: "test"
  clientSecret: ""
  issuer: "https://invalid-issuer.example.com"
  scopes:
    - openid
    - profile
    - email
   
Apply Configuration
helm upgrade webui open-webui/open-webui \
  -n openwebui \
  -f values-oidc.yaml

# 4. Debugging (Intentional Failure)
Issue Observed
After enabling OIDC, the application failed to become accessible.

Diagnosis Steps
kubectl get pods -n openwebui
kubectl logs open-webui-0 -n openwebui
kubectl exec -n openwebui open-webui-0 -- env | grep OAUTH
Root Cause
The OIDC issuer URL was invalid
The OIDC provider used a self-signed or untrusted certificate
The application failed during OIDC metadata discovery

Fix
Verified OIDC environment variables
Used a valid issuer or configured trust for a custom CA
Redeployed the Helm release
This fix works because OIDC requires successful TLS verification and metadata retrieval.

# 5. Ownership & Production Thinking
Production Readiness
Top Risks:
Single-node failure
No monitoring or alerting
No backups
Secrets exposed via environment variables
No ingress or TLS

First Fixes:
Add monitoring and alerting
Implement backups and secret management
Failure Scenario

Scenario: 10x traffic spike and node failure at 2 AM
First failure: Entire cluster goes down
Recovery: Restore node and redeploy workloads
Next-day improvements: Add HA, autoscaling, monitoring

Security & Secrets
Secrets should be managed via Kubernetes Secrets or an external secrets manager
Secrets must never be committed to Git
Rotate client secrets, tokens, and credentials regularly

Backups & Recovery
Backup persistent volumes and configuration
Daily incremental backups, weekly full backups
Test recovery by restoring into a test cluster

Cost Ownership (Hetzner)
Keep costs low using single-node k3s and right-sized instances
Avoid over-engineering early
Move away from k3s when HA or multi-node scaling is required

Extra Credit: Self-Signed OIDC Certificates
To trust a self-signed OIDC provider, mount a custom CA certificate into the container and
update the system trust store using Kubernetes ConfigMaps or Secrets. This approach is secure,
maintainable, and avoids disabling TLS verification.

Validation Outputs
kubectl get nodes
kubectl get all -n openwebui
