# My Homelab

A personal homelab project built on an old Lenovo G480 laptop. I use this environment to learn and practice Linux system administration, networking, Docker, Kubernetes, storage, monitoring, Git, and infrastructure management.

The project is intentionally lightweight because the server has only 4 GB of RAM.

## 🖥️ Infrastructure

```text
                         Home Network
                        192.168.1.0/24
                              │
                              ▼
                     Router / Gateway
                        192.168.1.1
                              │
                              ▼
                     Lenovo G480 Laptop
                        Debian 13
                      192.168.1.30
                              │
                    ┌─────────┴─────────┐
                    │                   │
                   SSH              K3s Kubernetes
                  Port 22                 │
                                        ▼
                                  Traefik Ingress
                                        │
              ┌─────────────────────────┼─────────────────────────┐
              │                         │                         │
              ▼                         ▼                         ▼
         namespace: lab          namespace: jellyfin       namespace: monitoring
              │                         │                         │
              ▼                         ▼              ┌──────────┼──────────┐
            Nginx                    Jellyfin           │          │          │
              │                         │           Prometheus  Grafana  Exporters
              ▼                         ▼
         nginx.lab                jellyfin.lab
```

## 💻 Server Hardware

| Component | Specification       |
| --------- | ------------------- |
| Device    | Lenovo G480         |
| CPU       | Intel Core i3-3110M |
| RAM       | 4 GB DDR3           |
| Storage   | ~466 GB             |
| GPU       | NVIDIA GeForce 610M |
| OS        | Debian 13 Minimal   |
| Server IP | `192.168.1.30`      |
| Gateway   | `192.168.1.1`       |

## 🌐 Network

My homelab uses the following local network:

```text
Network:       192.168.1.0/24
Gateway:       192.168.1.1
Homelab:       192.168.1.30
```

### Local Services

| Service    | Address           |
| ---------- | ----------------- |
| Nginx      | `nginx.lab`       |
| Jellyfin   | `jellyfin.lab`    |
| Grafana    | `grafana.lab`     |
| Prometheus | `prometheus.lab`  |
| SSH        | `192.168.1.30:22` |

## 🧰 Technology Stack

| Technology         | Purpose                                  |
| ------------------ | ---------------------------------------- |
| Debian 13          | Server operating system                  |
| SSH                | Remote server administration             |
| Docker             | Containerization                         |
| Docker Compose     | Running early services                   |
| K3s                | Lightweight Kubernetes                   |
| Traefik            | Kubernetes ingress                       |
| Nginx              | Web server / test application            |
| Jellyfin           | Media server                             |
| Prometheus         | Metrics collection                       |
| Grafana            | Monitoring dashboards                    |
| Node Exporter      | Linux host metrics                       |
| Kube State Metrics | Kubernetes resource metrics              |
| Kubernetes PVC     | Persistent storage                       |
| Git                | Version control                          |
| GitHub             | Source code and configuration management |

## ☸️ Kubernetes

The current environment uses **K3s**, a lightweight Kubernetes distribution suitable for the limited hardware available.

Kubernetes node:

```text
Node: debian
Kubernetes: v1.36.4+k3s1
```

### Namespaces

```text
lab
jellyfin
monitoring
```

## 📁 Repository Structure

```text
homelab/
├── .gitignore
├── README.md
│
├── archive/
│   └── docker/
│       ├── jellyfin-compose.yaml
│       ├── nginx-compose.yaml
│       └── nginx-html/
│           └── index.html
│
└── k8s/
    ├── apps/
    │   ├── jellyfin/
    │   │   ├── deployment.yaml
    │   │   ├── ingress.yaml
    │   │   └── service.yaml
    │   │
    │   └── nginx/
    │       ├── deployment.yaml
    │       ├── ingress.yaml
    │       └── service.yaml
    │
    ├── infrastructure/
    │   └── storage/
    │       ├── grafana-pvc.yaml
    │       ├── jellyfin-pvc.yaml
    │       └── prometheus-pvc.yaml
    │
    ├── monitoring/
    │   ├── grafana/
    │   │   ├── deployment.yaml
    │   │   └── ingress.yaml
    │   │
    │   ├── kube-state-metrics/
    │   │   └── deployment.yaml
    │   │
    │   ├── node-exporter/
    │   │   └── daemonset.yaml
    │   │
    │   └── prometheus/
    │       ├── configmap.yaml
    │       ├── deployment.yaml
    │       └── ingress.yaml
    │
    └── namespaces/
        ├── jellyfin.yaml
        ├── lab.yaml
        └── monitoring.yaml
```

## 🌐 Traefik Ingress

Traefik is used as the Kubernetes ingress controller.

Current routes:

```text
nginx.lab
    │
    └──> Nginx Service

jellyfin.lab
    │
    └──> Jellyfin Service :8096

grafana.lab
    │
    └──> Grafana Service :3000

prometheus.lab
    │
    └──> Prometheus Service :9090
```

This allows me to access services using hostnames instead of remembering different Kubernetes ports.

## 🎬 Jellyfin

Jellyfin runs inside the `jellyfin` namespace.

Configuration:

```text
Namespace:  jellyfin
Service:    jellyfin
Port:       8096
TargetPort: 8096
NodePort:   30096
```

Kubernetes manifests:

```text
k8s/apps/jellyfin/
├── deployment.yaml
├── service.yaml
└── ingress.yaml
```

