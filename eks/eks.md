---
description: 'Update : 2024.01.27'
---

# EKS 설치

eks 설치를 위해서 Tool들을 설치해야 합니다. 사전에 github에서 복제 해 둔 shell을 아래와 같이 실행합니다.

```
~/environment/useful-shell/c9-tool-set.sh

```

EKS Cluster를 VPC01 에 설치하기 위해 eksctl yaml을 생성합니다.

```
~/environment/gwlb_anfw/anfw/eksctl_cluster_2az.sh

```

생성된 eksctl yaml 로 VPC01에 EKS Cluster를 생성합니다.

```
eksctl create cluster --config-file=/home/ec2-user/environment/ekscluster01.yaml

```
