
# 🎯 Advanced DevOps & Deployment

## Table of Contents
- [Infrastructure as Code (IaC)](#infrastructure-as-code-iac)
- [Kubernetes Advanced Concepts](#kubernetes-advanced-concepts)
- [Service Mesh](#service-mesh)
- [GitOps](#gitops)
- [Observability](#observability)
- [Security & Compliance](#security--compliance)
- [Multi-Cloud Strategy](#multi-cloud-strategy)
- [Disaster Recovery](#disaster-recovery)
- [Cost Optimization](#cost-optimization)

---

## 🏗️ Infrastructure as Code (IaC)

### Terraform

```hcl
# main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.20"
    }
  }
  
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "infrastructure/terraform.tfstate"
    region = "us-west-2"
  }
}

# VPC Module
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "main-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-west-2a", "us-west-2b", "us-west-2c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  enable_vpn_gateway = true
  
  tags = {
    Environment = var.environment
    Terraform   = "true"
  }
}

# EKS Cluster
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  
  cluster_name    = "my-cluster"
  cluster_version = "1.27"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  
  eks_managed_node_groups = {
    main = {
      desired_capacity = 3
      max_capacity     = 10
      min_capacity     = 1
      
      instance_types = ["t3.medium"]
      
      k8s_labels = {
        Environment = var.environment
        NodeGroup   = "main"
      }
    }
  }
}

# Application Load Balancer
resource "aws_lb" "main" {
  name               = "main-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = module.vpc.public_subnets
  
  enable_deletion_protection = false
  
  tags = {
    Environment = var.environment
  }
}
```

### Pulumi (Code-based IaC)

```typescript
// index.ts
import * as aws from "@pulumi/aws";
import * as awsx from "@pulumi/awsx";
import * as k8s from "@pulumi/kubernetes";

// Create VPC
const vpc = new awsx.ec2.Vpc("main-vpc", {
    cidrBlock: "10.0.0.0/16",
    numberOfAvailabilityZones: 3,
    enableDnsHostnames: true,
    enableDnsSupport: true,
});

// Create EKS cluster
const cluster = new awsx.eks.Cluster("my-cluster", {
    vpcId: vpc.vpcId,
    subnetIds: vpc.privateSubnetIds,
    version: "1.27",
    nodeGroups: {
        main: {
            instanceType: "t3.medium",
            desiredCapacity: 3,
            minSize: 1,
            maxSize: 10,
        },
    },
});

// Deploy application
const app = new k8s.apps.v1.Deployment("web-app", {
    metadata: { namespace: "default" },
    spec: {
        replicas: 3,
        selector: { matchLabels: { app: "web-app" } },
        template: {
            metadata: { labels: { app: "web-app" } },
            spec: {
                containers: [{
                    name: "web-app",
                    image: "nginx:1.21",
                    ports: [{ containerPort: 80 }],
                }],
            },
        },
    },
}, { provider: cluster.provider });

export const kubeconfig = cluster.kubeconfig;
export const vpcId = vpc.vpcId;
```

---

## ☸️ Kubernetes Advanced Concepts

### Custom Resource Definitions (CRDs)

```yaml
# custom-resource.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: webapps.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              replicas:
                type: integer
                minimum: 1
                maximum: 10
              image:
                type: string
              resources:
                type: object
                properties:
                  requests:
                    type: object
                    properties:
                      cpu:
                        type: string
                      memory:
                        type: string
          status:
            type: object
            properties:
              phase:
                type: string
              readyReplicas:
                type: integer
  scope: Namespaced
  names:
    plural: webapps
    singular: webapp
    kind: WebApp
```

### Operators

```go
// controllers/webapp_controller.go
package controllers

import (
    "context"
    appsv1 "k8s.io/api/apps/v1"
    corev1 "k8s.io/api/core/v1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/runtime"
    ctrl "sigs.k8s.io/controller-runtime"
    "sigs.k8s.io/controller-runtime/pkg/client"
    
    examplev1 "github.com/myorg/webapp-operator/api/v1"
)

type WebAppReconciler struct {
    client.Client
    Scheme *runtime.Scheme
}

func (r *WebAppReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    // Fetch the WebApp instance
    webapp := &examplev1.WebApp{}
    err := r.Get(ctx, req.NamespacedName, webapp)
    if err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }
    
    // Create Deployment
    deployment := &appsv1.Deployment{
        ObjectMeta: metav1.ObjectMeta{
            Name:      webapp.Name,
            Namespace: webapp.Namespace,
        },
        Spec: appsv1.DeploymentSpec{
            Replicas: &webapp.Spec.Replicas,
            Selector: &metav1.LabelSelector{
                MatchLabels: map[string]string{
                    "app": webapp.Name,
                },
            },
            Template: corev1.PodTemplateSpec{
                ObjectMeta: metav1.ObjectMeta{
                    Labels: map[string]string{
                        "app": webapp.Name,
                    },
                },
                Spec: corev1.PodSpec{
                    Containers: []corev1.Container{
                        {
                            Name:  "webapp",
                            Image: webapp.Spec.Image,
                            Resources: webapp.Spec.Resources,
                        },
                    },
                },
            },
        },
    }
    
    // Set controller reference
    ctrl.SetControllerReference(webapp, deployment, r.Scheme)
    
    // Create or update deployment
    err = r.Client.Create(ctx, deployment)
    if err != nil {
        return ctrl.Result{}, err
    }
    
    return ctrl.Result{}, nil
}
```

---

## 🕸️ Service Mesh

### Istio Configuration

```yaml
# istio-gateway.yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: webapp-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: webapp-tls
    hosts:
    - webapp.example.com

---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: webapp-vs
spec:
  hosts:
  - webapp.example.com
  gateways:
  - istio-system/webapp-gateway
  http:
  - match:
    - uri:
        prefix: /api/v1
    route:
    - destination:
        host: backend-service
        port:
          number: 8080
      weight: 90
    - destination:
        host: backend-service-canary
        port:
          number: 8080
      weight: 10
  - match:
    - uri:
        prefix: /
    route:
    - destination:
        host: frontend-service
        port:
          number: 80

---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: backend-destination
spec:
  host: backend-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        maxRequestsPerConnection: 2
    circuitBreaker:
      consecutiveErrors: 3
      interval: 30s
      baseEjectionTime: 30s
    retryPolicy:
      attempts: 3
      perTryTimeout: 10s
```

### Traffic Management

```yaml
# canary-deployment.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: webapp-rollout
spec:
  replicas: 10
  strategy:
    canary:
      analysis:
        templates:
        - templateName: success-rate
        startingStep: 2
        args:
        - name: service-name
          value: webapp-service
      canaryService: webapp-canary-service
      stableService: webapp-stable-service
      trafficRouting:
        istio:
          virtualService:
            name: webapp-vs
            routes:
            - primary
          destinationRule:
            name: webapp-destination
            canarySubsetName: canary
            stableSubsetName: stable
      steps:
      - setWeight: 5
      - pause: {duration: 10m}
      - setWeight: 20
      - pause: {duration: 10m}
      - setWeight: 40
      - pause: {duration: 10m}
      - setWeight: 60
      - pause: {duration: 10m}
      - setWeight: 80
      - pause: {duration: 10m}
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
        image: webapp:v2.0.0
        ports:
        - containerPort: 8080
```

---

## 🔄 GitOps

### ArgoCD Application

```yaml
# argocd-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: webapp-production
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/webapp-manifests
    targetRevision: HEAD
    path: overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

---
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: webapp-environments
  namespace: argocd
spec:
  generators:
  - git:
      repoURL: https://github.com/myorg/webapp-manifests
      revision: HEAD
      directories:
      - path: overlays/*
  template:
    metadata:
      name: '{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/webapp-manifests
        targetRevision: HEAD
        path: '{{path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### Flux Configuration

```yaml
# flux-system.yaml
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: GitRepository
metadata:
  name: webapp-source
  namespace: flux-system
spec:
  interval: 1m
  ref:
    branch: main
  url: https://github.com/myorg/webapp-manifests

---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta1
kind: Kustomization
metadata:
  name: webapp-production
  namespace: flux-system
spec:
  interval: 10m
  path: "./overlays/production"
  prune: true
  sourceRef:
    kind: GitRepository
    name: webapp-source
  validation: client
  healthChecks:
  - apiVersion: apps/v1
    kind: Deployment
    name: webapp
    namespace: production
```

---

## 📊 Observability

### Prometheus Configuration

```yaml
# prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
    
    rule_files:
    - "/etc/prometheus/rules/*.yml"
    
    alerting:
      alertmanagers:
      - static_configs:
        - targets:
          - alertmanager:9093
    
    scrape_configs:
    - job_name: 'kubernetes-apiservers'
      kubernetes_sd_configs:
      - role: endpoints
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https
    
    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
    
    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
```

### Grafana Dashboards

```json
{
  "dashboard": {
    "title": "Application Performance Dashboard",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (service)",
            "legendFormat": "{{service}}"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "graph", 
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) by (service) / sum(rate(http_requests_total[5m])) by (service)",
            "legendFormat": "{{service}} Error Rate"
          }
        ]
      },
      {
        "title": "Response Time P95",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))",
            "legendFormat": "{{service}} P95"
          }
        ]
      }
    ]
  }
}
```

---

## 🔒 Security & Compliance

### Pod Security Standards

```yaml
# pod-security-policy.yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted-psp
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: webapp-network-policy
spec:
  podSelector:
    matchLabels:
      app: webapp
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to: []
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

