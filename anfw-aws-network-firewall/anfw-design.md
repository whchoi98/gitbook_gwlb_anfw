---
description: 'Update : 2022-06-12 / 2h'
---

# ANFW Design

## 목표 구성 개요

2개의 각 워크로드 VPC (VPC01,02)은 Account내에 구성된 ANFW 기반의 보안 VPC를 통해서 내, 외부 트래픽을 처리하는 구성입니다. ANFW(AWS Network Firewall)와 연계되는 N2S VPC는 VPC01,02의 내외부 트래픽을 ANFW로 우회시킵니다

이러한 구성은 VPC Endpoint를 특정 VPC에 구성하고, TransitGateway를 통해 ANFW에 VPC Endpoint Service를 연결하는 중앙집중 구조입니다.

ALB(Application Load Balancer)를 ANFW를 연계하는 VP..C에 배치해서, 내부의 VPC01,02의 서비스들이 외부에 제공하도록 할 수 있습니다

아래 그림은 목표 구성도 입니다.

![](<../.gitbook/assets/image (209) (1) (1) (1).png>)

## Cloudformation기반 VPC 배포

### 1.VPC yaml 파일 다운로드

Cloud9 콘솔에서 아래 github로 부터 VPC yaml 파일을 다운로드 합니다. (앞서 다운로드 하였으면 생략합니다.)

```
git clone https://github.com/whchoi98/gwlb_anfw.git

```

아래와 같은 순서로 Cloudformation에서 Yaml파일을 배포합니다.

1. ANFW-N2SVPC.yml
2. ANFW-VPC01.yml, ANFW-VPC02.yml
3. ANFW-TGW.yml

### 2.N2SVPC 배포

외부 인터넷으로 통신하는 North-South 트래픽 처리를 하는 VPC를 생성합니다. 해당 VPC는 ANFW과 연계합니다

N2SVPC를 Cloudformation에서 앞서 과정과 동일하게 생성합니다. 다운로드 받은 Yaml 파일들 중에 N2SVPC 선택해서 생성합니다.스택 이름을 생성하고, Tokyo Region에 배포합니다.&#x20;

* 스택이름 : ANFW-N2SVPC
* AvailabilityZone A : ap-northeast-2a
* AvailabilityZone B : ap-northeast-2b
* VPCCIDRBlock: 10.11.0.0
* GWLBeSubnetABlock:10.11.1.0/24
* GWLBeSubnetBBlock:10.11.2.0/24
* PublicSubnetABlock: 10.11.11.0/24
* PublicSubnetBBlock: 10.11.12.0/24
* PrivateSubnetABlock:10.11.21.0/24
* PrivateSubnetBBlock:10.11.22.0/24
* TGWSubnetABlock:10.11.251.0/24
* TGWSubnetBBlock:10.11.252.0/24
* DefaultRouteBlock: 0.0.0.0/0 (Default Route Table 주소를 선언합니다.)
* VPC1CIDRBlock : 10.1.0.0/16 (VPC1의 CIDR Block 주소를 선언합니다.)
* VPC2CIDRBlock: 10.2.0.0/16 (VPC2의 CIDR Block 주소를 선언합니다.)
* InstanceTyep: t3.small
* KeyPair : 사전에 만들어 둔 keyPair를 사용합니다.(예. mykey, 사전 준비에서 변수로 입력해 두었습니다)

```
export KeyName=mykey
echo "export KeyName=${KeyName}" | tee -a ~/.bash_profile
source ~/.bash_profile
export AWS_REGION=ap-northeast-2
aws cloudformation deploy \
  --region ${AWS_REGION} \
  --stack-name "ANFW-N2SVPC" \
  --template-file "/home/ec2-user/environment/gwlb_anfw/anfw/1.ANFW-N2SVPC.yml" \
  --parameter-overrides \
    "KeyPair=$KeyName" \
    "AvailabilityZoneA=ap-northeast-2a" \
    "AvailabilityZoneB=ap-northeast-2b" \
    "InstanceType=t3.small" \
  --capabilities CAPABILITY_NAMED_IAM
  
```

### 4.VPC01,02 배포

**나머지 ANFW-VPC01,ANFW-VPC02 의 Cloudformation Yaml 파일을 업로드 합니다.**

{% hint style="info" %}
VPC는 계정당 기본 5개가 할당되어 있습니다. 1개는 Default VPC로 사용 중이고, 4개를 사용 가능하므로 일반 계정에서는 N2SVPC, VPC01,VPC02 까지만 생성 가능합니다.
{% endhint %}

* 스택이름 : ANFW-VPC01,ANFW-VPC02
* AvailabilityZone A : ap-northeast-1a
* AvailabilityZone B : ap-northeast-1b
* VPCCIDRBlock: 10.1.0.0 (VPC01), 10.2.0.0 (VPC02)
* PrivateSubnetABlock:10.1.21.0/24 (VPC01), 10.2.22.0/24(VPC02)
* PrivateSubnetBBlock:10.1.22.0/24 (VPC01), 10.2.22.0/24(VPC02)
* TGWSubnetABlock:10.1.251.0/24 (VPC01), 10.2.251.0/24 (VPC02)
* TGWSubnetBBlock:10.1.252.0/24 (VPC01), 10.2.252.0/24 (VPC02)
* InstanceTyep: t3.small
* KeyPair : 사전에 만들어 둔 keyPair를 사용합니다.(예. mykey, 사전 준비에서 변수로 입력해 두었습니다)

