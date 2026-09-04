# My Homelab

This repository documents my personal homelab built with an old Lenovo G480 laptop.

The main purpose of this homelab is to learn Linux server administration, networking, Docker, Git, GitHub, and eventually CI/CD.

## Hardware

- Laptop: Lenovo G480
- CPU: Intel Core i3-3110M
- RAM: 4 GB DDR3
- Storage: ~466 GB
- GPU: NVIDIA GeForce 610M

## Operating System

- Debian 13
- Server/minimal installation
- No desktop environment

## Current Services

- SSH
- Docker
- Nginx

## Network

- Server IP: `192.168.1.30`
- Gateway: `192.168.1.1`

## Project Structure

```text
/homelab
├── README.md
└── nginx
    ├── compose.yaml
    └── html
        └── index.html

## Architecture

```text
                    Home Network
                         │
                         │
                  Router / Gateway
                    192.168.1.1
                         │
                         │
                 ┌───────┴───────┐
                 │               │
          Other Devices      Lenovo G480
                              Debian 13
                            192.168.1.30
                                 │
                         ┌───────┴───────┐
                         │               │
                       Docker           SSH
                         │
                       Nginx
                         │
                    Port 80 (HTTP)
                         │
                         ▼
                  Web Browser
