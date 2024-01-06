---
description: 'Update : 2020-12-20'
---

# Network Firewall 기본 구성

## 구성  아키텍쳐 소개

Network Firewall의 기본 동작 이해를 위해, 가장 기본이 되는 디자인을 먼저 구성해 봅니다.&#x20;

아래 그림은 이번 Chapter에서 구성해 볼 아키텍쳐 입니다. 인터넷으로 부터 AWS 자원을 보호하기 위해, 네트워크 방화벽 서브넷을 배치하고 인터넷을 통과하는 모든 트래픽은 네트워크 방화벽을 통과하게 하는 일반적인 On Prem구성과 유사합니다.

![\[Network Firewall 기본 구성 아키텍쳐\]](<.gitbook/assets/image (72).png>)

## Task1.Cloudformation 배포&#x20;

Cloudformation을 통해 기본이 되는 VPC구성을 먼저 구성합니다.

### 1.구성 목표.&#x20;

먼저 아래의 Cloudformation을 배포합니다. Cloudformation 구성을 배포하게 되면 아래와 같은 구성이 완료됩니다.

Routing Table 구성과 Network Firewall 구성은 다음단계에 별도로 진행하게 됩니다.

![\[Cloudformation 기반의 배포 아키텍쳐\]](<.gitbook/assets/image (42).png>)

Cloud9 터미널에서 아래 Github에서 실습에 사용할 Cloudformation yml 파일을 다운로드 받습니다.

```
git clone https://github.com/whchoi98/aws-nwfw-source
```

### 2.Cloudformation 생성.&#x20;

#### 먼저 새로운 스택을 생성합니다.

![](<.gitbook/assets/image (20).png>)

#### 앞서 다운로드 받은 git 파일 중에서**`"singleaz-vpc1-az-a.yml"`** 파일을 업로드 합니다.

![](<.gitbook/assets/image (50).png>)

Cloud9에서 직접 file을 업로드하기 위해서는 아래와 같이 S3를 활용할 수도 있습니다.

```
##S3 Bucket 생성합니다. 
##Bucket name은 고유해야 합니다.
export bucket_name="usernameDate"
echo "export bucket_name=${bucket_name}" | tee -a ~/.bash_profile
aws s3 mb s3://${bucket_name}

#생성한 S3 Bucket으로 파일을 모두 복사해 둡니다.
cd ~/environment/aws-nwfw-source

# Cloud9에서 변경되는 파일을 S3와 동기화 합니다. 
aws s3 sync ./ s3://${bucket_name}

## option - copy를 통해 사용해도 가능합니다.
aws s3 cp ./ s3://${bucket_name} --recursive

## LAB에서 사용할 Object접근을 허용합니다.
aws s3api put-object-acl --bucket ${bucket_name} --key EKSVPC3AZ.yml --acl public-read  
```

S3 URL 형식은 생성한 버킷 이름과 리전 주소, Object 로 생성되어 있습니다. 출력값을 복사해 둡니다.&#x20;

```
echo https://${bucket_name}.s3.ap-northeast-1.amazonaws.com/singleaz-vpc1-az-a.yml

```

#### stack의 상세내용을 정의합니다.

* stack name : Stack name을 정의합니다.
* VPC Parameters - AvailablilityZoneA : AZ를 선택합니다.
* KeyPair:사용할 KeyPair를 선택합니다. (사전에 keypair를 생성해 두어야 합니다.)
* LatestAmiId: 최신의 Amazon Linux2 이미지가 자동 선언됩니다.

![](<.gitbook/assets/image (2).png>)

#### Cloudformation이 IAM에 접근하여 사용할 수 있도록 체크합니다.

![](<.gitbook/assets/image (13).png>)

10분 후면 모든 자원이 생성됩니다.

![](<.gitbook/assets/image (96).png>)

{% hint style="info" %}
**본 랩에서는 EC2의 자원들에 손쉽게 접근 할 수 있도록 모두 Session Manager 접근 구성을 Cloudformation으로 배포합니다. AWS 콘솔이나, 다른 배포 도구로 구성하셔도 랩을 진행하는데는 이슈가 없습니다.**
{% endhint %}