```
export AWS_REGION=ap-northeast-2
aws cloudformation deploy \
  --region ${AWS_REGION}  \
  --stack-name "ANFW-VPC01" \
  --template-file "/home/ec2-user/environment/gwlb_anfw/anfw/2.ANFW-VPC01.yml" \
  --parameter-overrides \
    "KeyPair=$KeyName" \
    "AvailabilityZoneA=ap-northeast-2a" \
    "AvailabilityZoneB=ap-northeast-2b" \
    "InstanceType=t3.small" \
  --capabilities CAPABILITY_NAMED_IAM
  
```

```
export AWS_REGION=ap-northeast-2
aws cloudformation deploy \
  --region ${AWS_REGION} \
  --stack-name "ANFW-VPC02" \
  --template-file "/home/ec2-user/environment/gwlb_anfw/anfw/3.ANFW-VPC02.yml" \
  --parameter-overrides \
    "KeyPair=$KeyName" \
    "AvailabilityZoneA=ap-northeast-2a" \
    "AvailabilityZoneB=ap-northeast-2b" \
    "InstanceType=t3.small" \
  --capabilities CAPABILITY_NAMED_IAM
  
```

ANFW-N2SVPC, ANFW-VPC01,02 을 연결할 TGW를 생성합니다. ANFW-N2STGW는 TGW Routing Table과 각 VPC들이 Route Table을 자동으로 구성해 줍니다.

* Stack Name : GWLBTGW
* DefaultRouteBlock: 0.0.0.0/0
* VPC01CIDRBlock: 10.1.0.0/16
* VPC02CIDRBlock: 10.2.0.0/16

아래와 같이 VPC가 모두 정상적으로 설정되었는지 확인해 봅니다.

**`AWS 관리 콘솔 - VPC 대시 보드 - VPC`**

![](<../.gitbook/assets/image (218) (1) (1) (1).png>)

**`AWS 관리 콘솔 - VPC 대시 보드 - 서브넷`**

![](<../.gitbook/assets/image (210) (1) (1).png>)

### 5. TransitGateway 배포

ANFW-N2SVPC, ANFW-VPC01,ANFW-VPC02을 연결하기 위한 TransitGateway를 배포합니다. 앞서 git을 통해 다운 받은 파일 중 ANFW-TGW.yml 파일을 Cloudformation을 통해서 배포합니다.

```
aws cloudformation deploy \
  --region ${AWS_REGION} \
  --stack-name "ANFWTGW" \
  --template-file "/home/ec2-user/environment/gwlb_anfw/anfw/4.ANFW-TGW.yml"
  
```

### 6. 라우팅 테이블 확인

TransitGateway 구성과 RouteTable을 아래에서 확인합니다. Egress(VPC에서 외부로 향하는) 에 대한 각 테이블을 확인하고 , 이후 Ingress (IGW에서 내부로 향하는)에 대한 테이블을 확인해 봅니다.

![](<../.gitbook/assets/image (205) (1) (1) (1).png>)

**`AWS 관리콘솔 - VPC - 라우팅 테이블`** 을 선택하고, **`"ANFW-VPC01-Private-Subnet-A,B-RT"`**의 **`라우팅`**을 확인합니다.

![](<../.gitbook/assets/image (215) (1) (1).png>)

**`AWS 관리콘솔 - TransitGateway`** 를 선택하고, **`"ANFWTGW"`** 라는 이름으로 **`TransitGateway`**가 정상적으로 생성되었는지 확인합니다.

![](<../.gitbook/assets/image (214) (1) (1) (1).png>)

**`AWS 관리콘솔 - TransitGateway - TransitGateway Attachment(연결)`** 을 선택하고, 각 VPC에 연결된 Attachment를 확인해 봅니다.

![](<../.gitbook/assets/image (216) (1).png>)

**`AWS 관리콘솔 - TransitGateway - TransitGateway 라우팅테이블`**을 선택하고, **`"GWLBTGW-RT-VPC-OUT"`** 을 선택해서, TGW에서 트래픽이 외부로 가는 라우팅을 확인해 봅니다.

![](<../.gitbook/assets/image (208) (1) (1) (1).png>)

**`AWS 관리콘솔 - VPC - 라우팅 테이블`** 을 선택하고, **`"N2SVPC-Private-Subnet-A,B-RT"`**의 **`라우팅`**을 확인합니다.

![](<../.gitbook/assets/image (207) (1) (1).png>)

**`AWS 관리콘솔 - VPC - 라우팅 테이블`** 을 선택하고, **`"N2SVPC-Public-Subnet-A,B-RT"`**의 **`라우팅`**을 확인합니다.

