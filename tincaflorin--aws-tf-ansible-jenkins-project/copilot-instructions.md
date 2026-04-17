## aws-tf-ansible-jenkins-project

> This project automates the setup of a Jenkins CI/CD server on AWS. It uses a combination of Terraform for infrastructure provisioning and Ansible for server configuration. The goal is to create a repeatable and automated way to deploy a Jenkins environment ready for building and deploying applications.

# Gemini CLI Context: CI/CD Infrastructure with Jenkins, Terraform, and Ansible

## Project Overview

This project automates the setup of a Jenkins CI/CD server on AWS. It uses a combination of Terraform for infrastructure provisioning and Ansible for server configuration. The goal is to create a repeatable and automated way to deploy a Jenkins environment ready for building and deploying applications.

### Key Technologies

*   **Terraform:** Used to define and provision the necessary AWS infrastructure, including a VPC, EC2 instance, security groups, and networking components.
*   **Ansible:** Used to configure the EC2 instance, install Jenkins, and set up the required dependencies.
*   **Docker:** Jenkins is run as a Docker container, and the setup includes Docker-in-Docker (dind) to allow Jenkins to build Docker images.
*   **AWS:** The entire infrastructure is deployed on Amazon Web Services.

## Project Structure

*   `infra_tf/`: Contains the Terraform configuration files for the AWS infrastructure.
*   `ansible/`: Contains the Ansible playbook and inventory for configuring the Jenkins server.
*   `docker/`: Contains the Dockerfile for building the custom Jenkins image.

## How to Use

### 1. Provision the Infrastructure

1.  **Navigate to the Terraform directory:**
    ```bash
    cd infra_tf
    ```
2.  **Initialize Terraform:**
    ```bash
    terraform init
    ```
3.  **Apply the Terraform configuration:**
    ```bash
    terraform apply
    ```
    This will create the AWS resources and generate an Ansible inventory file at `ansible/inventory/jenkins_server.ini`.

### 2. Configure the Jenkins Server

1.  **Navigate to the project root directory.**
2.  **Run the Ansible playbook:**
    ```bash
    ansible-playbook -i ansible/inventory/jenkins_server.ini ansible/jenkins_playbook.yaml
    ```
    This will connect to the EC2 instance, install Docker, build the custom Jenkins image, and start the Jenkins container.

### 3. Access Jenkins

Once the Ansible playbook has finished, you can access the Jenkins web interface by navigating to `http://<jenkins_server_public_ip>:8080` in your web browser. The initial administrator password will be printed to the console by the Ansible playbook.

## Development Conventions

*   **Terraform:** The Terraform code is organized into logical files (`main.tf`, `variables.tf`, `povider.tf`). It follows standard Terraform practices.
*   **Ansible:** The Ansible playbook is self-contained and includes all the necessary tasks for setting up the Jenkins server.
*   **Docker:** The Dockerfile is used to create a custom Jenkins image with the required plugins, ensuring a consistent and repeatable Jenkins environment.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TincaFlorin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
