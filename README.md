# Kubernetes Microservices Deployment Project

## Overview

This project demonstrates an end-to-end deployment of a containerized microservices application on Kubernetes using Minikube.
It simulates a real production-like environment with multiple application tiers, external traffic routing, autoscaling, and persistent storage.

The application stack consists of a frontend service, a backend API service, and a MySQL database.
Kubernetes resources such as Deployments, Services, Persistent Volumes, Secrets, Ingress, and Horizontal Pod Autoscaler are used to manage the application lifecycle.

---

## Architecture

The application follows a typical three-tier architecture:

* **Frontend (UI Layer)**
  Serves the user interface and communicates with the backend API.

* **Backend (API Layer)**
  Handles application logic and processes requests received from the frontend.

* **Database (Data Layer)**
  Stores application data using a persistent storage volume to ensure data durability.

External traffic is routed into the cluster using an NGINX Ingress Controller.
Host-based routing is configured to direct requests to the appropriate service.

The backend deployment is configured with a Horizontal Pod Autoscaler (HPA), allowing the system to scale automatically based on CPU utilization.

---

## Key Components Used

* Kubernetes Deployments for stateless application components
* Stateful database deployment with Persistent Volume and Persistent Volume Claim
* Kubernetes Services for internal communication and service discovery
* Kubernetes Secrets for database credential management
* NGINX Ingress Controller for external access and host-based routing
* Horizontal Pod Autoscaler for dynamic scaling of backend pods
* Namespace isolation for environment separation

---

## Traffic Flow

1. User sends a request to the application domain.
2. The request reaches the NGINX Ingress Controller.
3. Based on host rules, the request is routed to either the frontend or backend service.
4. The backend communicates with the MySQL database through the internal cluster network.
5. Persistent storage ensures that database data is retained even if pods restart.

---

## Autoscaling Behaviour

The backend deployment is configured with CPU-based autoscaling.

* Minimum replicas: 1
* Maximum replicas: 5
* Target CPU utilization: 50%

When application load increases, additional backend pods are created automatically.
When the load decreases, the system scales down to conserve cluster resources.

---

## Deployment Steps

1. Start Minikube cluster with sufficient CPU and memory.
2. Enable required addons such as Ingress Controller and Metrics Server.
3. Create application namespace.
4. Deploy database stack (PV, PVC, Secret, Deployment, Service).
5. Deploy backend and frontend services.
6. Configure Ingress resource for external routing.
7. Configure Horizontal Pod Autoscaler for backend deployment.

---

## Project Structure

```
kubernetes-microservices-project/
│
├── db-stack.yaml
├── backend.yaml
├── frontend.yaml
├── ingress.yaml
└── README.md
```

---

## Ingress Routing Rules and Behaviour

The application is exposed externally using the NGINX Ingress Controller with host-based routing.

The following routing rules are configured:

* Requests to **api.app.local** are routed to the backend API service.
* Requests to **app.local** are routed to the frontend UI service.
* Requests made directly using the cluster IP address do not match any host rule and are handled by the default NGINX backend, which returns a 404 response.

### Routing Configuration (Conceptual)

```
User Request
      ↓
NGINX Ingress Controller
      ↓
---------------------------------
Host: api.app.local   → backend-service
Host: app.local       → frontend-service
No host match         → default nginx 404
---------------------------------
```

### Observed Behaviour During Testing

* Accessing `api.app.local` returned the backend application response (Flask API message).
* Accessing `app.local` reached the frontend container served by NGINX.
* Accessing the application using the node IP address resulted in a default **404 Not Found** page from the Ingress controller.

This confirms that host-based routing is functioning correctly and external traffic is being directed to the appropriate microservice within the cluster.

### Example Ingress Resource

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: microservices-ingress
  namespace: project-prod
spec:
  ingressClassName: nginx
  rules:
  - host: app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80

  - host: api.app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 5000
```



---

## Learning Outcomes

This project helped in understanding:

* Kubernetes resource management and application deployment strategies
* Service exposure using Ingress Controller
* Persistent storage handling for stateful workloads
* Horizontal scaling using resource metrics
* Rolling updates and rollback capability in Deployments
* Namespace-based logical isolation

---