![](<../.gitbook/assets/image (217) (1) (1) (1).png>)

**`AWS 관리콘솔 - VPC - 라우팅 테이블`** 을 선택하고, **`"N2SVPC-GWLBe-Subnet-A,B-RT"`**의 **`라우팅`**을 확인합니다.

![](<../.gitbook/assets/image (211) (1) (1).png>)

**`AWS 관리콘솔 - VPC - 라우팅 테이블`** 을 선택하고, **`"N2SVPC-IGW-Ingress-RT"`**의 **`라우팅`**을 확인합니다.

![](<../.gitbook/assets/image (212) (1) (1) (1) (1) (1).png>)

## Traffic 확인

### 7. 트래픽 확인.&#x20;

아래와 같은 트래픽 흐름으로 VPC 에서 외부로 트래픽을 처리하게 됩니다.

![](<../.gitbook/assets/image (213) (1) (1) (1) (1).png>)

1. ANFW-VPC01,02 Private Subnet Instance에서 TGW 로 트래픽 전송 (Private Subnet Routing Table 참조)
2. TGW에서 ANFW-VPC01의 Attachment 로 연결된 라우팅 테이블을 참조
3. TGW에서 라우팅 테이블을 참조해서 ANFW-N2VPC로 트래픽 전송
4. ANFW-N2SVPC NAT Gateway로 전송 (Private Subnet Routing Table 참조)
5. ANFW-N2SVPC Public Subnet 에서 VPC Endpoint로 전송 (Public Subnet Routing Table 참조)
6. ANFW-N2SVPC VPC Endpoint에서 ANFW 로 전송
7. ANFW 의 방화벽 정책 수행 후 Return
8. ANFW-N2SVPC ANFW Subnet에서 라우팅을 통해 IGW전달
9. IGW에서 인터넷으로 트래픽 처리

### 8. Egress 트래픽 확인

ANFW-VPC01,02의 EC2에서 외부로 정상적으로 트래픽이 처리되는 지 확인 해 봅니다.

Cloud9 터미널을 접속해서 , ANFW-VPC 01,02의 Private Subnet 에 배치된 EC2 인스턴스에 접속해 봅니다. Private Subnet은 직접 연결이 불가능하기 때문에 Session Manager를 통해 접속합니다.

ANFW-VPC01,02 을 Cloudformation을 통해 배포할 때 해당 인스턴스들에 Session Manager 접속을 위한 Role과 Session Manager 연결을 위한 Endpoint가 이미 구성되어 있습니다.

session manager 기반으로 접속하기 위해, 아래 명령을 실행하여 ec2 인스턴스의 id값을 확인합니다.

```
aws ec2 describe-instances --query 'Reservations[].Instances[].[Tags[?Key==`Name`] | [0].Value, Placement.AvailabilityZone,InstanceId, InstanceType, ImageId,State.Name, PrivateIpAddress, PublicIpAddress ]' --output table --region ${AWS_REGION}

```

```
### 예
$ aws ec2 describe-instances --query 'Reservations[].Instances[].[Tags[?Key==`Name`] | [0].Value, Placement.AvailabilityZone,InstanceId, InstanceType, ImageId,State.Name, PrivateIpAddress, PublicIpAddress ]' --output table --region ap-northeast-1
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
|                                                                         DescribeInstances                                                                        |
+------------------------------------+------------------+----------------------+-----------+------------------------+----------+----------------+------------------+
|  ANFW-VPC02-Private-B-10.2.22.102  |  ap-northeast-1c |  i-0ca2538a6bd937fa5 |  t3.small |  ami-0f9a314ce79311c88 |  running |  10.2.22.102   |  18.183.127.88   |
|  ANFW-VPC02-Private-B-10.2.22.101  |  ap-northeast-1c |  i-0ef947bbc040ba658 |  t3.small |  ami-0f9a314ce79311c88 |  running |  10.2.22.101   |  13.231.181.250  |
|  ANFW-N2SVPC-Private-B-10.11.22.101|  ap-northeast-1c |  i-02a3d7af737b0e344 |  t3.small |  ami-0f9a314ce79311c88 |  running |  10.11.22.101  |  None            |
|  ANFW-VPC01-Private-B-10.1.22.101  |  ap-northeast-1c |  i-0c75f95edd2692161 |  t3.small |  ami-0f9a314ce79311c88 |  running |  10.1.22.101   |  35.77.101.217   |
|  ANFW-VPC01-Private-B-10.1.22.102  |  ap-northeast-1c |  i-0102342c91c7cf613 |  t3.small |  ami-0f9a314ce79311c88 |  running |  10.1.22.102   |  35.78.89.45     |
|  ANFW-N2SVPC-Private-B-10.11.22.102|  ap-northeast-1c |  i-05459544c12ec5fdc |  t3.small |  ami-0f9a314ce79311c88 |  running |  10.11.22.102  |  None            |
|  ANFW-VPC01-Private-A-10.1.21.102  |  ap-northeast-1a |  i-08118aef0be9573f0 |  t3.small |  ami-0f9a314ce79311c88 |  running |  10.1.21.102   |  18.177.144.50   |
|  ANFW-VPC02-Private-A-10.2.21.102  |  ap-northeast-1a |  i-0c4dafdf88b772341 |  t3.small |  ami-0f9a314ce79311c88 |  running |  10.2.21.102   |  13.115.162.243  |
|  ANFW-VPC01-Private-A-10.1.21.101  |  ap-northeast-1a |  i-0640ca12e0368d96a |  t3.small |  ami-0f9a314ce79311c88 |  running |  10.1.21.101   |  18.179.42.219   |
|  ANFW-N2SVPC-Private-A-10.11.21.101|  ap-northeast-1a |  i-095092b0e70f3c8ec |  t3.small |  ami-0f9a314ce79311c88 |  running |  10.11.21.101  |  None            |
|  ANFW-N2SVPC-Private-A-10.11.21.102|  ap-northeast-1a |  i-0b427c46ca27f34db |  t3.small |  ami-0f9a314ce79311c88 |  running |  10.11.21.102  |  None            |
|  ANFW-VPC02-Private-A-10.2.21.101  |  ap-northeast-1a |  i-0f18c2b9cc6c1823f |  t3.small |  ami-0f9a314ce79311c88 |  running |  10.2.21.101   |  54.248.7.64     |
+------------------------------------+------------------+----------------------+-----------+------------------------+----------+----------------+------------------+
```

