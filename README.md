# Create a Terraform Kubernetes Python App
## User Story
Create a Terraform, Kubernetes, and a Hello World Python app. 

The project should demonstrates how to set up an AWS EKS cluster using Terraform, deploy a simple Python web server application to the cluster using Kubernetes, and expose the application via a LoadBalancer service.

### Definition of Done:
- Terraform Configurations for:
    1. AWS Provider setup.
    2. EKS.
    3. S3 bucket creation.
- Kubernetes Deployment & Service YAML Files for:
    1. Python app deployment (with http.server).
    2. Exposing the app via a service (LoadBalancer).
- Terraform Code to Integrate Kubernetes:
    1. Use Terraform to deploy the Python app and service to the Kubernetes cluster (EKS ).
    2. Verification of the deployed Python app being accessible through the service.

## Deliverables

### Prerequisites
- AWS account
- Terraform installed
- kubectl installed
- AWS CLI configured

### Terraform Configurations

#### AWS Provider & S3 Bucket Setup 

```hcl
provider "aws" {
  region = "us-west-2"
}

resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-unique-bucket-name"
  acl    = "private"
}
```

#### EKS Cluster

```hcl
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  cluster_name    = "my-cluster"
  cluster_version = "1.21"
  subnets         = ["subnet-12345678", "subnet-87654321"]
  vpc_id          = "vpc-12345678"

  node_groups = {
    eks_nodes = {
      desired_capacity = 2
      max_capacity     = 3
      min_capacity     = 1

      instance_type = "t3.medium"
    }
  }
}
```

### Kubernetes Deployment & Service YAML Files

#### Python App Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-world
        image: python:3.9-slim
        command: ["python", "-m", "http.server", "8080"]
        ports:
        - containerPort: 8080
```

#### Service to Expose the App

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world-service
spec:
  type: LoadBalancer
  selector:
    app: hello-world
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

### Terraform Code to Integrate Kubernetes

#### Deploy Python App and Service to EKS

```hcl
provider "kubernetes" {
  host                   = module.eks.cluster_endpoint
  cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)
  token                  = data.aws_eks_cluster_auth.my_cluster.token
}

resource "kubernetes_deployment" "hello_world" {
  metadata {
    name = "hello-world"
  }

  spec {
    replicas = 2

    selector {
      match_labels = {
        app = "hello-world"
      }
    }

    template {
      metadata {
        labels = {
          app = "hello-world"
        }
      }

      spec {
        container {
          name  = "hello-world"
          image = "python:3.9-slim"
          command = ["python", "-m", "http.server", "8080"]

          port {
            container_port = 8080
          }
        }
      }
    }
  }
}

resource "kubernetes_service" "hello_world_service" {
  metadata {
    name = "hello-world-service"
  }

  spec {
    selector = {
      app = "hello-world"
    }

    port {
      protocol    = "TCP"
      port        = 80
      target_port = 8080
    }

    type = "LoadBalancer"
  }
}
```

### Verification

To verify that the Python app is accessible, use the `kubectl get svc` command to get the external IP address of the LoadBalancer service and then access it via a web browser or `curl`.

```sh
kubectl get svc hello-world-service
```

Look for the `EXTERNAL-IP` in the output and navigate to `http://<EXTERNAL-IP>` in the browser.

---
