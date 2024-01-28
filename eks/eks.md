---
description: 'Update : 2024.01.27'
---

# EKS 설치

아래와 같은 구성으로 VPC01 워크로드에 EKS Cluster와 Istio를 설치할 것입니다.



### EKS 설치 준비

eks 설치를 위해서 Tool들을 설치해야 합니다. 사전에 github에서 복제 해 둔 shell을 아래와 같이 실행합니다.

```
~/environment/useful-shell/c9-tool-set.sh

```

EKS Cluster를 VPC01 에 설치하기 위해 eksctl yaml을 생성합니다.

```
~/environment/gwlb_anfw/anfw/eksctl_cluster_2az.sh

```



### EKS 설치 및 설치 확인

생성된 eksctl yaml 로 VPC01에 EKS Cluster를 생성합니다.

```
eksctl create cluster --config-file=/home/ec2-user/environment/ekscluster01.yaml

```

Amazon EKS Cluster는 설치 시간이 15분 \~20분 소요됩니다.

아래와 같은 메세지가 출력되면 모두 정상적으로 설치된 것입니다.

```
2024-01-27 07:39:54 [✔]  EKS cluster "C1" in "ap-northeast-2" region is ready
```

설치가 왼료되면 Amazon EKS 콘솔에서 정보를 확인하기 위해, 최초 생성한 계정이 정보에 권한을 부여합니다.

Cloud9에서 아래 명령을 복사해서 삽입합니다. 현재 AWS Console에 Login 한 IAM User가 정확한지 확인합니다. 아래 예제에서 User ID는 "user01" 입니다.

```
## USER_ID의 값은 현재 콘솔에서의 user id
export USER_ID=user01
echo "export USER_ID=${USER_ID}" | tee -a ~/.bash_profile
source ~/.bash_profile

## ACCOUNT ID 확인
echo $ACCOUNT_ID

## eksctl로 IAM ID를 Kubernetes 의 System Master 권한을 부여합니다.
eksctl create iamidentitymapping \
  --cluster ${CLUSTER1_NAME} \
  --arn arn:aws:iam::${ACCOUNT_ID}:user/${USER_ID}\
  --username ${USER_ID} \
  --group system:masters
  
```

EKS 콘솔에서 아래와 같은 정보가 출력되었는지 확인해 봅니다.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