배포된 인스턴스 정보들에 대해 변수로 저장해 두기 위해, 아래 Shell을 실행시킵니다.

```
~/environment/gwlb_anfw/anfw/anfw_ec2_shell.sh

```

session manager 명령을 통해 해당 인스턴스에 연결해 봅니다. (예. ANFW-VPC01-Private-A-10.1.21.101)

```
aws ssm start-session --target $ANFW_VPC01_21_101 --region ${AWS_REGION}

```

터미널에 접속한 후에 , 아래 명령을 통해 bash로 접근해서 외부로 트래픽을 전송해 봅니다.

```
sudo -s
ping www.aws.com

```

### 9. Ingress 트래픽 확인

외부에서 ANFW-N2SVPC의 ALB의 공인 DNS A레코드로 접근하기 위해, ANFW를 통과한 이후에 다시 ANFW-N2SVPC ALB 접근 이후 , Target Group을 ANFW-VPC01,02 인스턴스 IP주소로 구성해서 웹 서비스를 제공하는 방식을 구성했습니다.

아래와 같은 도식으로 외부에서 내부로 웹서비스나 기타 퍼블릭 서비스를 제공할 수 있습니다.

![](<../.gitbook/assets/image (205) (1) (1).png>)

1. 외부에 노출된 ALB DNS A 레코드로 접근 합니다.
2. IGW에서 Ingress Routing을 통해 ANFW-N2SVPC VPC Endpoint로 접근합니다.\
   (ALB의 내부 주소는 10.11.11.0/24,10.11.12.0/24 이고 , Ingress Routing Table에서는 VPC Endpoint로 목적지를 설정해 두었습니다.)
3. ANFW-N2SVPC VPC Endpoint에서 ANFW로 트래픽을 전송합니다.
4. ANFW에서 보안정책 처리 후 Return
5. ANFW-N2SVPC VPC Endpoint에서 ALB 전달
6. ALB에서 VPC01,02 Target Group으로 전달.

## ALB 확인

### 10. 인스턴스 패키지 설치&#x20;

ANFW-VPC01,02의 EC2 인스턴스는 ANFW-TGW(TransitGateway)가 생성된 이후 부터 인터넷이 가능했습니다. 아직까지 어떠한 패치나 패키지 설치가 이루어 지지 않았습니다.

AWS의 Resource Group 구성과 System Manager RunBook을 통해서 , Shell을 동시에 8개를 수행합니다.

**`AWS 관리콘솔 - Resource Group & Tag Editor`** 를 실행하고, **`리소스 그룹 생성`**을 선택합니다.

![](<../.gitbook/assets/image (184).png>)

아래와 같이 퀴리 기반 그룹을 생성합니다.

![](<../.gitbook/assets/image (213) (1) (1) (1).png>)

![](<../.gitbook/assets/image (209) (1) (1).png>)

* **`그룹 유형 : Cloudformation 스택기반`**
* **`그룹화 기준 - Cloudformation 스택 : ANFW-VPC01`**
* **`그룹화 기준 - Cloudformation 스택의 리소스 유형 : AWS::EC2::Instance`**
* **`그룹리소스 미리보기 선택`**
* **`그룹 세부 정보 : ANFW-VPC01-Private-Instance`**

반복해서 VPC02 도 구성합니다.

* **`그룹 유형 : Cloudformation 스택기반`**
* **`그룹화 기준 - Cloudformation 스택 : ANFW-VPC02`**
* **`그룹화 기준 - Cloudformation 스택의 리소스 유형 : AWS::EC2::Instance`**
* **`그룹리소스 미리보기 선택`**
* **`그룹 세부 정보 : ANFW-VPC02-Private-Instance`**