### OPA Gatekeeper Policies

```yaml
# constraint-template.yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredlabels
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredLabels
      validation:
        properties:
          labels:
            type: array
            items:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredlabels
        
        violation[{"msg": msg}] {
          required := input.parameters.labels
          provided := input.review.object.metadata.labels
          missing := required[_]
          not provided[missing]
          msg := sprintf("Missing required label: %v", [missing])
        }

---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: must-have-env-label
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment", "ReplicaSet"]
  parameters:
    labels: ["environment", "team", "version"]
```

---

## ☁️ Multi-Cloud Strategy

### Crossplane Configuration

```yaml
# crossplane-composition.yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: xpostgresqlinstances.aws.database.example.com
  labels:
    provider: aws
    service: postgresql
spec:
  compositeTypeRef:
    apiVersion: database.example.com/v1alpha1
    kind: XPostgreSQLInstance
  resources:
  - name: rdsinstance
    base:
      apiVersion: rds.aws.crossplane.io/v1alpha1
      kind: RDSInstance
      spec:
        forProvider:
          region: us-west-2
          dbInstanceClass: db.t3.micro
          engine: postgres
          engineVersion: "13.7"
          autoMinorVersionUpgrade: true
          publiclyAccessible: false
          storageEncrypted: true
        writeConnectionSecretsToNamespace: crossplane-system
    patches:
    - type: FromCompositeFieldPath
      fromFieldPath: spec.parameters.size
      toFieldPath: spec.forProvider.dbInstanceClass
      transforms:
      - type: map
        map:
          small: db.t3.micro
          medium: db.t3.small
          large: db.t3.medium

---
apiVersion: database.example.com/v1alpha1
kind: XPostgreSQLInstance
metadata:
  name: my-db
spec:
  parameters:
    size: medium
    region: us-west-2
  compositionRef:
    name: xpostgresqlinstances.aws.database.example.com
  writeConnectionSecretsToNamespace: default
```

