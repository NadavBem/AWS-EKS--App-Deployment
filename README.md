# Deploying Application on Amazon EKS
Here’s a simplified guide for setting up and deploying an application on an Amazon EKS cluster:

---

### 1. **Install `kubectl`**

`kubectl` is the tool for managing Kubernetes clusters. You need it to interact with your EKS cluster.

- **For Linux**: Follow [this guide](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/).
- **For macOS**: Follow [this guide](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/).
- **For Windows**: Follow [this guide](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/).

### 2. **Install `eksctl`**

`eksctl` is a CLI tool to manage EKS clusters. You need it to simplify cluster creation and management.

- **Installation Guide**: Visit [eksctl installation](https://eksctl.io/installation/?source=post_page-----211eb46c069c--------------------------------#).

### 3. **Set Up Authentication and Permissions**

To access your EKS cluster securely:

1. **Create an IAM User and Group**:
   - Go to AWS Management Console → IAM Service.
   - Create a new user group and attach the `AmazonEKSClusterPolicy`.
   - Create a new user and add it to the group.
   - Generate and download the Access Key and Secret Access Key.

2. **Best Practices**:
   - Don’t store your keys in plain text.
   - Rotate keys regularly and use least-privilege permissions.

### 4. **Configure AWS CLI**

Set up AWS CLI with your IAM user credentials:

1. **Install AWS CLI**: Follow [this guide](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html).

2. **Configure AWS CLI**:

   Open a terminal and run:

   ```bash
   aws configure
   ```

   Enter your Access Key ID, Secret Access Key, default region, and output format.

### 5. **Create Your EKS Cluster**

Use `eksctl` to create the cluster. Create a `cluster.yaml` file with:

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: nadav-eks
  region: us-east-2
  tags:
    project: kubes-practice

nodeGroups:
  - name: worker-nodes
    instanceType: m5.large
    desiredCapacity: 3
```

Create the cluster:

```bash
eksctl create cluster -f cluster.yaml
```
This process can take several minutes (10-20 minutes).

### 6. **Validate Cluster Creation**

Check your cluster’s status:

- **Using `kubectl`**:

  ```bash
  # Identify which k8s cluster configured
  kubectl config get-contexts
  
  # To get all the nodes
  kubectl get nodes
  ```

- **Using AWS Console**:
  - Verify cluster status in the Amazon EKS dashboard.

### 7. **Update `kubeconfig`**

Update your `kubeconfig` to use `kubectl` with your new EKS cluster:

```bash
aws eks update-kubeconfig --name my-cluster
```

### 8. **Deploy an Application**

1. **Create a Namespace**:

   `namespace.yaml`:

   ```yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: webapp
   ```

   Apply it:

   ```bash
   kubectl apply -f namespace.yaml
   ```

2. **Create a Deployment**:

   `deployment.yaml`:

   ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: webapp-deployment
      namespace: webapp
      labels:
        app: webapp
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: webapp
      template:
        metadata:
          labels:
            app: webapp
        spec:
          containers:
          - name: webapp
            image: nadav0176/voting-app:latest
            ports:
            - containerPort: 80
   ```

   Apply it:

   ```bash
   kubectl apply -f deployment.yaml
   ```

3. **Create a Service**:

   `service.yaml`:

   ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: webapp-service
    spec:
      selector:
        app: webapp  
      type: LoadBalancer
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
   ```

   Apply it:

   ```bash
   kubectl apply -f service.yaml
   ```

### 9. **Access Your Application**

Get the external IP or DNS name:

```bash
kubectl get services -n development
```

### 3. **Done!**
Open the URL in your browser to access your application!!.

---
