---
description: 'Update : 2024-01-27'
---

# 워크로드 설치

### **Amazon EKS 클러스터에 AWS Load Balancer Controller 배포**

EKS Workload 구성을 위해 아래 Git 을 복제합니다.

```
cd ~/environment
git clone https://github.com/whchoi98/myeks

```

IAM Policy를 생성합니다.

AWS API를 호출할 수 있는 AWS Load Balancer Controller의 IAM 정책을 다운로드합니다.

```
cd ~/environment
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json

```

IAM Role을 생성합니다.

AWS Load Balancer Controller의 `kube-system` 네임스페이스에 `aws-load-balancer-controller`라는 Kubernetes Service Account을 생성하고 IAM Role의 이름으로 Kubernetes Service Account에 주석을 답니다.

`eksctl` 또는 AWS CLI 및 `kubectl`을 사용하여 IAM 역할 및 Kubernetes 서비스 계정을 생성할 수 있습니다.

아래는 `eksctl` 을 사용하는 방법입니다.

```
export CLUSTER1_NAME="C1"
eksctl create iamserviceaccount \
  --cluster=${CLUSTER1_NAME} \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
  
```

Helm 기반으로 설치를 진행합니다.

```
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=${CLUSTER1_NAME}  \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
  
```

정상적으로 컨트롤러가 설치되었는지 확인해 봅니다.

```
kubectl get deployment -n kube-system aws-load-balancer-controller

```

다음과 같은 결과가 출력되면 AWS Load Balancer Controller가 정상으로 구성되었습니다.

```
$   kubectl get deployment -n kube-system aws-load-balancer-controller
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
aws-load-balancer-controller   2/2     2            2           9s
```

### NLB 기반의 워크로드 설치

테스트를 하기 위한 간단한 K8s Sample App을 배포합니다.

```
kubectl apply -f ~/environment/myeks/network-test/nlb-test-03.yaml

```

Sample App에 대한 Service Object를 생성합니다.

```
cat <<EoF > ~/environment/myeks/network-test/nlb-test-03-svc.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nlb-test-03-svc
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-name: "nlb-test-03-svc"
    service.beta.kubernetes.io/aws-load-balancer-internal: "true"
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb-ip"
    service.beta.kubernetes.io/aws-load-balancer-private-ipv4-addresses: "10.1.21.201, 10.1.22.201"
    service.beta.kubernetes.io/aws-load-balancer-subnets: "${C1_PrivateSubnet01}, ${C1_PrivateSubnet02}"
  namespace: nlb-test-03
spec:
  externalTrafficPolicy: Local
  selector:
    app: nlb-test-03
  type: LoadBalancer
  ports:
   -  protocol: TCP
      port: 80
      targetPort: 80
EoF

kubectl apply -f  ~/environment/myeks/network-test/nlb-test-03-svc.yaml

```

EKS Cluster가 배포된 VPC01의 Private Subent에 , menifast 파일로 NLB를 배포합니다.

* Object : Service
* Loadbalancer Type  : NLB
* NLB Register-target : Private IP Address (10.1.21.201, 10.1.22.201)
* NLB Target Type : IP Address

정상적으로 배포되었는지 확인합니다.

```
kubectl -n nlb-test-03 get pods,svc

```

아래와 같이 출력됩니다.

```
$ kubectl -n nlb-test-03 get pods,svc
NAME                               READY   STATUS    RESTARTS   AGE
pod/nlb-test-03-58cff44f7b-hjl2b   1/1     Running   0          6h24m
pod/nlb-test-03-58cff44f7b-tz4lt   1/1     Running   0          6h24m
생략...

NAME                      TYPE           CLUSTER-IP      EXTERNAL-IP                                                                          PORT(S)        AGE
service/nlb-test-03-svc   LoadBalancer   172.20.194.83   k8s-nlbtest0-nlbtest0-aff40e2f5e-978ee89e8ab10bcf.elb.ap-northeast-2.amazonaws.com   80:31915/TCP   6h13m
```

### Inspection VPC에서 ALB 구성하기

이제 외부에서 ANFW를 통과해서, VPC01의 Kubernetes 워크로드에 연결할 수 있도록 Inspection VPC (ANFW-N2SVPC)에 ALB를 배포하고 구성합니다.

아래 명령어를 수행해서 자동으로 배포합니다.

```
~/environment/gwlb_anfw/anfw/alb_for_eks.sh

```

ALB가 정상적으로 배포되었는지 확인합니다.

```
while true; do aws elbv2 describe-load-balancers --load-balancer-arns $ALB_FOR_EKS_ARN | jq -r '.LoadBalancers[].State.Code'; sleep 10; done

```

3분 정도 후면 "Active" 결과가 표기 됩니다.

이제 외부에서 VPC01 의 EKS POD로 접근되는지 아래 명령으로 확인합니다.

```
while true; do curl http://$EKS_Sample_APP_URL ;sleep 1;done

```

앞서 ANFW LAB에서 Flow Log에 대해서 이미 설정이 되어 있습니다.

CloudWatch - Logs Insight 로 이동하고, "ANFW-N2SVPC/N2S-fw/flow 로그 그룹을 선택합니다.

Query 문에는 아래 결과로 출력된 Cloud9의 Public IP를 입력합니다.

```
echo "export CLOUD9_IP_ADDR=$(aws ec2 describe-instances --filters Name=tag:Name,Values=aws-cloud9-* | jq -r '.Reservations[].Instances[].PublicIpAddress')"
source ~/.bash_profile
echo $CLOUD9_IP_ADDR

```

아래는 Cloud9 IP (외부 공인 주소)에 대해 ANFW(Amazon Network FireWall)에서 Flow를 조회하는 Query 문의 예제입니다.

```
fields @timestamp, @message, @logStream, @log
| filter (event.src_ip like /{Cloud9 Public IP addr}/) 
| sort @timestamp desc
| limit 20
```

<figure><img src="../.gitbook/assets/image (248).png" alt=""><figcaption></figcaption></figure>

실제 ANFW의 로그를 분석해 봅니다.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* dest\_ip : ELB(ALB)의 Internal CIDR 주소입니다.
* src\_ip  :  Cloud9 의 Public IP 주소입니다.

