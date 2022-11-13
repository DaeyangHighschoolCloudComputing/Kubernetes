<h1 align="center"> Kubernetes / k8s ☸</h1>
<p align="center"><img src="https://github.com/kubernetes/kubernetes/raw/master/logo/logo.png" width="100"></p>

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
sed -i "s/PasswordAuthentication no/PasswordAuthentication yes/g" /etc/ssh/sshd_config
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
#### - components.yaml
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl edit deploy -n kube-system metrics-server
```
![image](https://user-images.githubusercontent.com/86287920/201477488-955c3f7c-b32d-442c-8fee-d8c21c2aedc4.png)
![image](https://user-images.githubusercontent.com/86287920/201477492-36db425f-9639-44b1-942e-d178e98ab764.png)

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

## RBAC(Role-based access control)
#### - role.yaml
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: worldskills-cloud-control-role-user
  namespace: worldskills-ns
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: worldskills-cloud-control-role
  namespace: worldskills-ns
rules:
  - apiGroups:
      - ""
      - "apps"
      - "batch"
      - "extensions"
    resources:
      - "configmaps"
      - "cronjobs"
      - "deployments"
      - "events"
      - "ingresses"
      - "jobs"
      - "pods"
      - "pods/attach"
      - "pods/exec"
      - "pods/log"
      - "pods/portforward"
      - "secrets"
      - "services"
    verbs:
      - "create"
      - "delete"
      - "describe"
      - "get"
      - "list"
      - "patch"
      - "update"
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: worldskills-cloud-control-rolebinding
  namespace: worldskills-ns
subjects:
- kind: ServiceAccount
  name: worldskills-cloud-control-role-user
  namespace: worldskills-ns
roleRef:
  kind: Role
  name: worldskills-cloud-control-role
  apiGroup: rbac.authorization.k8s.io
```
aws-auth ConfigMap에 IAM역활을 RBAC 역할 및 그룹에 매핑해준다.

```
eksctl create iamidentitymapping --cluster worldskills-cloud-cluster --arn arn:aws:iam::계정ID:role/worldskills-cloud-control-role --username worldskills-cloud-control-role-user
```

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
      serviceAccountName: worldskills-cloud-eks-sa  #Create a sa with s3 access
      containers:
      - name: app-container
        image: jeonilshin/task1:latest
        env:
          - name: s3_bucket
            value: "버킷이름" 

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
spec:
  selector:
    app: worldskills
  ports:
  - port: 80
    targetPort: 3000
    protocol: TCP
  type: NodePort
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
```
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master
kubectl apply -k github.com/aws/eks-charts/stable/aws-load-balancer-controller/crds?ref=master
```
2. Create an IAM Policy to grant to the AWS Load Balancer Controller
```
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.3.1/docs/install/iam_policy.json
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
3. Create ServiceAccount for AWS Load Balancer Controller
```
eksctl create iamserviceaccount \
    --cluster=worldskills-cloud-cluster \
    --namespace=kube-system \
    --name=aws-load-balancer-controller \
    --attach-policy-arn=arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
```
4. Add AWS Load Balancer controller to the cluster.
(install helm)

```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```
```
helm repo add eks https://aws.github.io/eks-charts
helm repo update
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
    -n kube-system \
    --set clusterName=worldskills-cloud-eks-cluster \
    --set serviceAccount.create=false \
    --set serviceAccount.name=aws-load-balancer-controller \
    --set region=ap-northeast-2
```
```
kubectl get deployment -n kube-system aws-load-balancer-controller
```
```
kubectl get sa aws-load-balancer-controller -n kube-system -o yaml
```
##### aws controller logs
```
kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o "aws-load-balancer[a-zA-Z0-9-]+")
```

### - ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: "worldskills-cloud-ingress"
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/group.name: worldskills-cloud-tg
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/load-balancer-name: worldskills-cloud-alb
    alb.ingress.kubernetes.io/subnets: worldskills-cloud-pub-sn-a, worldskills-cloud-pub-sn-c
    alb.ingress.kubernetes.io/algorithm-type: least-connection

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
                  number: 80
```

```kubectl apply -f ingress.yaml --validate=false```
#### alb 잘 실행되는지 확인하기
``` kubectl get ingress -n worldskills-ns```

![1](https://user-images.githubusercontent.com/86287920/185655862-e4fe2c3d-20d7-43a6-91ba-8cf123bae4f0.PNG)

## AutoScaling
```
curl -O https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

```vim cluster-autoscaler-autodiscover.yaml```

![image](https://user-images.githubusercontent.com/86287920/201500846-5c8d41e8-d192-4c9c-9c2e-5b6874fa8b73.png)

![image](https://user-images.githubusercontent.com/86287920/201500849-b5e574de-de9e-4b4b-8389-8a401c3ceba6.png)

![image](https://user-images.githubusercontent.com/86287920/201500850-b98e1d01-8021-4701-80e2-f9604ca1061c.png)

``` kubectl apply -f cluster-autoscaler-autodiscover.yaml ```
#### - hpa.yaml
```
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: worldskills-scaler
  namespace: worldskills-ns
  labels:
    app: worldskills
spec:
  minReplicas: 2
  maxReplicas: 20

  metrics:
  - resource:
      name: cpu
      targetAverageUtilization: 20
    type: Resource

  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: worldskills-cloud-deployment
```
``` kubectl apply -f hpa.yaml ``` 

#### - cwinsight.yaml
```
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/master/k8s-yaml-templates/cloudwatch-namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/master/k8s-yaml-templates/cwagent-kubernetes-monitoring/cwagent-serviceaccount.yaml
curl -O https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/master/k8s-yaml-templates/cwagent-kubernetes-monitoring/cwagent-configmap.yaml
```

```vim cwagent-configmap.yaml```
![image](https://user-images.githubusercontent.com/86287920/201500941-7de31fce-1b8b-4624-84b2-0ee194562b89.png)

```
kubectl apply -f cwagent-configmap.yaml
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/master/k8s-yaml-templates/cwagent-kubernetes-monitoring/cwagent-daemonset.yaml
```
