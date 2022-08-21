# Kubernetes / k8s ☸
<img src="https://github.com/kubernetes/kubernetes/raw/master/logo/logo.png" width="100">

----

Kubernetes, also known as K8s, is an open source system for managing [containerized applications]
across multiple hosts. It provides basic mechanisms for deployment, maintenance,
and scaling of applications.

Kubernetes builds upon a decade and a half of experience at Google running
production workloads at scale using a system called [Borg],
combined with best-of-breed ideas and practices from the community.

Kubernetes is hosted by the Cloud Native Computing Foundation ([CNCF]).
If your company wants to help shape the evolution of
technologies that are container-packaged, dynamically scheduled,
and microservices-oriented, consider joining the CNCF.
For details about who's involved and how Kubernetes plays a role,
read the CNCF [announcement].

----
[announcement]: https://cncf.io/news/announcement/2015/07/new-cloud-native-computing-foundation-drive-alignment-among-container
[Borg]: https://research.google.com/pubs/pub43438.html
[CNCF]: https://www.cncf.io/about
[containerized applications]: https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/

#### files that used in this project.
- namespace.yaml
- cluster.yaml
- deployment.yaml
- limit.yaml
- service.yaml
- ingress.yaml
- controller.yaml
- cwinsight.yaml
- scaler.yaml
- autoscaler.yaml

