# k8s-minikube
k8s on minikube

# Minikube Setup & Cluster Configuration Guide

This guide walks through installing Minikube on macOS, initializing a local Kubernetes cluster, inspecting node information, configuring a dedicated `cha-express` namespace, and identifying the active Kubernetes control plane components.

---

## Prerequisites

- **OS:** macOS (Intel or Apple Silicon)
- **Resources:** Minimum 2 CPUs, 2 GB free RAM, 20 GB free disk space
- **Driver:** Container or VM manager installed and running (e.g., [Docker Desktop](https://www.docker.com/products/docker-desktop/), HyperKit, or QEMU)
- **Package Manager:** [Homebrew](https://brew.sh/)

---

## 1. Install Minikube and Start the Cluster

Install `minikube` and `kubectl` using Homebrew:

```bash
brew install minikube kubectl

