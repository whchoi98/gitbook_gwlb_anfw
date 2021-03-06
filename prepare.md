---
description: 'Update : 2022-06-12/ 20min'
---

# 사전 준비

## 시작에 앞서&#x20;

GWLB & ANFW(AWS Network Firewall) Workshop은 AWS 워크로드를 보호하기 위해서, TGW를 활용한 Centralized Design 방법으로 GWLB와 ANFW을 구성하는 방법을 제공합니다

#### GWLB Design  -  TransitGateway를 기반으로 Centralized 기반으로 구성합니다. 외부에 서비스를 제공하기 위해 ALB를 구성합니다. 내부 Private Subnet 자원들의 패치를 위해 NAT Gateway를 구성합니다.

보안 어플라이언스는 상용 방화벽이나 기타 어플라이언스를 연동 가능합니다. 이 랩에서는 리눅스 기반 IPTABLE을 사용합니다.

#### ANFW Design  -  TransitGateway를 기반으로 Centralized 기반으로 구성합니다. 외부에 서비스를 제공하기 위해 ALB를 구성합니다. 내부 Private Subnet 자원들의 패치를 위해 NAT Gateway를 구성합니다.

AWS의 완전관리형 방화벽을 구성해서, 내외부 보안 정책을 구현해 봅니다. &#x20;

## Cloud9 구성

### Cloud9 소개&#x20;

AWS Cloud9은 브라우저만으로 코드를 작성, 실행 및 디버깅할 수 있는 클라우드 기반 IDE(통합 개발 환경)입니다. 코드 편집기, 디버거 및 터미널이 포함되어 있습니다. Cloud9은 JavaScript, Python, PHP를 비롯하여 널리 사용되는 프로그래밍 언어를 위한 필수 도구가 사전에 패키징되어 제공되므로, 새로운 프로젝트를 시작하기 위해 파일을 설치하거나 개발 머신을 구성할 필요가 없습니다. Cloud9 IDE는 클라우드 기반이므로, 인터넷이 연결된 머신을 사용하여 사무실, 집 또는 어디서든 프로젝트 작업을 할 수 있습니다. 또한, Cloud9은 서버리스 애플리케이션을 개발할 수 있는 원활한 환경을 제공하므로 손쉽게 서버리스 애플리케이션의 리소스를 정의하고, 디버깅하고, 로컬 실행과 원격 실행 간에 전환할 수 있습니다. Cloud9에서는 개발 환경을 팀과 신속하게 공유할 수 있으므로 프로그램을 연결하고 서로의 입력 값을 실시간으로 추적할 수 있습니다.

### Cloud9 구성

Cloud9을 실행하기 위해 아래와 같이 AWS 관리콘솔에서 **`"Cloud9"`** 을 입력합니다.

![](<.gitbook/assets/image (87).png>)

**`AWS 관리 콘솔 - Cloud9 - Create environment`**를 선택합니다.

* name : mycloud9

![](<.gitbook/assets/image (203) (1) (1) (1).png>)

모든 설정을 기본값으로 사용하고, 인스턴스타입은 t3.small ,Cost-Saving Setting Never로 변경합니다. 절전모드로 변경되는 것을 방지하게 됩니다. 다음 진행 버튼을 계속 누르고 Cloud9을 생성합니다.

* instance type : t3.small
* Cost-saving setting : Never
* 기타 옵션 : 기본

![](<.gitbook/assets/image (88).png>)

2\~3분 후에 Cloud9 이 동작하는 것을 확인 할 수 있습니다. Cloud9 창에서 "+" 버튼을 누르고 New Terminal을 띄워서 터미널을 생성합니다. 추가로 "+"를 계속 생성하게 되면 Terminal을 다중으로 사용할 수 있습니다.

![](<.gitbook/assets/image (15).png>)

Cloud9 IDE는 이미 AWS CLI가 설치되어 있습니다. 하지만 기본 1.x 버전이 설치되어 있습니다.

```
$ aws --version
aws-cli/1.19.39 Python/2.7.18 Linux/4.14.225-169.362.amzn2.x86_64 botocore/1.20.39
```

아래 명령을 통해 CLI를 2.0으로 업그레이드합니다.

```
# AWS CLI upgrade
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

```

정상적으로 업그레이드 되었는지 확인합니다.

```
source ~/.bashrc
aws --version

```

aws cli 자동완성을 설치 합니다.

```
# aws cli 자동완성 설치 
which aws_completer
export PATH=/usr/local/bin:$PATH
source ~/.bash_profile
complete -C '/usr/local/bin/aws_completer' aws

```



Cloud9 권한 부여

이 랩에서는 ap-northeast-2에서 생성한 Cloud9이 다른 리전의 자원을 제어하도록 구성되어 있습니다. 권한을 부여해서 사용합니다.&#x20;

먼저 아래와 같이 IAM에서 역할을 선택합니다.&#x20;

* IAM - 역할 - 역할 만들기

![](<.gitbook/assets/image (217).png>)

* AWS 서비스 - EC2 선택

![](<.gitbook/assets/image (228).png>)

* 권한 추가 - AdministratorAccess 선택

