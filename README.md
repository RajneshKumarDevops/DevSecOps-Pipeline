# DevSecOps CI/CD Pipeline

## Overview
This project demonstrates a modern DevSecOps CI/CD pipeline using best practices to automate, secure, and monitor application delivery on Kubernetes.
It combines Jenkins and GitHub Actions for CI, HashiCorp Vault for secure secrets management, SonarQube and Trivy for code quality and vulnerability scanning, and ArgoCD with Argo Rollouts for GitOps and progressive delivery. Prometheus and Grafana provide real-time monitoring. Ingress handles clean URL routing and TLS.

## Key Features
- Continuous integration with Jenkins and GitHub Actions  
- Secure secret injection using HashiCorp Vault with AppRole authentication  
- Static code analysis with SonarQube and test coverage enforcement  
- Vulnerability scanning with Trivy and Grype for containers and dependencies  
- GitOps deployment with ArgoCD  
- Progressive canary deployments using Argo Rollouts  
- Ingress controller with domain routing and TLS termination  
- Monitoring with Prometheus metrics and Grafana dashboards


## Installation & Setup

1. Clone the repository   
   ```bash
   git clone https://github.com/RihabHaddad/DevSecOps-pipeline.git
   cd DevSecOps-pipeline
2. Configure Vault  
3. Set up Jenkins  
4. Configure GitHub Actions  
5. Deploy ArgoCD and Argo Rollouts  
6. Configure the Ingress Controller  
7. Deploy Prometheus and Grafana

## Usage
- Push code or manifest changes to the GitHub repository  
- CI pipelines run automatically: unit tests, code scans, Docker image builds, and pushes  
- GitOps repo is updated with the new image tag and deployment configuration  
- ArgoCD syncs changes and deploys via canary strategy  
- Traffic is routed through the Ingress controller  
- Grafana provides real-time visibility into app and rollout metrics

## Learn More
For a full hands-on tutorial with architecture, pipeline examples, and configuration details, check out the article:  
👉 [Build a Secure DevSecOps CI/CD Pipeline with Jenkins, ArgoCD, Trivy, and Vault](https://devsecopsai.today/build-a-secure-devsecops-ci-cd-pipeline-with-jenkins-argocd-trivy-and-vault-hands-on-tutorial-47aeb65090b1)

## Contributing
Contributions are welcome!  
Please fork the repository and submit a pull request.  
For issues or feature requests, open a GitHub issue.