## Task2. Network Firewall 기본 구성.&#x20;

먼저 Network Firewall을 생성합니다. 기본 Firewall Policy가 생성되어 있지 않기 때문에, 함께 생성합니다.

### 1.Network Firewall 및 Policy 생성.

**`service - VPC - AWS Network Firewall - Firewall - Create`** 를 선택하고, Firewall과 Firewall Policy를 생성합니다.

<div align="left">

<img src=".gitbook/assets/image (71).png" alt="">

</div>

#### 먼저 Firewall을 생성하고, Firewall Policy 생성하여 연결합니다.

기존에 Firewall Policy가 있다면 생성한 Firewall에 연결할 수 있습니다.

1. **Name** : 방화벽 이름을 정의합니다.&#x20;
2. **Description(Optional)** : 방화벽에 대한 설명을 정의합니다.
3. **VPC** : 생성한 VPC를 선택합니다. (eg. VPC1)
4. **Firewall subnets - Availability Zone** : AZ Zone을 선택합니다. (eg. us-west-2a)\
   **Firewall subnets - Subnet** : 생성한 방화벽용 Subnet을 선택합니다. (eg. VPC1-FWSubnet1)
5. **New Firewall policy name :** 신규 생성한 NWFW의 방화벽 정책이름을 정의합니다.
6. **Description(Optional)** : 방화벽 정에 대한 설명을 정의합니다.  &#x20;
7. **Firewall Tag :** Firewall 자원에 대한 Tag를 정의합니다.

![](<.gitbook/assets/image (1).png>)

![](<.gitbook/assets/image (70).png>)

### 2.Network Firewall 확인.&#x20;

방화벽을 생성하고 나면, **`provisoning`** 상태가 진행되며 완료까지 5분 내외가 소요됩니다.

![](<.gitbook/assets/image (93).png>)

정상적으로 설치되면 아래 그림처럼 **`Status:Ready`** 상태로 변경됩니다.

![](<.gitbook/assets/image (100).png>)

생성한 Firewall을 선택하고 **`Firewall details`** 를 선택하면, 해당 서브넷에 Endpoint가 정상적으로 생성된 것을 확인 할 수 있습니다.

![](<.gitbook/assets/image (92).png>)

**`Service-VPC-Virtual Private Cloud-Endpoint`** 메뉴에서 Firewall Endpoint를 확인 할 수 있습니다.

![](<.gitbook/assets/image (106).png>)

{% hint style="info" %}
**Endpoint 메뉴에서 특이점을 발견할 수 있습니다. Endpoint type이 GatewayLoadBalancer 라는 것입니다. 이것은 Firewall Endpoint가 별도로 생성되지 않고, GatewayLoadBalancer를 그대로 사용한다는 것입니다. 즉 동작방식이 GatewayLoadBalancer를 이용한다는 것을 알 수 있습니다.**
{% endhint %}

## Task3. VPC Route Table 구성

이제 라우팅 테이블을 정의하고, 인터넷과 EC2간의 통신을 확인해 봅니다.

![](<.gitbook/assets/image (115).png>)

### 1. VPC Ingress 라우팅 테이블 구성.&#x20;

외부 인터넷 트래픽인 Firewall Endpoint를 경유하도록 라우팅 테이블을 구성하기 위해서는 InternetGateway 의 라우팅 테이블 구성이 필요합니다. AWS에서는 이러한 Edge에서의 라우팅 테이블 구성이 가능하도록 VPC Ingress Routing을 지원합니다.

#### &#x20;VPC Ingress Route Table을 생성합니다.

**`Service - VPC - Virtual Private Cloud - Route Table - Create route table`**

![](<.gitbook/assets/image (101).png>)

신규 생성한 InternetGateway용 라우팅 테이블을 선택하고, **`Edge Associations`** 를 선택합니다.

![](<.gitbook/assets/image (113).png>)

InternetGateway용 라우팅 테이블을 InternetGateway(이하 IGW)에 연결합니다.

![](<.gitbook/assets/image (107).png>)

