🚀 Milestone Achieved: Successfully Migrated & Deployed a Full-Stack E-Commerce Platform on Microsoft Azure! 🌐



I’m excited to share that I have successfully completed the migration of a three-tier e-commerce application (EasyShop) from AWS to Microsoft Azure, deploying it dynamically via Kubernetes (AKS)!



Here is a quick look at the journey and the architectural changes made:



🏗️ The Architecture Setup:



Frontend/Backend: Built on Next.js, Redux, and Tailwind CSS.

Database: MongoDB StatefulSet configured with Azure Disk dynamic volume provisioning (PVCs).

Ingress & Routing: Deployed Nginx Ingress Controller and Cert-Manager via Helm for path-based routing.

🛠️ Key DevOps & Cloud Tasks Completed:



AWS to Azure Migration: Converted all AWS components (VPC, EKS, EC2) to Azure counterparts (VNet, AKS, Virtual Machines) using Terraform.

Infrastructure as Code (IaC): Standardized deployments using Azure provider-specific configurations (Standard SKU Public IPs, system-assigned identities, custom service CIDRs to prevent subnet overlaps).

VM Provisioning & Automation: Bootstrapped the Jenkins VM (Standard_B2s_v2 for quota-efficiency) using advanced bash provisioning scripts to auto-configure Docker, Java 21, Jenkins, Helm, and Kubectl.

App Deployment: Successfully orchestrated the stack, resolved Next.js standalone container hostname bindings, and exposed the application live via the Ingress LoadBalancer!

This project gave me deep hands-on experience in cloud networking, handling Azure subscription quota limits, configuring Kubernetes ingress routing, and automating deployments.







#CloudComputing #MicrosoftAzure #DevOps #Kubernetes #AKS #Terraform #NextJS #InfrastructureAsCode #CloudMigration #SystemArchitecture #TechJourney
