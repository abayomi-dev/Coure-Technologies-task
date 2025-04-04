# SOLUTION TO THE COURE ASSESSMENT

## PROBLEM STATEMENT  
- Connecting the frontend service to the backend service, which includes both the Node.js frontend application and backend services.  
- Resolving environmental inconsistencies and ensuring a fault-tolerant, resilient system that can handle rollout and deployment failures.  
- Illustrating the architectural flow and utilizing CI/CD to create a fast and efficient development environment.  
- Diagnosing and mitigating memory leaks while implementing proper monitoring for services.  
- Setting up alerts and monitoring systems to address issues in production and critical network environments.  

## QUESTIONS AND SOLUTIONS  
First of all, We setup two architectural diagrams as well as schematics and skeletal files structures to show a practical way of tackling issues raised here.</br>
We have the **`.github`** files to host the CICD pipelines, as well as the **`/app`** folder for the applications source code services which includes **Deployment and Dockerfiles** for the working and building of the application.</br>
We also have infrastructure files for the provisioning of any VM Resource neccesary as well as the use of monitoring scripts for log collection and dashboards, also with the use of scripts to work on file deployments and rollbacks.</br>

These are necessary to working on all the applications and problems to ensure a smooth easy and highly resilient infrastructure with ease of deployment, **faster** & accurate environmental segregation using git branches and namespaces.

As well as the reduction in production issues(either to resources or to services) running the applications so as to also prevent memory leaks and have a **cluster-wide** robust monitoring and alerting mechanism.

**BELOW ARE SOLUTIONS TO THE ASSESSMENT**</br>
### ARCHITECTURAL OVERVIEW  
For the Kubernetes cluster design, including the number of nodes, node pools, resource allocation, network design, and storage considerations, refer to the diagram below:  

**[Application Architectural Diagram](images/Coure.drawio.png)**  

In this setup, we employ a **three-node pool strategy** to ensure high availability and redundancy, suitable for both on-premise and cloud deployments. Key design principles include:  

- **Stateful Node Group**: Handles stateful services like databases (MongoDB, Redis) and RabbitMQ.  
- **Stateless Node Group**: Hosts services that do not require persistent storage, including the frontend and backend APIs.  
- **Tools Node Group**: Supports monitoring tools like Prometheus and Grafana.  

**Networking Considerations:**  
- Use of **network policies** to control inter-service communication.  
- **Ingress resources** for external traffic management, ensuring proper routing between services.  
- **Load balancer** to distribute traffic efficiently across services.  

**Resource Allocation & Scaling:**  
- **Requests and limits** are defined to manage CPU and memory usage efficiently.  
- **Horizontal Pod Autoscaler (HPA)** ensures services scale dynamically based on demand.  

**Storage Considerations:**  
- **Persistent storage (gp2/gp3 classes)** mounted via **CSI drivers** for stateful applications.  
- **ConfigMaps and Secrets** handle configuration storage securely.  

---

### HIGH AVAILABILITY STRATEGY  

#### **Handling Failures**  
- **Multi-AZ node distribution** to prevent single points of failure.  
- **Pod Anti-Affinity rules** to avoid co-locating critical pods.  
- **Cluster Autoscaler** replaces unhealthy nodes dynamically.  

#### **Database Replication**  
- **MongoDB**: Uses a three-node replica set with one arbiter.  
- **Redis**: Cluster mode with sharding and replication.  
- **RabbitMQ**: Mirrored queues to ensure redundancy.  

#### **Load Balancing**  
- **L4/L7 Load Balancers** for traffic distribution.  
- **Istio Gateway** for advanced traffic management and canary deployments.  
- **Failover Load Balancer** setup prevents single points of failure.  

#### **Disaster Recovery**  
- **Daily snapshots** of MongoDB/Redis stored in offsite or object storage (e.g., S3).  
- **Geo-redundant cluster** for secondary region disaster recovery.  

