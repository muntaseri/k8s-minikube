# Task 1: Cluster & Namespace


## Minikube Setup & Cluster Configuration Guide

Run Kubernetes locally on macOS with Minikube.

---

## Prerequisites

- **OS:** macOS (Intel or Apple Silicon)
- **Resources:** Minimum 2 CPUs, 2 GB free RAM, 20 GB free disk space
- **Driver:** Container or VM manager installed and running (e.g., [Docker Desktop](https://www.docker.com/products/docker-desktop/), HyperKit, or QEMU)
- **Package Manager:** [Homebrew](https://brew.sh/)

## 1. Install Minikube

Install `minikube` and `kubectl` with Homebrew:

```bash
brew install minikube kubectl
```

![Install Minikube](./screenshots/minikube-install.jpg)

## 2. Start the Cluster

```bash
minikube start --driver=docker
```

![Start Minikube](./screenshots/minikube-start.jpg)
## 3. Verify the Cluster

Check the overall Minikube status:

```bash
minikube status
```

![Minikube status](./screenshots/minikube-status.jpg)
View a summary of the cluster nodes:

```bash
kubectl get nodes -o wide
```

View the full specifications, capacity, and conditions for the Minikube node:

```bash
kubectl describe node minikube
```

![Minikube node info1](./screenshots/minikube-node.jpg)

![Minikube node info2](./screenshots/minikube-node2.jpg)
## 4. Create the Namespace

Create the `cha-express` namespace to isolate all subsequent application resources:

```bash
kubectl create namespace cha-express
```

Verify that the namespace was created:

```bash
kubectl get namespaces
```

Set `cha-express` as the default namespace for your current kubectl context:

```bash
kubectl config set-context --current --namespace=cha-express
```

![Namespace configuration](./screenshots/namespace.jpg)

## 5. Control Plane Components

In a Minikube cluster, the core Kubernetes control plane components run as pods inside the kube-system namespace.

List all running system pods to inspect them:

```bash
kubectl get pods -n kube-system
```

![kube-system pods list](./screenshots/kubectl-info.jpg)

Check the kubelet service status inside the Minikube virtual machine:

```bash
minikube ssh -- systemctl status kubelet
```

![kubelet status](./screenshots/kubelet.jpg)


# Task 2: ConfigMap

## 1. ConfigMap

Create the `cha-config` ConfigMap in the `cha-express` namespace. The manifest contains environment key-value pairs and the `index.html` file:

[View cha-config.yaml](./config/cha-config.yaml)

### Apply and verify the ConfigMap

Run the following commands to create and inspect the ConfigMap inside cha-express:

```bash
kubectl apply -f config/cha-config.yaml
kubectl get configmap cha-config -n cha-express
kubectl describe configmap cha-config -n cha-express
```

![configmap](./screenshots/configmap.jpg)



# Task 3: Secret

### Secret `cha-secret` and secret encryption

### 1. Create the `cha-secret` Secret

Create a manifest named `cha-secret.yaml` in the `cha-express` namespace with dummy values for `ADMIN_PASSWORD` and `PAYMENT_API_KEY`:

[View cha-secret.yaml](./config/cha-secret.yaml)

Apply and inspect the Secret:

```bash
kubectl apply -f config/cha-secret.yaml
kubectl get secret cha-secret -n cha-express
kubectl describe secret cha-secret -n cha-express
```

![Secret configuration](./screenshots/secret.jpg)

### 2. Are Kubernetes Secrets encrypted?

By default, no. Kubernetes Secrets are only base64-encoded, not encrypted at rest in etcd. Encryption at rest is optional and must be explicitly enabled on the cluster with an `EncryptionConfiguration` passed to `kube-apiserver`, using providers such as `aescbc`, `secretbox`, or a KMS provider.

Fetch and decode a Secret value to demonstrate that base64 encoding is reversible:

```bash
kubectl get secret cha-secret -n cha-express -o jsonpath='{.data.ADMIN_PASSWORD}' | base64 -d && echo
```

![Base64 decode](./screenshots/secret2.jpg)

# Task 4: Deployment & Service

### 1. Create the Deployment (`cha-deployment.yaml`)

This Deployment runs `nginx:1.25-alpine` with **3 replicas** inside the `cha-express` namespace, loads all keys from `cha-config` via `envFrom`, injects only `ADMIN_PASSWORD` from `cha-secret`, mounts `index.html` from `cha-config` into Nginx's web root (`/usr/share/nginx/html/index.html`), and defines CPU and memory resource requests and limits.

[View cha-deployment.yaml](./config/cha-deployment.yaml)



Apply and verify the Deployment:

```bash
kubectl apply -f config/cha-deployment.yaml
kubectl rollout status deployment/cha-web -n cha-express
kubectl get pods -n cha-express -l app=cha-web
```


![Deployment](./screenshots/deploy.jpg)

### 2. Create the NodePort Service

Create a NodePort Service targeting the `cha-web` pods on port 80 and exposing external node port 30080:

[View cha-service.yaml](./config/cha-service.yaml)

Apply and inspect the Service:

```bash
kubectl apply -f config/cha-service.yaml
kubectl get svc cha-web-service -n cha-express
```

![Service](./screenshots/service.jpg)

### 3. Open Cha Express in a Browser

On macOS with the Minikube Docker driver, open a tunnel and launch the menu page in your browser:

```bash
minikube service cha-web-service -n cha-express
```

![Service URL](./screenshots/service2.jpg)

![Cha Express menu](./screenshots/service3.jpg)


### 4. Proof That Environment Variables Are Available Inside the Container

Execute printenv inside one of the running cha-web pods to verify that:
All key-value pairs from cha-config (SHOP_NAME, CITY, THEME_COLOR, OPENING_HOURS) are present.
ADMIN_PASSWORD from cha-secret is present.

PAYMENT_API_KEY from cha-secret is not injected.


```bash

POD_NAME=$(kubectl get pods -n cha-express -l app=cha-web -o jsonpath='{.items[0].metadata.name}')

kubectl exec -n cha-express "$POD_NAME" -- sh -c \
  'echo "SHOP_NAME=$SHOP_NAME"; \
   echo "CITY=$CITY"; \
   echo "THEME_COLOR=$THEME_COLOR"; \
   echo "OPENING_HOURS=$OPENING_HOURS"; \
   echo "ADMIN_PASSWORD=$ADMIN_PASSWORD"; \
   echo "PAYMENT_API_KEY=${PAYMENT_API_KEY:-NOT_SET}"'


```


![Pod environment variables](./screenshots/pod-env.jpg)






# Task 5: GitHub Copilot



## 1. Prompt Given to GitHub Copilot

The following prompt was provided to GitHub Copilot Chat (in VS Code) to generate both manifests:

> *"Generate a Kubernetes Deployment and NodePort Service YAML for an Nginx app (`nginx:1.25-alpine`) in the `cha-express` namespace with 3 replicas. Load all keys from ConfigMap `cha-config` as environment variables, mount its `index.html` key to `/usr/share/nginx/html/index.html`, inject only `ADMIN_PASSWORD` from Secret `cha-secret` as an env var, set CPU and memory requests/limits, and expose the app on NodePort `30080`."*

---

## 2. Raw Output Generated by GitHub Copilot

> *"The request is already implemented in cha-deployment.yaml and cha-service.yaml."*


## No change required, as the deployment and service was exposed on nodeport manuallly.








# Task 6: Rolling Update & Rollback



## 1. Update Image to `nginx:1.27-alpine` and Record the Change

First, annotate the initial revision (Revision 1) so its rollout history has a clear change cause:

```bash
kubectl annotate deployment/cha-web -n cha-express kubernetes.io/change-cause="Initial deployment with nginx:1.25-alpine"
```


Update the container image to nginx:1.27-alpine, record the reason in kubernetes.io/change-cause, and watch the rolling update progress in real time:


```bash
kubectl set image deployment/cha-web nginx=nginx:1.27-alpine -n cha-express
kubectl annotate deployment/cha-web -n cha-express kubernetes.io/change-cause="Upgrade nginx image from 1.25-alpine to 1.27-alpine"

# Watch the rollout progress live
kubectl rollout status deployment/cha-web -n cha-express
kubectl get pods -n cha-express -l app=cha-web -w
```

![Rolling update](./screenshots/rollback.jpg)

## 2. View Rollout History with Recorded Reasons

Inspect the revision history of cha-web:

```bash
kubectl rollout history deployment/cha-web -n cha-express
```

To inspect the exact image and pod template details of a specific revision (for example, Revision 2):

```bash
kubectl rollout history deployment/cha-web -n cha-express --revision=2
```

![Rollout history](./screenshots/rollback2.jpg)

## 3. Deploy an Invalid Image Tag and Restore the Previous Working Version

### Step A: Trigger a broken rollout (`nginx:invalid-tag-999`)

Update the deployment to a non-existent image tag and record the change cause:

```bash
kubectl set image deployment/cha-web nginx=nginx:invalid-tag-999 -n cha-express
kubectl annotate deployment/cha-web -n cha-express kubernetes.io/change-cause="Deploy broken image tag nginx:invalid-tag-999"
```

Check pod status to observe the stalled rollout (ImagePullBackOff / ErrImagePull):

```bash
kubectl get pods -n cha-express -l app=cha-web
```

![Invalid image rollout](./screenshots/invalid.jpg)

### Step B: Roll back to the previous working revision (`nginx:1.27-alpine`)

Undo the failed rollout and verify that all 3 replicas return to a healthy state:

```bash
kubectl rollout undo deployment/cha-web -n cha-express
kubectl rollout status deployment/cha-web -n cha-express
kubectl get pods -n cha-express -l app=cha-web
kubectl rollout history deployment/cha-web -n cha-express
```

![Rollback verification](./screenshots/rollback3.jpg)

## 4. Question: Did the Website Go Down During the Update?

No, the website did not go down during either the valid upgrade (1.25-alpine → 1.27-alpine) or the broken image rollout (invalid-tag-999).

### Why it stayed online

The default `RollingUpdate` strategy uses `maxSurge: 25%` and `maxUnavailable: 25%`:

For a Deployment with replicas: 3, Kubernetes rounds 25% down to 0 for maxUnavailable (meaning at least 3 ready pods must remain available at all times) and rounds 25% up to 1 for maxSurge (allowing up to 4 total pods during the transition).

Kubernetes spins up a new pod first and waits until it passes readiness checks (1/1 Running) before terminating an old pod.

### Service endpoint filtering

The NodePort Service (cha-web-service) only routes incoming traffic to pods whose Ready condition is True.

When nginx:invalid-tag-999 failed with ImagePullBackOff, the new pod never became Ready, so Kubernetes halted replacement and kept all 3 existing nginx:1.27-alpine pods serving traffic without interruption.