연결하면 아래와 같이 정상적으로 IGW에 라우팅 테이블(Ingress Routing)이 생성됩니다.

![](<.gitbook/assets/image (78).png>)

이제 인터넷에서 유입되는 트래픽이 Firewall Endpoint를 향하도록 Ingress Routing을 설정합니다.

**`Route - Edit Routes`** 를 선택하고 수정합니다.

![](<.gitbook/assets/image (6).png>)

목적지는 **`0.0.0.0/0`**을 설정하고, Target은 **`Gateway Load Balancer Endpoint`**를 선택합니다.​

{% hint style="info" %}
**Target이 Gateway Load Balancer Endpoint가 되어야 하는 이유는 앞서 설명하였습니다.**
{% endhint %}

![](<.gitbook/assets/image (46).png>)

Gateway Load Balancer Endpoint를 선택하게 되면, Network Firewall을 생성한 이후에 자동 생성된 VPC Endpoint를 확인할 수 있습니다. 해당 Endpoint를 선택하고 라우팅 테이블을 완료합니다.

이제 10.1.1.0/24 로 외부에서 인입되는 트래픽은 모두 Firewall을 경유하게 됩니다.

![](<.gitbook/assets/image (26).png>)

### 2. FW Subnet 라우팅 테이블 구성.&#x20;

FW Subnet은 인터넷으로 향하는 트래픽에 대한 라우팅 생성을 합니다. Ingress Routing은 별도로 구성할 필요가 없습니다. VPC를 구성할때 CIDR를 생성하면 자동으 Local Routing이 구성되기 때문입니다.&#x20;

FW Subnet Routing Table을 선택하고, **`Route-Edit Routes`** 를 선택합니다.

![](<.gitbook/assets/image (122).png>)

Ingress Routing은 Local이 이미 구성되어 있으므로 별도 구성없이, Egress Routing에 대한 구성만 합니다.&#x20;

외부로 향하는 트래픽은 모두 목적지 IGW로 향하도록 구성하고 저장합니다.

![](<.gitbook/assets/image (16).png>)

![](<.gitbook/assets/image (117).png>)

Firewall 서브넷을 위한 라우팅 구성이 정상적으로 되었는지 확인합니다.

![](<.gitbook/assets/image (36).png>)

### 3. Protect Subnet 테이블 구성.&#x20;

Ingress Routing은 이미 Local Routing 구성이 자동으로 되어 있으므로, Egress Routing에 대한 처리만 합니다.

Protect Subnet을 위한 라우팅을 선택하고, **`Route-Edit Routes`** 를 선택해서 Egress Routing 구성을 합니다.

&#x20;Protect Subnet에 속한 자원들이 외부로 트래픽을 보낼 때 모두 Firewall을 통과하도록, 모든 라우팅 목적지를 Firewall VPC Endpoint로 향하게 구성합니다.

![](<.gitbook/assets/image (24).png>)

![](<.gitbook/assets/image (80).png>)

![](<.gitbook/assets/image (23).png>)

Protect Subnet을 위한 라우팅 테이블이 정상적으로 구성되었는 지 확인합니다.

![](<.gitbook/assets/image (4).png>)

### 4. 트래픽 흐름 확인  &#x20;

이제 모든 라우팅 구성은 완료되었습니다. 앞서 Cloudformation 을 통해서 생성한 EC2 자원들에 대한 Security Group은 이미 설정되어 있습니다.

![](<.gitbook/assets/image (59).png>)

또한 System Manager를 통한 Session Manager구성도 Cloudformation을 통해 구성되어 있습니다. 본 랩에서는 Session Manager를 통해서 접속해서 시험합니다.

{% hint style="success" %}
Protect Subnet의 EC2 자원은 IGW와 1:1 NAT 구성이 되도록 설정되어 있습니다. Session Manager 뿐만 아니라, SSH 접속도 가능합니다. 하지만 보안 정책 테스트를 하기 어렵기 때문에 Session Manager로 접속하는 것을 권고합니다.
{% endhint %}

**`Service - System Manager - Session Manager`** 를 선택하고, **`Start Session`**을 선택합니다.

![](<.gitbook/assets/image (28).png>)