생성된 Resource Group을 **`"저장된 리소스 그룹"`** 에서 확인해 봅니다.

![](<../.gitbook/assets/image (219) (1) (1).png>)

**`AWS 관리콘솔 - System Manager`** 를 실행하고, **`"Run Command"`** 를 빠른 설정 메뉴에서 선택합니다.

**`명령 실행`**을 선택합니다.

![](<../.gitbook/assets/image (188).png>)

**`명령 실행`**에서 **`AWS-RunShellScript`** 를 선택합니다.

![](<../.gitbook/assets/image (189).png>)

명령 파라미터에서 아래 Shell 값을 입력합니다.

```
#!/bin/bash
sudo yum -y update;
sudo yum -y install yum-utils; 
sudo yum -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm;
sudo yum -y install iotop iperf3 iptraf tcpdump git bash-completion; 
sudo yum -y install httpd php mysql php-mysql; 
sudo yum -y install python-pip;
sudo yum -y install nethogs iftop lnav nmon tmux wireshark vsftpd ftp golang;
sudo systemctl start httpd;
sudo systemctl enable httpd;
cd /var/www/html/;
sudo git clone https://github.com/whchoi98/ec2meta-webpage.git;
sudo systemctl restart httpd;
exit;

```

![](<../.gitbook/assets/image (191).png>)

대상에서 리소스그룹을 선택하고, 리소스 그룹은 앞서 생성한 "ANFW-VPC01-Private-Instance", "ANFW-VPC02-Private-Instance"를 선택합니다.

![](<../.gitbook/assets/image (212) (1) (1) (1) (1).png>)

ANFW-VPC01-Private-Instance, ANFW-VPC02-Private-Instance를 각각 실행합니다.

모두 실행하고 나면, 아래와 같이 명령기록에 Shell이 8개 인스턴스에 모두 실행된 것을 확인할 수 있습니다.

![](<../.gitbook/assets/image (222) (1) (1).png>)

### 11. ALB 구성

이제 N2SVPC에서 VPC01,VPC02의 인스턴스 로드밸런서를 위한 ALB 구성을 하고, Target Group을 각각 VPC01,02로 지정합니다.

`AWS 관리콘솔 - EC2 - 로드밸런서 - Application Load Balancer`를 선택하고 `로드 밸런서 생성`을 선택합니다. Application Loadbalancer를 선택하고 생성합니다

![](<../.gitbook/assets/image (208) (1) (1).png>)

기본 구성&#x20;

* **`이름 : "ALB-VPC01"`** 와 같은 이름을 입력합니다.
* **`체계 : "인터넷 경계"`** 를 선택합니다.
* **`IP 주소유형 - "IPv4"`** 를 선택합니다.

![](<../.gitbook/assets/image (210) (1).png>)

**네트워크 매핑**

* **`VPC : "ANFW-N2SVPC"`** 와 같은 이름을 입력합니다.
* **`매핑 : "ap-northeast-1a (ANFW-N2SVPC-Public-Subnet-A), ap-northeast-1c(ANFW-N2SVPC-Public-Subnet-B)"`** 를 선택합니다.

![](<../.gitbook/assets/image (205) (1).png>)

**보안그룹**

* **`보안 그룹 : "ALBSecurityGroup"`** 를 선택합니다

![](<../.gitbook/assets/image (218) (1) (1).png>)

리스너 및 라우팅

Target Group (대상그룹) 생성을 선택해서, 새로운 창을 오픈합니다.

![](<../.gitbook/assets/image (206) (1).png>)

**그룹 세부 정보 지정**

* 대상유형 선택 - IP 주소 (다른 VPC의 인스턴스로 타겟그룹을 지정하기 위해서는 IP주소만 가능합니다)
* 대상그룹 이름 - **`"ANFW-VPC01-TG", "ANFW-VPC02-TG"`**
* 프로토콜 - HTTP / 80
* VPC - N2SVPC 선택
* 프로토콜 버전 - HTTP1
* 상태검사 프로토콜 - HTTP
* 상태검사 경로 - 아래를 복사해서 입력합니다.

```
/ec2meta-webpage/index.php
```

![](<../.gitbook/assets/image (207) (1).png>)

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

* 대상유형 선택 - IP 주소 (다른 VPC의 인스턴스로 타겟그룹을 지정하기 위해서는 IP주소만 가능합니다)
* 대상그룹 이름 - "**`VPC01-TG`**"**`, "VPC02-TG"`**
* 프로토콜 - HTTP / 80
* VPC - ANFW-N2SVPC 선택
* 프로토콜 버전 - HTTP1
* 상태검사 프로토콜 - HTTP
* 상태검사 경로 - 아래를 복사해서 입력합니다.&#x20;

```
/ec2meta-webpage/index.php
```



**대상등록**&#x20;

대상 등록에서는 ANFW-N2SVPC 가 아닌, ANFW-VPC01,02의 인스턴스IP가 Target이 되어야 합니다.

