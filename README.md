# Learn Kubernetes

This repository contains practical lessons, scripts, and configuration files for
setting up and using a Kubernetes cluster.

## Learning path

| Lesson | Description |
| --- | --- |
| [Cluster setup](./01.%20Setupk8s/0.%20full%20step%20setup%20k8s.md) | Create a Kubernetes cluster on AWS EC2 instances and configure Cilium networking. |
| [OS and network setup](./01.%20Setupk8s/1.%20setup-os.md) | Configure node hostnames and `/etc/hosts`. |
| [Install Envoy Gateway](./install-envoy-gateway.md) | Replace an Ingress controller with Envoy Gateway and expose it through NodePort. |

## Repository contents

- [`01. Setupk8s/`](./01.%20Setupk8s/) — cluster setup guides and scripts
- [`1. setup-os.md`](./01.%20Setupk8s/1.%20setup-os.md) — basic OS configuration
- [`2. install-containerd.sh`](./01.%20Setupk8s/2.%20install-containerd.sh) — containerd installation
- [`3. install-k8s-component.sh`](./01.%20Setupk8s/3.%20install-k8s-component.sh) — Kubernetes component installation
- [`nginx-deployment.yaml`](./01.%20Setupk8s/nginx-deployment.yaml) — example NGINX deployment
- [`nginx-service.yaml`](./01.%20Setupk8s/nginx-service.yaml) — example NGINX service
- [`utilit-comand.md`](./01.%20Setupk8s/utilit-comand.md) — useful Kubernetes commands

## Prerequisites

The setup guides assume that you have:

- Several Linux servers or virtual machines
- SSH access to each node
- `sudo` privileges
- A network that allows the required Kubernetes and Cilium ports
- Basic familiarity with Linux and container concepts

## Basic usage

Run cluster setup scripts on every Kubernetes node and run control-plane
commands on the control-plane node. Review each script before running it,
especially values that reference node addresses or network ranges.