---

## 🚨 Disaster Recovery

### Velero Backup Configuration

```yaml
# velero-backup.yaml
apiVersion: velero.io/v1
kind: Backup
metadata:
  name: daily-backup
  namespace: velero
spec:
  includedNamespaces:
  - production
  - staging
  excludedResources:
  - events
  - events.events.k8s.io
  ttl: 720h
  storageLocation: default
  volumeSnapshotLocations:
  - default

---
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: daily-backup-schedule
  namespace: velero
spec:
  schedule: "0 2 * * *"
  template:
    includedNamespaces:
    - production
    ttl: 720h
    storageLocation: default

---
apiVersion: velero.io/v1
kind: RestoreRequest
metadata:
  name: restore-from-backup
  namespace: velero
spec:
  backupName: daily-backup-20231201120000
  includedNamespaces:
  - production
  restorePVs: true
```

### Cluster Autoscaling

```yaml
# cluster-autoscaler.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      containers:
      - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.21.0
        name: cluster-autoscaler
        resources:
          limits:
            cpu: 100m
            memory: 300Mi
          requests:
            cpu: 100m
            memory: 300Mi
        command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-cluster
        env:
        - name: AWS_REGION
          value: us-west-2
```

---

## 💰 Cost Optimization

### Resource Quotas and Limits

