<p align="center">🚀 SkillPulse – End-to-End DevOps CI/CD Pipeline on Kubernetes</p>
<p align="center"> Production-Ready Multi-Tier Application with GitHub Actions, Docker, Kubernetes (Kind), AWS EC2, Self-Hosted Runner & MySQL </p> <p align="center">
📖 Project Overview
🎯 Project Summary

SkillPulse is a production-inspired three-tier web application that demonstrates a complete DevOps CI/CD pipeline using modern cloud-native technologies. The project automates the entire software delivery lifecycle—from source code management to containerization and Kubernetes deployment—using GitHub Actions and a Self-Hosted Runner.

The application consists of an Nginx Frontend, a Go Backend API, and a MySQL Database, all deployed on a Kind Kubernetes Cluster running on an AWS EC2 instance. Docker is used for containerization, Docker Hub serves as the container registry, and Kubernetes manages application orchestration.

This project emphasizes automation, scalability, reliability, and production-ready deployment practices, making it an ideal portfolio project for DevOps and Cloud Engineers.
Solution Architecture
<p align="center"> <img src="assets/diagrams/solution-architecture.png" width="100%"> </p>
🏗️ Architecture Overview

The deployment workflow follows this architecture:

Developer
     │
     ▼
GitHub Repository
     │
     ▼
GitHub Actions
     │
     ▼
Self-Hosted Runner (AWS EC2)
     │
     ▼
Docker Build
     │
     ▼
Docker Hub
     │
     ▼
Kind Kubernetes Cluster
     │
     ▼
Namespace (skillpulse)
     │
     ▼
MySQL Database
     │
     ▼
Go Backend API
     │
     ▼
Nginx Frontend
     │
     ▼
Web Browser
🔄 End-to-End CI/CD Workflow
<p align="center"> <img src="assets/diagrams/cicd-workflow.png" width="100%"> </p>
📂 Repository Structure
skillpulse-devops/
│
├── .github/
│   └── workflows/
│       └── ci.yml                  # GitHub Actions CI/CD Pipeline
│
├── backend/                        # Go Backend Source Code
│   ├── main.go
│   ├── go.mod
│   ├── go.sum
│   └── Dockerfile
│
├── frontend/                       # Nginx Frontend
│   ├── index.html
│   ├── nginx.conf
│   └── Dockerfile
│
├── kubernetes/
│   ├── 00-namespace.yaml           # Namespace
│   ├── 10-mysql.yaml               # MySQL Deployment & Service
│   ├── 20-backend.yaml             # Backend Deployment
│   ├── 30-frontend.yaml            # Frontend Deployment
│   └── backend-service.yaml        # Backend Service
│
├── assets/
│   ├── banner/
│   │   └── banner.svg
│   │
│   ├── diagrams/
│   │   ├── solution-architecture.png
│   │   └── cicd-workflow.png
│   │
│   └── screenshots/
│       ├── step-01-clone-repository.png
│       ├── step-02-docker-compose.png
│       ├── ...
│       └── step-20-final-pipeline.png
│
├── docker-compose.yml              # Local Development
├── kind-config.yaml                # Kind Cluster Configuration
├── README.md                       # Project Documentation
├── LICENSE
├── CONTRIBUTING.md
└── CHANGELOG.md
📁 Folder Description
Folder/File	Description
.github/workflows/ci.yml	GitHub Actions workflow for CI/CD automation
backend/	Go backend application source code
frontend/	Nginx frontend application
kubernetes/	Kubernetes manifests for namespace, MySQL, backend, frontend, and services
assets/screenshots/	Deployment screenshots used in the README
assets/diagrams/	Architecture and CI/CD workflow diagrams
docker-compose.yml	Local multi-container development environment
kind-config.yaml	Kind Kubernetes cluster configuration
README.md	Complete project documentation
⚙️ Technology Stack
Category	Technologies
Programming Language	Go
Frontend	HTML, CSS, JavaScript, Nginx
Database	MySQL
Containerization	Docker
Container Registry	Docker Hub
CI/CD	GitHub Actions
Runner	Self-Hosted Runner
Orchestration	Kubernetes (Kind)
Cloud Platform	AWS EC2
Version Control	Git & GitHub
Operating System	Ubuntu (Linux)


---

## Acknowledgments & Credits

* **Project Concept & Base Architecture:** Inspired by the *TrainWithShubham GitHub Actions & Kubernetes Masterclass*. 
* **Implementation & Deployment:** Fully configured, deployed, and debugged by me. This includes setting up the multi-tier Docker containers, writing Kubernetes manifests, configuring GitHub Actions workflows, and managing the AWS infrastructure.