* 네트워크 : **`다른 프라이빗 IP 주소`** 를 선택합니다.
* IP : ANFW-VPC01,02 의 IP 주소를 입력합니다.
* 목록에 추가를 선택하여 ANFW-VPC01,VPC02의 대상등록을 완료합니다.

<figure><img src="../.gitbook/assets/image (232).png" alt=""><figcaption></figcaption></figure>

```
#ANFW-VPC01 IP address
10.1.21.101
10.1.21.102
10.1.22.101
10.1.22.102

#ANFW-VPC02 IP address
10.2.21.101
10.2.21.102
10.2.22.101
10.2.22.102
```

대상 그룹 생성이 완료되면 Application Load Balancer 생성 메뉴창으로 다시 돌아갑니다

아래 리스너 및 라우팅 메뉴에서, 앞서 생성한 대상그룹을 선택합니다

* ALB-VPC01 - **`ANFW-VPC01-TG`**
* ALB-VPC02 - **`ANFW-VPC02-TG`**

![](<../.gitbook/assets/image (216).png>)

ALB 구성의 최종 구성 정보를 확인하고 , ALB를 생성합니다.&#x20;

**`AWS 관리콘솔 - EC2 - 로드밸런서`** 에서 생성한 ANFW-VPC01,02 의 ALB로드밸런서를 확인합니다. **`ALB DNS A 레코드 값`**을 복사해 둡니다.

![](<../.gitbook/assets/image (214) (1) (1).png>)

**`AWS 관리콘솔 - EC2 - 로드밸런서`** 에서 ANFW-VPC01,VPC02 를 대상그룹으로 생성한 Target 인스턴스들이 "Healthy" 상태인지 확인합니다.

![](<../.gitbook/assets/image (211) (1).png>)

### 12. ALB 트래픽 확인

웹 브라우저에서 앞서 복사해둔 ALB DNS A Record와 나머지 URL을 입력합니다.

```
aws elbv2 describe-load-balancers --names ALB-VPC01 --region ${AWS_REGION}| jq -r '.LoadBalancers[].DNSName'
export ANFW_ALB_VPC01_URL=$(aws elbv2 describe-load-balancers --names ALB-VPC01 --region ${AWS_REGION} | jq -r '.LoadBalancers[].DNSName') 
echo "export ANFW_ALB_VPC01_URL=${ANFW_ALB_VPC01_URL}"| tee -a ~/.bash_profile
curl $ANFW_ALB_VPC01_URL
```

```
echo URL= http://${ANFW_ALB_VPC01_URL}/ec2meta-webpage/index.php
```

![](<../.gitbook/assets/image (201).png>)

VPC02의 인스턴스들과 ALB 로드밸런스도 위와 같은 방법으로 확인해 봅니다.



## &#x20;Network Firewall 확인

### 13. Network Firewall 확인.

Cloudformation으로 배포된 Network Firewall이 정상적으로 배포되었는지 확인해 봅니다. Cloudformation으로 사전에 Rule을 배포해 두었습니다.&#x20;

* VPC - Amazon Network Firewall - 방화벽

![](<../.gitbook/assets/image (221) (1).png>)

![](<../.gitbook/assets/image (220) (1) (1).png>)

### 14. Network Firewall 정책

이제 생성된 Firewall과 Firewall Policy에 Rule(보안 규칙)을 설정하여, 상세한 보안 규칙들을 설정해 봅니다.

먼저 Firewall 구성은 아래와 같은 방식으로 구성할 수 있습니다.

![](<../.gitbook/assets/image (213) (1) (1).png>)

### 15. Stateless Rule 적용

앞서 생성한 ALB-VPC01, ALB-VPC02 웹서비스 중에 1개의 ALB를 Block 해 봅니다

먼저 ALB의 실제 내부 주소를 확인합니다.

* EC2 - 네트워크 및 보안 - 네트워크 인터페이스 - ALB-VPC02 -프라이빗 IPv4 확인
* 검색 키워드

```
ALB-VPC02
```

![](<../.gitbook/assets/image (212) (1) (1) (1).png>)

Stateless 정책을 생성하고 적용합니다

* VPC - 방화벽 - N2S-firewall-ANFW-N2SVPC - 상태 비저장 규칙 그룹 - 작업 - 무상태 규칙 그룹 생성

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

![](<../.gitbook/assets/image (218) (1).png>)

* 이름 :  Stateless-Rule
* 용량 : 100

![](<../.gitbook/assets/image (205).png>)

* 우선순위 : 11
* 프로토콜 : All
* 소스 : 모든 IP 주소
* 대상 : 사용자 지정 / ALB-VPC02 의 네트워크 인터페이스 IP를 입력
* 작업 - 삭제
* 규칙추가 선택

![](<../.gitbook/assets/image (209) (1).png>)

* 생성 및 추가 선택

ALB-VPC02 DNS 주소를 입력하고 Cloud9 터미널에서 Curl을 실행하거나,브라우저에서 아래 주소를 입력해 봅니다.

* ALB-VPC01, 02 주소 확인 및 보안 적용 확인&#x20;

