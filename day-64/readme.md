## 🚀 Day 64/90: EasyShop – Full-Stack Microservices on Kubernetes

Today I built and deployed a complete **MERN e-commerce application (EasyShop)** to **Kubernetes**. 🎉

This wasn't a hello-world project. It was a real application with a **React frontend**, **Node/Express backend**, and **MongoDB database**. Everything was containerized and orchestrated with Kubernetes. 🐳☸️

### 🏗️ Full Architecture

✅ **Frontend Deployment**

* React application
* Multiple replicas
* HPA auto scaling

✅ **Backend Deployment**

* Node.js/Express API
* Load balanced across pods

✅ **MongoDB StatefulSet**

* Persistent storage
* Dedicated database pod

✅ **Database Migration Job**

* Runs once
* Initializes the database

✅ **Ingress**

* `/` → Frontend
* `/api` → Backend

✅ **Services**

* Internal networking between microservices

✅ **Metrics Server**

* Enables HPA to monitor CPU usage

### ⚙️ Kubernetes Concepts Used

📦 Deployments
💾 StatefulSets
🌐 Services
🚪 Ingress
🛠️ Jobs
📈 Horizontal Pod Autoscaler (HPA)
📝 ConfigMaps
🔐 Secrets
💽 Persistent Volumes

Everything was integrated into one fully working system. 🚀

This is what production infrastructure looks like. Not isolated components, but an application where every piece works together. 💪

**64/90 complete. On to Day 65!** 🔥

#90DaysOfDevOps #Kubernetes #Microservices #EasyShop #DevOps #Docker #MERN #CloudComputing

