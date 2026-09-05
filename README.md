# My Homelab

This repository documents my personal homelab built with an old Lenovo G480 laptop.

The main purpose of this homelab is to learn Linux server administration, networking, Docker, Kubernetes, Git, GitHub, and eventually monitoring and CI/CD.

## Hardware

* Laptop: Lenovo G480
* CPU: Intel Core i3-3110M
* RAM: 4 GB DDR3
* Storage: ~466 GB
* GPU: NVIDIA GeForce 610M

## Operating System

* Debian 13
* Server/minimal installation
* No desktop environment

## Current Services

* SSH
* Docker
* Jellyfin
* Kubernetes (K3s)
* Traefik

## Network

* Server IP: `192.168.1.30`
* Gateway: `192.168.1.1`

## Project Structure

```text
/homelab
├── README.md
├── .gitignore
├── nginx
│   ├── compose.yaml
│   └── html
│       └── index.html
├── jellyfin
│   └── compose.yaml
└── k8s
    └── nginx
        ├── deployment.yaml
        ├── service.yaml
        └── ingress.yaml
```

## Architecture

```text
                         Home Network
                              │
                              │
                       Router / Gateway
                         192.168.1.1
                              │
                              │
              ┌───────────────┴───────────────┐
              │                               │
       Other Devices                    Lenovo G480
                                      Debian 13
                                    192.168.1.30
                                          │
                       ┌──────────────────┼──────────────────┐
                       │                  │                  │
                      SSH               Docker           Kubernetes
                    Port 22                │                  │
                       │                Jellyfin              K3s
                       │                Port 8096             │
                       │                                     Traefik
                       │                                       │
                       │                                    Ingress
                       │                                       │
                       │                                   nginx.lab
                       │                                       │
                       │                               Nginx Service
                       │                                       │
                       │                                    Nginx Pod
                       │
                       ▼
                  SSH Client
```

## Jellyfin Media Server

Jellyfin is running on my Lenovo G480 as a Docker container.

It provides a self-hosted media server that I can access from devices on my home network.

### Jellyfin Configuration

* Container: `jellyfin`
* Port: `8096`
* Web interface: `http://192.168.1.30:8096`
* Docker Compose file: `jellyfin/compose.yaml`

### Media Storage

The media files are stored outside the Git repository:

```text
/media/
├── movies/
├── tv/
└── music/
```

The Jellyfin configuration is also excluded from Git because it contains generated configuration and database files.

## Kubernetes

Kubernetes is running using K3s because the Lenovo G480 has limited hardware resources.

### Kubernetes Cluster

* Distribution: K3s
* Node: `debian`
* Kubernetes version: `v1.36.4+k3s1`
* Namespace: `lab`

### Nginx Deployment

I deployed Nginx inside Kubernetes to practice basic Kubernetes concepts.

The Deployment is defined in:

```text
k8s/nginx/deployment.yaml
```

The Deployment creates an Nginx Pod using the `nginx:alpine` image.

### Kubernetes Service

The Nginx Pod is exposed internally using a ClusterIP Service.

Configuration:

```text
k8s/nginx/service.yaml
```

Service:

* Name: `lab-nginx-service`
* Type: `ClusterIP`
* Port: `80`

The Service provides stable internal networking to the Nginx Pod.

### Traefik Ingress

K3s includes Traefik as an Ingress controller.

I created an Ingress configuration in:

```text
k8s/nginx/ingress.yaml
```

The Ingress routes requests for:

```text
nginx.lab
```

to:

```text
lab-nginx-service:80
```

The local hostname `nginx.lab` is mapped to the server IP `192.168.1.30` on my Windows computer.

The Nginx application can then be accessed from my local network using:

```text
http://nginx.lab
```

### Kubernetes Architecture

```text
Windows PC
    │
    │ http://nginx.lab
    ▼
192.168.1.30
    │
    ▼
  Traefik
    │
    ▼
  Ingress
    │
    ▼
lab-nginx-service
    │
    ▼
 Nginx Pod
```

## Git and GitHub

The homelab configuration is managed with Git.

The repository is hosted on GitHub.

I use Git to track changes to configuration files and document the progress of the homelab.

## Learning Goals

My current learning goals are:

1. Linux server administration
2. Networking
3. Docker and Docker Compose
4. Kubernetes and K3s
5. Kubernetes Services and Ingress
6. Traefik
7. Persistent storage
8. Helm
9. Server and Kubernetes monitoring
10. Prometheus and Grafana
11. CI/CD

## Roadmap

* [x] Install Debian server
* [x] Configure static IP
* [x] Configure SSH
* [x] Install Docker
* [x] Run Nginx with Docker Compose
* [x] Run Jellyfin with Docker Compose
* [x] Create Git repository
* [x] Push homelab configuration to GitHub
* [x] Install K3s
* [x] Create Kubernetes namespace
* [x] Deploy Nginx to Kubernetes
* [x] Create Kubernetes Service
* [x] Configure Traefik Ingress
* [x] Configure local `nginx.lab` hostname
* [ ] PersistentVolume
* [ ] Helm
* [ ] Kubernetes monitoring
* [ ] Prometheus
* [ ] Grafana
* [ ] Alerts
* [ ] CI/CD