```
#ALB-VPC01
export ALBVPC01URL=$(aws elbv2 describe-load-balancers --query "LoadBalancers[].[LoadBalancerName,Scheme,DNSName,VpcId]" --output text --region ap-northeast-1 | awk '/ALB-VPC01-/ {print $3}')
echo $ALBVPC01URL

#ALB-VPC02
export ALBVPC02URL=$(aws elbv2 describe-load-balancers --query "LoadBalancers[].[LoadBalancerName,Scheme,DNSName,VpcId]" --output text --region ap-northeast-1 | awk '/ALB-VPC02-/ {print $3}')
echo $ALBVPC02URL

curl -v  $ALBVPC01URL/ec2meta-webpage/index.php -I
curl -v  $ALBVPC02URL/ec2meta-webpage/index.php -I

```

{% hint style="info" %}
ALBVPC01 은 접속이 허용되고, ALBVPC02는 접속되지 않습니다
{% endhint %}

확인 후 보안 룰을 삭제합니다.&#x20;

* Amazon Network Firewall - 방화벽 - 방화벽 정책 - 상태비저장 규칙 그룹

![](<../.gitbook/assets/image (206).png>)

* Amazon Network Firewall - Network Firewall 규칙 그룹 - Stateless-Rule 선택 - 삭제

![](<../.gitbook/assets/image (204) (1).png>)

### 16. Stateful Rule 적용

&#x20;Cloudformation으로 배포된 Stateful Rule에 ALB-VPC02 만 허용하도록 변경합니다. &#x20;

* VPC - Amazon Network Firewall - allpermit 규칙을 선택합니다

![](<../.gitbook/assets/image (224) (1).png>)

* 규칙편집을 선택합니다.

![](<../.gitbook/assets/image (207).png>)

* 규칙 추가를 선택합니다

<figure><img src="../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

* 프로토콜 - IP
* 소스 - ANY
* 대상 - 앞서 구성했던 ALB-VPC02의 ENI 주소 2개를 입력합니다
* 트래픽 방향 - 임의
* 작업 - 통과

![](<../.gitbook/assets/image (219) (1).png>)

기존 IP Allpermit Rule의 허용 규칙을 Drop(삭제)로 변경합니다.

* 기존 IP Allpermit Rule은 작업 - 삭제로 변경합니다.

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

![](<../.gitbook/assets/image (214) (1).png>)

변경이 완료되면 아래와 같이 변경됩니다.&#x20;

![](<../.gitbook/assets/image (220) (1).png>)

ALB-VPC02 DNS 주소를 입력하고 Cloud9 터미널에서 Curl을 실행하거나,브라우저에서 아래 주소를 입력해 봅니다.

* ALB-VPC01, 02 주소 확인 및 보안 적용 확인&#x20;

```
#ALB-VPC01
export ALBVPC01URL=$(aws elbv2 describe-load-balancers --query "LoadBalancers[].[LoadBalancerName,Scheme,DNSName,VpcId]" --output text --region ap-northeast-1 | awk '/ALB-VPC01-/ {print $3}')
echo $ALBVPC01URL

#ALB-VPC02
export ALBVPC02URL=$(aws elbv2 describe-load-balancers --query "LoadBalancers[].[LoadBalancerName,Scheme,DNSName,VpcId]" --output text --region ap-northeast-1 | awk '/ALB-VPC02-/ {print $3}')
echo $ALBVPC02URL

curl -v  $ALBVPC02URL/ec2meta-webpage/index.php -I
curl -v  $ALBVPC01URL/ec2meta-webpage/index.php -I

```

{% hint style="info" %}
ALBVPC0**2** 은 접속이 허용되고, ALBVPC01는 접속되지 않습니다
{% endhint %}

확인 후 보안 룰을 원복합니다.

![](<../.gitbook/assets/image (204).png>)

### 17. Stateful Rule - Suricata 적용

Suricata는 IDS(탐지)/IPS(탐지,차단)가 가능한 Open source 도구 입니다. Snort와 완벽하게 호환이 가능하며, 멀티 쓰레드 지원과 GPU 지원등으로 성능 부분에서 높은 평가를 받고 있습니다. (2020년 부터 Snort 3.0 출시와 함께 멀티 쓰레지 지원)

Amazon Network Firewall의 Stateful IPS는 Suricata IPS를 통해서, Deep Inspection구현이 가능합니다. 또한 상용도구와 연계도 가능합니다. ( Fortinet 지원- 상용)

먼저 새로운 Stateful Rule Group 생성을 합니다.

**`VPC-Firewall policies - 생성한 Policy - Stateful rule groups - Add rule groups - Create and add new stateful rule group`**

Stateful rule group을 생성합니다.

1. **Name : Stateful Rule 이름을 정의합니다.**
2. **Capacity : Rule Group의 Rule의 숫자를 정의합니다.(최대 30,000개)**
3. **Stateful rule group options : Suricata IPS Rule을 선택합니다.**
4. **Suricata IPS Rule을 설정합니다.**

![](<../.gitbook/assets/image (229).png>)

![](<../.gitbook/assets/image (224).png>)