---

### CI/CD PIPELINE DESIGN  
**[CICD PIPELINE ILLUSTRATION](images/cicd.drawio.png)**  
In this Pipeline example we see how the python and Javascript workflow was designed using prettier for code validation and then approved by a management team or whoever has permissions and if not approved we head back to the work item. </br>
When tested as seen in the first CI stage using the SNYK and Sonarqube tools for SAST, we then deploy using branching styles from main branch to production and Dev/feature branch to alternate staging environment.</br>
#### **Code Validation**  
- Prettier and ESLint for formatting and linting, depending on Personal preference and projects requirements

#### **Security Scanning**  
- Integrate **Snyk** or **Trivy** for Docker image and Kubernetes manifest security scanning.  

#### **Testing Strategy**  
- **Unit Testing**: Jest for backend, Mocha/Chai for Node.js services.  
- **End-to-End Testing**: Cypress or Selenium for frontend testing.  

#### **Deployment Strategy**  
- **Helm Charts** ensure configuration consistency and version control.  
- **Helmfile** manages multiple Helm charts for large-scale infrastructure.  
- **Canary Deployments** gradually introduce updates while monitoring performance.  

#### **Rollback Procedures**  
- **Helm Rollbacks** allow reverting to previous releases if issues arise.  
- **Kubernetes Rollout Undo** ensures failed deployments can be undone immediately.  

---

### GITOPS APPROACH  
**Branch Strategy</br>**
</pre>``` # infra/helm/mongodb/templates/statefulset.yaml
kind: StatefulSet
spec:
  replicas: 3
  volumeClaimTemplates:
    - metadata:
        name: mongodb-data
      spec:
        storageClassName: "gp2"
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 10Gi
```</pre> </br>

**Snippet illustrating this**
</pre>``` # infra/argocd/applications/prod.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ecommerce-prod
spec:
  project: default
  source:
    repoURL: https://github.com/your-username/ecommerce-k8s-modernization
    targetRevision: main
    path: infra/helm/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: prod```</pre>
Use a Gitflow branching model(either from concurrent environment or features development):

Master/Main: For production-ready code.
Develop: For integration and testing.
Feature branches: For individual features or bug fixes.

Environment Promotion
Implement GitOps using tools like ArgoCD in this case, This ensures that the code automatically deploys to different environments based on the branch, we setup branche thast deployes the features into the respective envioronemt due to the branching.
`Main <----> Production Ready Environment probable segreageted by uts environment or namespaces.`</br>
`Dev branch <----> Develop environemts.`</br>
`Test branches <----> Test environments either for features or integrated tests on a service feature etc.`</br>

**Configuration Management**</br>
Use Kubernetes ConfigMaps and Secrets to store configuration variables (e.g., API keys, database credentials).
We deploy the ap[plication by making changes into the repository hosting the source codes and then we deploy the changes on our repository which  auto reflects on our argocd dashboard after propeerrly setting up the necessay kustomize files as seen in the files structures showing a skeletal of how this process might have been when setup.</br>
Store environment-specific configurations in separate files or branches to ensure they don’t leak into production.
**Change Approval Process**</br>
Integrate Pull Request (PR) Reviews for code changes.
We Use tools like ArgoCD to automatically apply changes to the Kubernetes cluster once the PR is merged.

# PAIN POINTS
- With the keen and simplicication of the workflow using the afore mentioned guidelines we should estasblish a faster and more in sync cluser setup that works for fast deployment using argocd and helm.</br>
- Proper Monitoring is implemented to address the issues regarding late discovery of production issues with proper system and service metrics set up  with log aggregation tools to alert if a log throws a certain error or on a system/service/resource downtime.</br>
- To Address resource leaks with Resource limits and Prometheus alerts to assist with this failure.</br>
-  Isolated staging environemt with the right branching strategy will address issue of inconstent or no staging environment using the GitOps process.</br>
---