```yaml
# resource-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "100"
    requests.memory: 200Gi
    limits.cpu: "200"
    limits.memory: 400Gi
    persistentvolumeclaims: "10"
    count/deployments.apps: "50"
    count/services: "20"

---
apiVersion: v1
kind: LimitRange
metadata:
  name: production-limits
  namespace: production
spec:
  limits:
  - default:
      cpu: 200m
      memory: 256Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container
  - max:
      cpu: "2"
      memory: 4Gi
    min:
      cpu: 50m
      memory: 64Mi
    type: Container
```

### Cost Monitoring

```yaml
# kubecost-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kubecost-cost-analyzer-config
data:
  kubecost-token: "your-kubecost-token"
  cluster-name: "production-cluster"
  currency: "USD"
  log-level: "info"
  
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubecost-cost-analyzer
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cost-analyzer
  template:
    metadata:
      labels:
        app: cost-analyzer
    spec:
      containers:
      - name: cost-analyzer-frontend
        image: kubecost/frontend:latest
        ports:
        - containerPort: 9090
        env:
        - name: GET_HOSTS_FROM
          value: dns
      - name: cost-analyzer-server
        image: kubecost/server:latest
        ports:
        - containerPort: 9003
        env:
        - name: KUBECOST_TOKEN
          valueFrom:
            configMapKeyRef:
              name: kubecost-cost-analyzer-config
              key: kubecost-token
```

---

## 📚 Summary

### Advanced DevOps Capabilities:
- **Infrastructure as Code**: Terraform, Pulumi, CloudFormation
- **Container Orchestration**: Kubernetes operators, custom resources
- **Service Mesh**: Istio, Linkerd for microservices communication
- **GitOps**: ArgoCD, Flux for declarative deployments
- **Observability**: Prometheus, Grafana, distributed tracing
- **Security**: Pod security policies, network policies, compliance
- **Multi-Cloud**: Crossplane for cloud abstraction
- **Disaster Recovery**: Backup strategies, chaos engineering
- **Cost Optimization**: Resource quotas, monitoring, optimization

### Enterprise Best Practices:
1. **Implement GitOps workflows** for all deployments
2. **Use Infrastructure as Code** for reproducible environments
3. **Adopt service mesh** for microservices architectures
4. **Implement comprehensive monitoring** and alerting
5. **Enforce security policies** at all levels
6. **Plan for disaster recovery** and business continuity
7. **Monitor and optimize costs** continuously
8. **Automate compliance** and security scanning
9. **Use canary deployments** for risk mitigation
10. **Implement chaos engineering** for resilience testing

### Next Steps:
1. Study cloud-native security patterns
2. Learn about serverless and edge computing
3. Explore AI/ML operations (MLOps)
4. Study compliance frameworks (SOC2, PCI-DSS)
5. Learn advanced Kubernetes patterns and operators