IP Set 정의 (옵션) 를 합니다. IP 대역에 대한 변수 설정을 미리 해 둘 수 있습니다.&#x20;

![](<../.gitbook/assets/image (220).png>)

Suricata IPS Rule을 정의합니다

![](<../.gitbook/assets/image (231).png>)

IPS Rule은 아래와 같이 구성해 봅니다.

```
#AWS 키워드 메세지가 들어가는 접속은 Alert을 띄웁니다
alert tcp any any -> any any (msg: "No access to the EC2-1 Webpage"; content: "AWS"; sid: 101; rev:1;)
#ALB_VPC01 을 접속할때 Alert을 띄웁니다
alert http any any -> $ALB_VPC01 any (msg: "User agent"; http.user_agent; content:"Firefox"; sid:102; rev:1;)
#ALB_VPC02 를 Firefox로 접속하면 Drop 됩니다.
drop http any any -> $ALB_VPC02 any (msg: "User agent"; http.user_agent; content:"Firefox"; sid:103; rev:1;)
```

"생성하여 정책에 추가" 버튼을 선택합니다.

아래 내용을 참고해서 시험하기 전에 수행합니다.

{% hint style="info" %}
기존에 IP 전체에 대한 Permit 이 적용되어 있기 때문에, suricata rule이 적용되지 않고, Pass 됩니다. 해당 정책의 연결을 해제해야 Suricata 정책이 적용됩니다. 이것을 Strict Mode라고 하며, 기본 동작방식입니다.&#x20;
{% endhint %}

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

**`VPC - 방화벽 정책 - 생성되어 있는 정책 선택`**

**`"allpermit-ANFW-N2SVPC"` 상태 저장 규칙 그룹**을 **`정책에서 연결 해제`**합니다.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

이제 Web 브라우저에서 접속해 봅니다.

```
#ALB-VPC01
export ALBVPC01URL=$(aws elbv2 describe-load-balancers --query "LoadBalancers[].[LoadBalancerName,Scheme,DNSName,VpcId]" --output text --region ap-northeast-1 | awk '/ALB-VPC01-/ {print $3}')
echo $ALBVPC01URL/ec2meta-webpage/index.php

#ALB-VPC02
export ALBVPC02URL=$(aws elbv2 describe-load-balancers --query "LoadBalancers[].[LoadBalancerName,Scheme,DNSName,VpcId]" --output text --region ap-northeast-1 | awk '/ALB-VPC02-/ {print $3}')
echo ${ALBVPC02URL}/ec2meta-webpage/index.php

```

![](<../.gitbook/assets/image (215).png>)

### 18. Log확인

Cloudformation으로 배포 할때 CloudWatch Log Group을 생성해서 Flow, Alert/Drop에 대해 로그를 확인 할 수 있도록 하였습니다

CloudWatch에서 아래에서 처럼 Log를 확인해 봅니다.

![](<../.gitbook/assets/image (230).png>)

* CloudWatch - Logs - Log Groups - /ANFW-N2SVPC/N2S-fw/alert - LogStream 선택

![](<../.gitbook/assets/image (219).png>)

* CloudWatch - Logs - Log Groups - /ANFW-N2SVPC/N2S-fw/flow - LogStream 선택&#x20;

![](<../.gitbook/assets/image (218).png>)



## 자원 삭제



로드 밸런서를 삭제 합니다. (**ALB-VPC01-TG, ALB-VPC02-TG 만 삭제합니다.**)

* EC2- LoadBalancing -Load Balancer

![](<../.gitbook/assets/image (211).png>)

**`AWS 관리 콘솔 - 로드밸런싱 - 로드밸런서 - ANFW-VPC01-TG, ANFW-VPC02-TG 선택 - 작업 - 삭제`**

ALB와 대상 그룹을 삭제합니다. (**VPC01-TG,VPC02-TG 만 삭제 합니다.**)

* **`EC2 - 로드밸런싱 - 대상 그룹 - ANFW-VPC01-TG 선택 - 작업 선택 - 삭제`**
* **`EC2 - 로드밸런싱 - 대상 그룹 - ANFW-VPC02-TG 선택 - 작업 선택 - 삭제`**

Cloud9 콘솔에서 아래 명령을 통해서 삭제 합니다.&#x20;

1. ANFWTGW를 삭제합니다. (3\~4분 소요됩니다.)
2. ANFW-VPC01,VPC02를 삭제합니다. (3\~4분 소요됩니다. 동시 진행합니다.)
3. ANFW-N2SVPC를 삭제 합니다. (3\~4분 소요됩니다.)

```
aws cloudformation delete-stack --stack-name ANFWTGW --region ap-northeast-1
aws cloudformation delete-stack --stack-name ANFW-VPC02 --region ap-northeast-1
aws cloudformation delete-stack --stack-name ANFW-VPC01 --region ap-northeast-1
aws cloudformation delete-stack --stack-name N2SVPC --region ap-northeast-1

```

랩을 완전히 종료하려면 **`AWS 관리콘솔 - Cloudformation - 스택`** aws cloud9 콘솔 스택도 삭제합니다.
