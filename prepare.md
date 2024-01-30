---
description: 'Update : 2024-01-27/ 20min'
---

# 사전 준비

## 1.시작에 앞서&#x20;

GWLB & ANFW(AWS Network Firewall) Workshop은 AWS 워크로드를 보호하기 위해서, TGW를 활용한 Centralized Design 방법으로 GWLB와 ANFW을 구성하는 방법을 제공합니다

#### GWLB Design  -  TransitGateway를 기반으로 Centralized 기반으로 구성합니다. 외부에 서비스를 제공하기 위해 ALB를 구성합니다. 내부 Private Subnet 자원들의 패치를 위해 NAT Gateway를 구성합니다.

보안 어플라이언스는 상용 방화벽이나 기타 어플라이언스를 연동 가능합니다. 이 랩에서는 리눅스 기반 IPTABLE을 사용합니다.

#### ANFW Design  -  TransitGateway를 기반으로 Centralized 기반으로 구성합니다. 외부에 서비스를 제공하기 위해 ALB를 구성합니다. 내부 Private Subnet 자원들의 패치를 위해 NAT Gateway를 구성합니다.

AWS의 완전관리형 방화벽을 구성해서, 내외부 보안 정책을 구현해 봅니다. &#x20;

## 2.Cloud9 구성

### Cloud9 소개&#x20;

AWS Cloud9은 브라우저만으로 코드를 작성, 실행 및 디버깅할 수 있는 클라우드 기반 IDE(통합 개발 환경)입니다. 코드 편집기, 디버거 및 터미널이 포함되어 있습니다. Cloud9은 JavaScript, Python, PHP를 비롯하여 널리 사용되는 프로그래밍 언어를 위한 필수 도구가 사전에 패키징되어 제공되므로, 새로운 프로젝트를 시작하기 위해 파일을 설치하거나 개발 머신을 구성할 필요가 없습니다. Cloud9 IDE는 클라우드 기반이므로, 인터넷이 연결된 머신을 사용하여 사무실, 집 또는 어디서든 프로젝트 작업을 할 수 있습니다. 또한, Cloud9은 서버리스 애플리케이션을 개발할 수 있는 원활한 환경을 제공하므로 손쉽게 서버리스 애플리케이션의 리소스를 정의하고, 디버깅하고, 로컬 실행과 원격 실행 간에 전환할 수 있습니다. Cloud9에서는 개발 환경을 팀과 신속하게 공유할 수 있으므로 프로그램을 연결하고 서로의 입력 값을 실시간으로 추적할 수 있습니다.

### Cloudshell 기반 생성

Cloud9을 실행하기 위해 아래와 같이 AWS 관리콘솔에서 **`"Cloudshell"`** 을 사용해서 구성합니다.

<figure><img src=".gitbook/assets/image (233).png" alt=""><figcaption></figcaption></figure>

`아래와 같이 iam user와 패스워드, user를 위한 Policy를 생성해서 연결합니다.`

```
export user_name=user01
export pass_word=1234Qwer
aws iam create-user --user-name ${user_name}
aws iam create-login-profile --user-name ${user_name} --password ${pass_word} --no-password-reset-required
aws iam attach-user-policy --user-name ${user_name} --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

제공된 AWS 계정에 손쉽게 접근하기  위해서 Alisa를 생성합니다. Alias는 고유해야 하므로 , 중복되지 않도록 합니다.

```
aws iam create-account-alias --account-alias ${alias-name}
```

앞서 생성한 Alias로 접속하고, 새로운 User로 인증해서 로그인 합니다.

정상적으로 접속하면, 다시 CloudShell을 사용해서 , Cloud9을 구성합니다.

```
# 아래 git을 cloudshell에 복제합니다.
git clone https://github.com/whchoi98/useful-shell.git

# cloud9 을 생성합니다.2~3분 시간이 소요됩니다.
~/useful-shell/create-cloud9.sh

```

### Cloud9 환경 구성

**`AWS 관리 콘솔 - Cloud9 - Create environment`**를 선택합니다.

생성된 Cloud9을 콘솔에서 열고, 새로운 터미널을 생성합니다.

<figure><img src=".gitbook/assets/image (234).png" alt=""><figcaption></figcaption></figure>

![](<.gitbook/assets/image (15).png>)

새로운 터미널에서 필요한 도구를 설치합니다.

```
git clone https://github.com/whchoi98/useful-shell.git
~/environment/useful-shell/c9-tool-set.sh

```

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
mv ~/environment/mykey ~/environment/mykey.pem
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

**이제 사전 구성이 완료되었습니다.**