The preferred access method is through the Traefik hostname:

```text
http://jellyfin.lab
```

The direct Kubernetes NodePort is:

```text
http://192.168.1.30:30096
```

## 📊 Monitoring

The homelab includes a monitoring stack based on Prometheus and Grafana.

```text
                    ┌───────────────┐
                    │   Prometheus  │
                    │     :9090    │
                    └───────┬───────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
        Node Exporter   Kube State    Kubernetes
          :9100         Metrics       Metrics
                         :8080
              │
              ▼
        Linux Host Metrics

                    Prometheus
                        │
                        ▼
                    Grafana
                      :3000
                        │
                        ▼
                 Monitoring Dashboard
```

### Prometheus

Prometheus collects metrics from:

* Node Exporter
* Kube State Metrics
* Kubernetes workloads

Prometheus configuration is located at:

```text
k8s/monitoring/prometheus/
├── configmap.yaml
├── deployment.yaml
└── ingress.yaml
```

Prometheus uses persistent storage:

```text
prometheus-data
```

The current configuration uses:

```text
Scrape interval: 15s
Retention:       24h
```

The short retention period helps reduce disk usage on the small server.

### Grafana

Grafana is used to visualize Prometheus metrics.

Configuration:

```text
k8s/monitoring/grafana/
├── deployment.yaml
└── ingress.yaml
```

Access:

```text
http://grafana.lab
```

Grafana uses persistent storage:

```text
grafana-data
```

## 📦 Persistent Storage

Persistent storage is configured using Kubernetes PersistentVolumeClaims.

```text
k8s/infrastructure/storage/
├── grafana-pvc.yaml
├── jellyfin-pvc.yaml
└── prometheus-pvc.yaml
```

Current PVCs:

```text
jellyfin-data
prometheus-data
grafana-data
```

Persistent storage is important because application data should survive pod restarts or recreation.

Jellyfin media is kept outside the Git repository.

Example:

```text
/media/
├── movies/
├── tv/
└── music/
```

## 🐳 Docker

Docker was used during the earlier stage of the homelab project.

The original Docker Compose configurations are kept in:

```text
archive/docker/
```

```text
archive/docker/
├── jellyfin-compose.yaml
├── nginx-compose.yaml
└── nginx-html/
    └── index.html
```

These files are archived because the current infrastructure is primarily Kubernetes-based.

## 🔐 Resource-Conscious Design

This server only has **4 GB RAM**, so resource usage is important.

Kubernetes workloads use CPU and memory requests/limits where appropriate.

For example:

```yaml
resources:
  requests:
    cpu: 50m
    memory: 128Mi
  limits:
    cpu: 200m
    memory: 256Mi
```

The goal is to keep the infrastructure stable while still running useful services.

## ❤️ Health Checks

Important workloads use Kubernetes health probes.

Health checks help Kubernetes determine whether an application is:

* Running correctly
* Ready to receive traffic
* In need of restarting

This is currently used for services such as Prometheus and Grafana.

## 📌 Git and Configuration Management

The homelab configuration is maintained with Git.

Repository:

```text
https://github.com/pkkmd11/homelab
```

The goal is to keep infrastructure configuration version-controlled and reproducible.

Typical workflow:

```bash
git status
git add .
git commit -m "describe the change"
git push origin main
```

## 🚀 Learning Goals

This homelab is mainly a learning environment.

Current learning areas include:

* Linux server administration
* SSH
* Networking
* Docker
* Docker Compose
* Kubernetes
* K3s
* Kubernetes namespaces
* Services
* Ingress
* Traefik
* Persistent storage
* Monitoring
* Prometheus
* Grafana
* Git and GitHub
* Infrastructure organization
* Resource management
* Application deployment

## ✅ Completed

* [x] Install Debian 13 minimal server
* [x] Configure static IP
* [x] Configure SSH
* [x] Install Docker
* [x] Run applications with Docker Compose
* [x] Create Git repository
* [x] Push infrastructure configuration to GitHub
* [x] Install K3s
* [x] Create Kubernetes namespaces
* [x] Deploy Nginx
* [x] Configure Traefik ingress
* [x] Deploy Jellyfin
* [x] Configure Jellyfin persistent storage
* [x] Deploy Prometheus
* [x] Deploy Grafana
* [x] Deploy Node Exporter
* [x] Deploy Kube State Metrics
* [x] Add persistent storage for Prometheus
* [x] Add persistent storage for Grafana
* [x] Add health probes
* [x] Add CPU and memory requests/limits
* [x] Pin monitoring container images
* [x] Organize Kubernetes manifests

## 🔜 Future Plans

* [ ] Improve Grafana dashboards
* [ ] Add useful Prometheus alerts
* [ ] Improve backup strategy
* [ ] Learn Helm
* [ ] Add CI/CD
* [ ] Improve Kubernetes security
* [ ] Add more services
* [ ] Document disaster recovery
* [ ] Improve infrastructure automation

## 🎯 Purpose

This project is not designed to be a production enterprise environment.

It is my personal learning lab where I can experiment with real infrastructure, make mistakes, troubleshoot problems, and improve my skills.

The main goal is to build practical experience with **Linux, networking, containers, Kubernetes, monitoring, and software infrastructure** using hardware that I already have.

---

**Maintained by Phyo Kyaw Ko**

GitHub: `https://github.com/pkkmd11`
