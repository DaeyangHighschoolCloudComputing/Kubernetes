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

### Software and Framework to be install and download.
- [eksctl](https://www.eksctl.io)
- [kubectl](https://kubernetes.io/docs/home/)
- [awscliv2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

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
sudo echo -e "Skills2024**\nSkills2024**" | sudo passwd ec2-user
sudo echo -e "AKIA55IMW4OZ2EXYAWGQ\nC/nk7hmoQ7TIjtTWEvZ+tego+8+8Or3Ax3/scuwW\nap-northeast-2\njson" | aws configure
```