EC2에 이미 System Manager Agent가 설치되어 Web에서 접속이 가능합니다. 접속을 원하는 EC2 인스턴스를 선택하고 **`Start Session`**을 시작합니다.

![](<.gitbook/assets/image (48).png>)

AWS CLI 가 설치된 경우에는  Session Manager Plugin을 설치하여, CLI로 구성과 시험이 가능합니다. ([AWS CLI용 Session Manager  Plugin 설치](https://docs.aws.amazon.com/ko\_kr/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html) )

본 랩에서는 Cloudshell을 사용해서, Session Manager를 사용합니다.

**`Service - Cloudshell`** 을 선택하여, Cloudshell 콘솔을 실행합니다. 아래와 같이 session-manager-plugin 을 설치하고, 랩에 필요한 yml 및 source 들을 설치합니다.

![](<.gitbook/assets/image (124).png>)

```
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/linux_64bit/session-manager-plugin.rpm" -o "session-manager-plugin.rpm"
sudo yum install -y session-manager-plugin.rpm
git clone https://github.com/whchoi98/useful-shell

```

앞서 Git을 통해 다운로드 받은파일 가운데 shell 또는 아래 aws cli를 통해 배포된 인스턴스 id를 확인합니다.

```
./useful-shell/aws_ec2_ext.sh >>vpc1-ec2.txt

```

vpc1-ec2.txt 결과의 예입니다.

```
-----------------------------------------------------------------------------------------------------------------------------------------
|                                                           DescribeInstances                                                           |
+-----------------+-------------+----------------------+-----------+------------------------+----------+--------------+-----------------+
|  Protect-EC2-101|  us-west-2a |  i-040d4d15aebc8fb32 |  t3.small |  ami-0e472933a1395e172 |  running |  10.1.1.101  |  52.34.16.59    |
|  Protect-EC2-102|  us-west-2a |  i-068a26aee30adb069 |  t3.small |  ami-0e472933a1395e172 |  running |  10.1.1.102  |  35.166.81.128  |
+-----------------+-------------+----------------------+-----------+------------------------+----------+--------------+-----------------+
```

인스턴스 id를 Shell에 저장해 둡니다.

```
export VPC1_AZA_101="i-040d4d15aebc8fb32"
export VPC1_AZA_102="i-068a26aee30adb069"
echo "export VPC1_AZA_101=$VPC1_AZA_101" | tee -a ~/.bash_profile
echo "export VPC1_AZA_102=$VPC1_AZA_102" | tee -a ~/.bash_profile
echo $VPC1_AZA_101
echo $VPC1_AZA_102

```

아래와 같은 방법으로 Session Manager를 통해 인스턴스에 접속합니다.

```
#VPC1 AZ-A EC2-101
aws ssm start-session --target $VPC1_AZA_101 --region ap-northeast-1
sudo -s
su ec2-user
cd ~
 
```

```
#VPC1 AZ-A EC2-102
aws ssm start-session --target $VPC1_AZA_102 --region ap-northeast-1
sudo -s
su ec2-user
cd ~

```

각 인스턴스에서 아래 명령을 통해 생성된 EC2 인스턴스들의 local(Private) IP 주소와 공인(Public) IP를 확인합니다.

```
curl http://169.254.169.254/latest/meta-data/local-ipv4
curl http://169.254.169.254/latest/meta-data/public-ipv4
curl -s ifconfig.co

```

먼저 사용자 웹브라우저에서 각 인스턴스의 Public IP 주소로 아래 웹사이트에 접근해 봅니다.

```
http://ec2-101-public-ip/ec2meta-webpage/index.php
```

![](<.gitbook/assets/image (116).png>)

```
http://ec2-102-public-ip/ec2meta-webpage/index.php
```

![](<.gitbook/assets/image (27).png>)

모든 인스턴스에 정상적으로 접속되는 것을 확인할 수 있습니다.

{% hint style="info" %}
**방화벽을 통과하는 트래픽 흐름이지만 , 별도의 정책없이도 정상적으로 웹브라우저가 출력이 됩니다. 이 구성을 통해 알 수 있는 것은, 기본 방화벽 정은 묵시적 허용이라는 것을 알 수 있습니다.**
{% endhint %}

이제 트래픽의 흐름을 아래와 같이 이해 할 수 있습니다.&#x20;

![](<.gitbook/assets/image (108).png>)

EC2-101에서 EC2-102로 Curl을 통해 트래픽 흐름을 확인 해 볼 수 있습니다. 아래 명령을 CloudShell에서 Session Manager를 접속한 상태에서 실행해 봅니다.

```
#EC2-101
curl -I http://ec2-102-public-ip/ec2meta-webpage/index.php
```

```
#EC2-102
sudo tcpdump -i eth0 src ec2-101-public-ip
```

아래 처럼 결과가 출력됩니다.

```
[ec2-user@ip-10-1-1-101 ~]$ curl -I http://35.166.81.128/ec2meta-webpage/index.php
HTTP/1.1 200 OK
Date: Tue, 15 Dec 2020 23:47:15 GMT
Server: Apache/2.4.46 () PHP/5.4.16
Upgrade: h2,h2c
Connection: Upgrade
X-Powered-By: PHP/5.4.16
Content-Type: text/html; charset=UTF-8

```

```
[ec2-user@ip-10-1-1-102 ~]$ sudo tcpdump -i eth0 src 52.34.16.59
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
23:47:11.940652 IP ec2-52-34-16-59.us-west-2.compute.amazonaws.com.45606 > ip-10-1-1-102.us-west-2.compute.internal.http: Flags [S], seq 3254533980, win 26883, options [mss 1460,sackOK,TS val 2878837554 ecr 0,nop,wscale 7], length 0
23:47:11.968465 IP ec2-52-34-16-59.us-west-2.compute.amazonaws.com.45606 > ip-10-1-1-102.us-west-2.compute.internal.http: Flags [.], ack 3141908793, win 211, options [nop,nop,TS val 2878837576 ecr 3940610722], length 0
23:47:11.968479 IP ec2-52-34-16-59.us-west-2.compute.amazonaws.com.45606 > ip-10-1-1-102.us-west-2.compute.internal.http: Flags [P.], seq 0:102, ack 1, win 211, options [nop,nop,TS val 2878837577 ecr 3940610722], length 102: HTTP: HEAD /ec2meta-webpage/index.ph HTTP/1.1
```

## Task4. Network Firewall 상세 구성

이제 생성된 Firewall과 Firewall Policy에 Rule(보안 규칙)을 설정하여, 상세한 보안 규칙들을 설정해 봅니다.

### 1.Firewall 구성 이해

먼저 Firewall 구성은 아래와 같은 방식으로 구성할 수 있습니다.

![](<.gitbook/assets/image (14).png>)

1. **Firewall 을 생성합니다.**
2. **Firewall Policy를 생성합니다.**
3. **Stateless Rule 을 생성합니다.**
4. **Stateful Rule을 생성합니다.**
5. **Stateful Rule의 Domain list 을 생성합니다.**
6. **Stateful Rule의 Suricata IPS Rule을 생성합니다.**

앞서 Firewall과 Firewall 정책은 생성 완료했습니다. [(Task2. Network Firewall 기본 구성)](single-az-nwfw.md#task-2-network-firewall)

### 2.Firewall Rule의 이해와 구성

Network Firewall의 정책을 이해하기 위해 아래 그림을 이해해야 합니다.

![](<.gitbook/assets/image (55).png>)

### 3. Stateless Rule 구성&#x20;

생성한 Firewall Policy를 선택합니다.

**`VPC - AWS Network Firewall`**

![](<.gitbook/assets/image (41).png>)

새로운 Stateless Rule Group 생성을 합니다.

**`Create and add new stateless rule group`**

![](<.gitbook/assets/image (114).png>)

Stateless rule group을 생성합니다.

1. **Name : Stateless Rule 이름을 정의합니다.**
2. **Capacity : Rule Group의 Rule의 숫자를 정의합니다.(최대 10,000개)**
3. **Priority : Stateless Rule의 Priority를 정의합니다. Rule 번호를 의미하며 Rule 번호가 우선 순위를 가지게 됩니다. (NACL과 규칙 동일)**
4. **Protocol : 프로토콜을 정의합니다.**
5. **Source IP/Port**&#x20;
6. **Destination IP/Port**
7. **Action : Pass/Drop/Forward to stateful rule groups 를 선택합니다.**&#x20;
8. **Add rule : 생성한 Rule을 추가합니다.**

![](<.gitbook/assets/image (51).png>)

![](<.gitbook/assets/image (56).png>)

Rule을 추가하면 , 추가된 Rule 을 확인하고 생성완료합니다.

![](<.gitbook/assets/image (111).png>)

생성한 룰을 확인하기 위해 외부에서 인스턴스의 공인 IP 주소로 ICMP를 요청해 봅니다. 10.1.1.101에 Mapping 된 공인 IP주로로 ICMP가 거부되고, 10.1.1.102에 Mapping된 공인 IP주소는 응답합니다. (NACL과 유사합니다.)

![](<.gitbook/assets/image (40).png>)

### 4. Stateful Rule 구성&#x20;

새로운 Stateful Rule Group 생성을 합니다.

**`VPC-Firewall policies - 생성한 Policy - Stateful rule groups - Add rule groups - Create and add new stateful  rule group`**

![](<.gitbook/assets/image (105).png>)

Stateful rule group을 생성합니다.

1. **Name : Stateful Rule 이름을 정의합니다.**
2. **Capacity : Rule Group의 Rule의 숫자를 정의합니다.(최대 10,000개)**
3. **Stateful rule group options : 5 tuple을 선택합니다.**
4. **Protocol : 프로토콜을 정의합니다.**
5. **Source IP/Port**&#x20;
6. **Destination IP/Port**
7. **Traffic direction : Any/Forward를 선택합니다 .**&#x20;
8. **Action : Pass,Drop,Alert 을 선택합니다.**

**SSH 에 대한 정책을 임의로 생성해 봅니다. (10.1.1.101 인스턴스에 대한 SSH Drop)**

![](<.gitbook/assets/image (125).png>)

![](<.gitbook/assets/image (57).png>)

Rule을 추가하면 , 추가된 Rule 을 확인하고 생성완료합니다.

![](<.gitbook/assets/image (123).png>)

&#x20;앞서 생성한 Cloud9에서 각 EC2 인스턴스로 ssh 접속을 실행해 봅니다.

```
ssh -i ~/environment/anfwkey.pem ec2-user@ec2-101-public-ip
ssh -i ~/environment/anfwkey.pem ec2-user@ec2-102-public-ip
```

&#x20;

Stateful Rule에 의해서 , EC20-102(10.1.1.102) 인스턴스만 접속이 가능합니다.

![](<.gitbook/assets/image (60).png>)

### 5. Stateful Domain list Rule 구성&#x20;

새로운 Stateful Rule Group 생성을 합니다.

**`VPC-Firewall policies - 생성한 Policy - Stateful rule groups - Add rule groups - Create and add new stateful  rule group`**

![](<.gitbook/assets/image (109).png>)

Stateful rule group을 생성합니다.

1. **Name : Stateful Rule 이름을 정의합니다.**
2. **Capacity : Rule Group의 Rule의 숫자를 정의합니다.(최대 10,000개)**
3. **Stateful rule group options : Domain list을 선택합니다.**
4. **Domain list - Rule에 정의할 도메인을 정의합니다. (예. www.google.com)**
5. **Protocol : HTTP/HTTPS 를 선택합니다.**
6. **Action : Allow/Deny 선택합니다.**

**www.google.com을 Filtering하는 예제를 설정해 봅니다.**

![](<.gitbook/assets/image (31).png>)

![](<.gitbook/assets/image (98).png>)

[Task.VPC Route Table 구성-4.트래픽 흐름 확인](single-az-nwfw.md#4) 에서 구성한 CloudShell에서 2개의 창을 열고, 아래과 같은 명령을 통해 각각의 인스턴스에 Session Manager를 통해 접속합니다.

```
aws ssm start-session --target $VPC1_AZA_101
sudo -s
su ec2-user
cd ~
curl -I www.google.com

```

```
aws ssm start-session --target $VPC1_AZA_102
sudo -s
su ec2-user
cd ~
curl -I www.google.com

```

아래와 같이 모든 인스턴스에서 www.google.com 의 접속이 filtering 됩니다.

![](<.gitbook/assets/image (38).png>)

### 6. Suricata IPS Rule 구성&#x20;

Suricata는 IDS(탐지)/IPS(탐지,차단)가 가능한 Open source 도구 입니다. Snort와 완벽하게 호환이 가능하며, 멀티 쓰레드 지원과 GPU 지원등으로 성능 부분에서 높은 평가를 받고 있습니다. (2020년 부터 Snort 3.0 출시와 함께 멀티 쓰레지 지원)

AWS Network Firewall의 Stateful IPS는 Suricata IPS를 통해서, Deep Inspection구현이 가능합니다. 또한 상용도구와 연계도 가능합니다. (2020년 12월 현재 기준 Fortinet 지원- 상용)

새로운 Stateful Rule Group 생성을 합니다.

**`VPC-Firewall policies - 생성한 Policy - Stateful rule groups - Add rule groups - Create and add new stateful  rule group`**

Stateful rule group을 생성합니다.

1. **Name : Stateful Rule 이름을 정의합니다.**
2. **Capacity : Rule Group의 Rule의 숫자를 정의합니다.(최대 10,000개)**
3. **Stateful rule group options : Suricata IPS Rule을 선택합니다.**
4. **Suricata IPS Rule을 설정합니다.**

**www.google.com을 Filtering하는 예제를 설정해 봅니다.**

![](<.gitbook/assets/image (128).png>)

![](<.gitbook/assets/image (18).png>)

IPS Rule은 아래와 같이 구성해 봅니다.

```
# 10.1.1.101 을 소스로 Contents에 AWS가 포함되면 Alert을 발생.
alert tcp 10.1.1.101 any -> any any (msg: "No access to the EC2-1 Webpage"; content: "AWS"; sid: 101; rev:1;)
alert tcp 10.1.1.102 any -> any any (msg: "No access to the EC2-1 Webpage"; content: "AWS"; sid: 102; rev:1;)
# 10.1.1.101,10.1.1.102 를 접속하는 User Agent가 Firefox 브라우저는 Drop.
drop http any any -> [10.1.1.101,10.1.1.102] any (msg: "User agent"; http.user_agent; content:"Firefox"; sid:103; rev:1;)

```

{% hint style="info" %}
Suricata Rule은 [https://suricata.readthedocs.io/en/latest/index.html](https://suricata.readthedocs.io/en/latest/index.html) 을 참고하여서 , 정책을 생성할 수 있습니다.
{% endhint %}

각 인스턴스에 접속해서,  아래 명령을 통해 접속 하거나, Web 브라우저에서 접속해 봅니다.

```
#EC2-101
curl -I http://ec2-102-public-ip/ec2meta-webpage/index.php
#EC2-102
curl -I http://ec2-101-public-ip/ec2meta-webpage/index.php

```

사용자 브라우저에서, Firefox와 Chrom을 통해서 EC2-101,102의 공인 IP 주소로 접속해 봅니다. 아래에서 처럼 Firefox는 접속되지 않습니다.

![](<.gitbook/assets/image (12).png>)

## Task 5. Network Firewall Logging 및 모니터링.

### 1.Loggin 목적지 설정

Network Firewall은 Logging 목적지를 3가지 지원합니다.

* S3 - bucket name과 Prefix를 설정합니다.
* Cloudwatch - Cloudwatch Log group을 지정합니다.
* Kinesis data firehose - Kinesis data firehose delivery stream name을 설정합니다.

Cloudwatch log group을 지정하고, Log를 살펴봅니다.

**`Cloudwatch - Cloudwatch logs - log groups`** 를 선택하고, **`Create log group`** 을 선택해서 Log Group을 생성합니다.

![](<.gitbook/assets/image (3).png>)

Alert , Flow log group을 각각 생성합니다.

* Log group name : NWFW-Alert , NWFW-Flow

![](<.gitbook/assets/image (7).png>)

생성된 Log group을 확인합니다.&#x20;

![](<.gitbook/assets/image (68).png>)

### 2. Firewall logging 구성.&#x20;

이제 다시 Network Firewall에서 Logging 구성을 진행합니다.

**`VPC-Firewalls- 생성한 Firewall`**&#x20;

![](<.gitbook/assets/image (99).png>)

Firewall details 메뉴를 선택하고, logging 메뉴에서 Edit 를 선택합니다.

**`VPC -Firewalls - 생성한 Firewall - Firewall details - Logging - Edit`**

![](<.gitbook/assets/image (127).png>)

firewall loggig을 구성합니다.

1. **log type - Alert, Flow 로그를 선택합니다.**
2. **log destination for alert - Alert logging 목적지를 선택합니다.**
3. **log destination for flows - flow logging 목적지를 선택합니다.**

Lab 에서는 앞서 이미 생성한 CloudWatch log group을 선택합니다.

![](<.gitbook/assets/image (43).png>)

### 3. Firewall Logging 확인.

사용자 랩탑에서 EC2 101,102 의 공인 주소로 Firefox로 접속해 봅니다.

각 인스턴스에 접속해서,  아래 명령을 통해 접속 하거나, Web 브라우저에서 접속해 봅니다.

```
#EC2-101
curl -I http://ec2-102-public-ip/ec2meta-webpage/index.php
#EC2-102
curl -I http://ec2-101-public-ip/ec2meta-webpage/index.php

```

사용자 브라우저에서, Firefox와 Chrom을 통해서 EC2-101,102의 공인 IP 주소로 접속해 봅니다. 아래에서 처럼 Firefox는 접속되지 않습니다. 관련 로그를 CloudWatch에서 확인해 봅니다.

![](<.gitbook/assets/image (12).png>)

Cloudwatch에서 Alert log를 선택합니다.

**`Cloudwatch - Cloudwatch logs - log groups - Alert log`**

![](<.gitbook/assets/image (83).png>)

Block 된 로그를 확인합니다. Signature ID 103에 의해서 Block 된 것을 확인 할 수 있습니다.

```
# 10.1.1.101,10.1.1.102 를 접속하는 User Agent가 Firefox 브라우저는 Drop.
drop http any any -> [10.1.1.101,10.1.1.102] any (msg: "User agent"; http.user_agent; content:"Firefox"; sid:103; rev:1;)
```

![](<.gitbook/assets/image (102).png>)

### 4. Firewall Monitoring

Firewall 에 대한 간단한 모니터링을 아래 메뉴를 통해서 확인 할 수 있습니다.

VPC - Firewall - 생성한 Firewall - Monitoring

* **Stateless ReceivedPackets**
* **Stateless DroppedPackets**
* **Stateless PassedPackets**
* **Stateful ReceivedPackets**
* **Stateful DroppedPackets**
* **Stateful PassedPackets**

![](<.gitbook/assets/image (65).png>)

## Task6. 자원 삭제

1.Network Firewall  policy 에서 Rule 제거

**`VPC - Firewall Policies - Firewall Polices 선택 - Stateleess Rule/Stateful rule group 제거`**&#x20;

2\. Route Table에서 GWLB Endpoint 제거

VPC-IGW-RT 에서 Egde Associations 제거 / VPC-IGW  제거.

**`Virtual Private Cloud - Route Table - VPC1-IGW-RT - Edge Associations - IGW Uncheck`**

VPC-ProtectSubnet1-RT 에서 GWLB Endpoint route 제거

**`Virtual Private Cloud - Route Table - VPC1-ProtectSubnet1-RT 선택 - GWLB Endpoint Route 제거`**

3\. Network Firewall logging 구성 제거

**`VPC - Firewall - Firewall Details - Logging - Edit - Loggig 해제.`**

4\. Network Firewall 제거

**`VPC - Firewall  - 제거`**

5\. Cloudformation 에서 Stack 제거

**`Cloudformation - Stacks - Stack 선택 - 삭제`**&#x20;