### Software and Framework to be install and download.
- [eksctl](https://www.eksctl.io)
- [kubectl](https://kubernetes.io/docs/home/)
- [awscliv2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

## Kubernetes Project



##### userdata that was used in this project

```
#!/bin/bash
apt install -y unzip jq curl wget
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.23.7/2022-06-29/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
cat <<EOF >>/etc/ssh/sshd_config
Port 2220
EOF
sed -i "/PasswordAuthentication/d" /etc/ssh/sshd_config
systemctl restart sshd
sudo echo -e "Skills2024**\nSkills2024**" | sudo passwd ubuntu
```
```
sudo echo -e "{password}\n{password}" | sudo passwd {user}
```

`-e` is to recognize `\n` as new line.

`sudo` is **root** access for Ubuntu.

The above command passes the password and a new line, two times, to `passwd`, which is what passwd requires.

If not using variables, I think this probably works.

##### example: (amazon linux 2)

```
sudo echo -e "Daeyang1@#\nDaeyang1@#" | sudo passwd ec2-user
```

## Manifest Files: Kubernetes Objects
#### - cluster.yaml
```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: worldskills-cloud-cluster
  region: ap-northeast-2
  version: "1.22"

vpc:
  id: vpc-0f8fd1f154d4c39c4
  subnets:
    private:
      worldskills-cloud-priv-sn-a:
        id: subnet-0c92699bef1d18422
      worldskills-cloud-priv-sn-c:
        id: subnet-091a1baf6da30ed33

managedNodeGroups:
  - name: worldskills-cloud-node
    instanceName: worldskills-cloud-node
    instanceType: t3.medium
    desiredCapacity: 2
    volumeSize: 80
    privateNetworking: true
    iam:
      withAddonPolicies:
        imageBuilder: true
        autoScaler: true
        awsLoadBalancerController: true
        cloudWatch: true
      attachPolicyARNs:
        - arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
        - arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy
        - arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
        - arn:aws:iam::aws:policy/ElasticLoadBalancingFullAccess
        - arn:aws:iam::aws:policy/AmazonS3FullAccess

fargateProfiles:
  - name: worldskills-cloud-ws-profile
    selectors:
      - namespace: worldskills-ns

secretsEncryption:
  keyARN: arn:aws:kms:ap-northeast-2:956193760179:key/d42fbb4c-6c77-4aab-af4f-478211c948f3

cloudWatch:
  clusterLogging:
    enableTypes: ["*"]
```
use ```eksctl``` to launch eks cluster and nodegroup

``` eksctl create cluster -f cluster.yaml ```

creation of cluster can take up to **15-20 minutes** so be patient...

After creating cluster try to check if our nodes are made by: ``` kubectl get nodes ```

if it says ```The connection to the server localhost:8080 was refused - did you specify the right host or port?``` it means you need to connect to eks cluster

```aws eks --region {region} update-kubeconfig --name {cluster name}```

#### - namespace.yaml
```
apiVersion: v1
kind: Namespace

metadata:
  name: worldskills-ns
```
``` kubectl apply -f namespace.yaml ```

**또는**

```kubectl create ns {네임스페이스 이름}``` or ```kubectl create namespace {네임스페이스 이름}```

In Kubernetes, namespaces provides a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc).

#### - deployment.yaml
```
apiVersion: apps/v1
kind: Deployment

metadata:
  name: worldskills-cloud-deployment
  namespace: worldskills-ns
  labels:
    app: worldskills
spec:
  replicas: 2
  selector:
    matchLabels:
      app: worldskills
  template:
    metadata:
      labels:
        app: worldskills
    spec:
      containers:
      - name: app-container
        image: jeonilshin/task1:latest
        ports:
        - name: http
          containerPort: 3000
          protocol: TCP
```

```kubectl apply -f deployment.yaml```

A Deployment provides declarative updates for Pods and ReplicaSets.

You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

#### - limit.yaml
```
apiVersion: v1
kind: LimitRange

metadata:
  name: worldskills-cloud-limit
  namespace: worldskills-ns
spec:
  limits:
  - max:
      cpu: "2"
      memory: "512Mi"
    min:
      cpu: "1"
      memory: "256Mi"
    type: Pod
```

```kubectl apply -f limit.yaml```

By default, containers run with unbounded compute resources on a Kubernetes cluster. With resource quotas, cluster administrators can restrict resource consumption and creation on a namespace basis. Within a namespace, a Pod or Container can consume as much CPU and memory as defined by the namespace's resource quota. There is a concern that one Pod or Container could monopolize all available resources. A LimitRange is a policy to constrain resource allocations (to Pods or Containers) in a namespace.

#### - service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: worldskills-cloud-service
  namespace: worldskills-ns
  annotations:
    alb.ingress.kubernetes.io/healthcheck-path: "/health"
spec:
  selector:
    app: worldskills
  ports:
    - protocol: TCP
      port: 3000
      name: http
```

```kubectl apply -f service.yaml```

An abstract way to expose an application running on a set of Pods as a network service.
With Kubernetes you don't need to modify your application to use an unfamiliar service discovery mechanism. Kubernetes gives Pods their own IP addresses and a single DNS name for a set of Pods, and can load-balance across them.

## ALB Ingress Controller
1. First, create **IAM OpenID Connect(OIDC) identity provider** for the cluster. **IAM OIDC provider** must exist in the cluster in order for objects created by Kubernetes to use service account which purpose is to authenticate to API Server ro external services.
```
eksctl utils associate-iam-oidc-provider \
    --region ap-northeast-2 \
    --cluster worldskills-cloud-cluster \
    --approve
```
2. Create an IAM Policy to grant to the AWS Load Balancer Controller
```
wget https://jeonilshin.s3.ap-northeast-2.amazonaws.com/iam_policy.json && aws iam create-policy \
--policy-name AWSLoadBalancerControllerIAMPolicy \
--policy-document file://iam_policy.json
```
3. Create ServiceAccount for AWS Load Balancer Controller
```
eksctl create iamserviceaccount \
    --cluster worldskills-cloud-cluster \
    --namespace kube-system \
    --name aws-load-balancer-controller \
    --attach-policy-arn arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
```
4. Add AWS Load Balancer controller to the cluster. First, install **cert-manager**  to insert the certificate configuration into the Webhook. **Cert-manager** is an open source that automatically provisions and manages TLS certificates within a Kubernetes cluster.
```
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.5.3/cert-manager.yaml
```
#### - controller.yaml
```
wget https://jeonilshin.s3.ap-northeast-2.amazonaws.com/controller.yaml
```

```kubectl apply -f controller.yaml```
#### ALB Controller 정상적으로 실해되는지 확인하기
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```
```
kubectl get sa aws-load-balancer-controller -n kube-system -o yaml
```
##### logs
```
kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o "aws-load-balancer[a-zA-Z0-9-]+")
```

### - ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: worldskills-cloud-ingress
  namespace: worldskills-ns
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/load-balancer-name: worldskills-cloud-alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/subnets: worldskills-cloud-pub-sn-a, worldskills-cloud-pub-sn-c
    alb.ingress.kubernetes.io/ip-address-type: dualstack
    alb.ingress.kubernetes.io/target-group-attributes: load_balancing.algorithm.type=least_outstanding_requests
spec:
  rules:
  - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: worldskills-cloud-service
                port:
                  number: 3000
```

```kubectl apply -f ingress.yaml```
#### alb 잘 실행되는지 확인하기
``` kubectl get ingress -n worldskills-ns```

![1](https://user-images.githubusercontent.com/86287920/185655862-e4fe2c3d-20d7-43a6-91ba-8cf123bae4f0.PNG)

#### - cwinsight.yaml
```
curl https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluentd-quickstart.yaml -o cwinsight.yaml
```
```
sed -i "s/{{cluster_name}}/worldskills-cloud-cluster/;s/{{region_name}}/ap-northeast-2/" ./cwinsight.yaml
```


```kubectl apply -f cwinsight.yaml```

## Autoscaling
#### Apply HPA
1. Create **metrics server**. **Metrics Server** aggregates resource usage data across the Kubernetes cluster. Collect metrics such as the CPU and memory usage of the worker node or container through kubelet installed on each worker node.
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
### - scaler.yaml
```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: worldskills-cloud-cluster
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: worldskills-cloud-deployment
  minReplicas: 2
  maxReplicas: 5
  targetCPUUtilizationPercentage: 20
```

```kubectl apply -f scaler.yaml```

2. In **Auto Scaling Groups** page, click ASG applied in worker node, and update **Group details** value same as below.

![2](https://user-images.githubusercontent.com/86287920/185660226-d42b6746-3dcf-4fc1-a8fc-c9443a2eeb97.png)

![3](https://user-images.githubusercontent.com/86287920/185660246-69b4e84c-da62-45cc-ab6e-88928c5ec82e.png)

![4](https://user-images.githubusercontent.com/86287920/185660249-76245a7b-801a-406d-a930-ff1e8d234361.png)

### - autoscaler.yaml
```
wget https://jeonilshin.s3.ap-northeast-2.amazonaws.com/autoscaler.yaml
```

```kubectl apply -f autoscaler.yaml```