![](<.gitbook/assets/image (223).png>)

* 역할 세부 정보에 역할 이름을 입력합니다. (예. Cloud9-Admin)
* 하단의 역할 생성을 선택합니다.&#x20;

![](<.gitbook/assets/image (213) (1).png>)

Cloud 9의 역할을 변경하기 위해서, IAM 역할을 수정합니다

* AWS 서비스 - EC2 - 인스턴스 - Cloud9 선택 - 작업 - 보안 - IAM 역할 수정

![](<.gitbook/assets/image (212) (1).png>)

* IAM의 역할을 위에서 만든 역할 이름으로 변경하고 업데이트 합니다.&#x20;

![](<.gitbook/assets/image (208).png>)

Cloud9 화면의 우측 상단 톱니바퀴를 선택하고, 아래에서 처럼 AWS Settings의 Credential 을 변경합니다.&#x20;

![](<.gitbook/assets/image (222) (1).png>)



## keypair 만들기

keypair를 Cloud9에서 생성합니다.

```
ssh-keygen

```

key이름은 gwlbkey 로 설정합니다.

```
mykey
```

아래와 같이 ssh key가 구성됩니다.

```
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ec2-user/.ssh/id_rsa): mykey
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in gwlbkey.
Your public key has been saved in gwlbkey.pub.
The key fingerprint is:
SHA256:ZId12JDdlSjIuhBym08BKU/EtYbMj9EkCYtTwYpP9sY ec2-user@ip-172-31-63-114.ap-northeast-2.compute.internal
The key's randomart image is:
+---[RSA 2048]----+
|  .+=+=o. +*....o|
|  o++B=..=ooo... |
|.o..*=++* . .    |
|..+  === .       |
| + o .+.S        |
|  . E  o         |
|   .             |
|                 |
|                 |
+----[SHA256]-----+
```

Cloud9 Terminal 에서 생성되는 Public EC2들에 대한 접근을 할 수 있도록 아래와 같이 구성합니다.

```
mv ~/environment/mkey ~/environment/mykey.pem
chmod 400 ./mykey.pem
```

이제 생성된 Public Key를 계정으로 업로드 합니다. **`"--region {AWS Region}"`** 리전 옵션에서 각 리전을 지정하게 되면 해당 리전으로 생성한 Public Key를 전송합니다. 아래에서는 도쿄,서울, 버지니아, 오레곤 리전으로 전송하는 예제입니다.

```
#Tokoy Region 전송 
aws ec2 import-key-pair --key-name "mykey" --public-key-material fileb://mykey.pub --region ap-northeast-1
#Seoul Region 전송
aws ec2 import-key-pair --key-name "mykey" --public-key-material fileb://mykey.pub --region ap-northeast-2
#버지니아 리전 전송
aws ec2 import-key-pair --key-name "mykey" --public-key-material fileb://mykey.pub --region us-east-1
#오레곤 리전 전송
aws ec2 import-key-pair --key-name "mykey" --public-key-material fileb://mykey.pub --region us-west-2

```

아래와 같이 업로드가 완료됩니다.

```
 $ aws ec2 import-key-pair --key-name "mykey" --public-key-material fileb://mykey.pub --region ap-northeast-2
{
    "KeyFingerprint": "xx:xx:xx:xx:xx:65:3a:70:fb:b1:fa:dd:6c:59:c6:9e",
    "KeyName": "gwlbkey",
    "KeyPairId": "key-xxxxxxxxx"
}

```

정상적으로 public key가 업로드되었는지 AWS 관리콘솔에서 확인합니다.

**`AWS 관리 콘솔 - EC2 - 네트워크 및 보안 - 키페어`**

![](<.gitbook/assets/image (213).png>)

## Session Manager Plugin 설치

Session Manager는 AWS Systems Manager의 일부 기능입니다. Session Manager에서, EC2 인스턴스, 엣지 디바이스, 온프레미스 서버 및 가상 머신(VM)을 관리할 수 있습니다. 브라우저 기반 셸 또는 AWS Command Line Interface(AWS CLI)을 사용할 수 있습니다. Session Manager는 인바운드 포트를 열고, 배스천 호스트를 유지하고, SSH 키를 관리할 필요 없이 보안성과 감사 가능성을 갖춘 노드 관리 기능을 제공합니다.&#x20;

또한 Session Manager를 통해 노드에 대한 제어된 액세스를 요구하는 회사 정책, 엄격한 보안 관행을 손쉽게 준수하고, 관리형 노드 액세스 세부 정보가 포함된 완전히 감사 가능한 로그를 제공합니다

Workshop에서는 Cloud9에 아래와 같이 Plugin을 설치하면, Private Subnet의 인스턴스에 접속 할 수 있습니다

```
#session manager plugin 설치
cd ~/environment
curl "https://s3.amazonaws.com/session-manager-downloads/plugin/latest/linux_64bit/session-manager-plugin.rpm" -o "session-manager-plugin.rpm"
sudo yum install -y session-manager-plugin.rpm
git clone https://github.com/whchoi98/useful-shell.git

```

**이제 사전 구성이 완료되었습니다.**



