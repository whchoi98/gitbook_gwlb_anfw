---
description: 'update : 2022-06-12'
---

# ANFW 소개

## Overview

AWS Network Firewall은 Amazon Virtual Private Cloud (Amazon VPC)에서 생성 한 가상 사설 클라우드 (VPC)를위한 Stateless,Stateful 기반 관리 형 네트워크 방화벽 및 IDS/IPS 서비스입니다.

![](<../.gitbook/assets/image (210) (1) (1).png>)

## 장점

네트워크 방화벽을 사용하면 VPC 내부 트래픽을 필터링 할 수 있습니다. 터넷 게이트웨이, NAT 게이트웨이 또는 VPN 또는 AWS Direct Connect를 통해 들어오고 나가는 트래픽 필터링이 포함됩니다. 네트워크 방화벽은 Stateful 기반 보안 위해 오픈 소스 IPS (침입 방지 시스템) 인 Suricata를 사용합니다. 네트워크 방화벽은 Suricata 호환 규칙을 지원합니다.

네트워크 방화벽을 사용하여 다음과 같은 다양한 방법으로 Amazon VPC 트래픽을 모니터링하고 보호 할 수 있습니다.

* 허용하 AWS 서비스 도메인 또는 Amazon S3와 같은 IP 주소 엔드 포인트에서만 트래픽을 전달합니다.
* 차단이 필요 도메인의 사용자 지정 목록을 사용하여 애플리케이션이 액세스 할 수있는 도메인 제한합니다.
* VPC에 들어 오거나 나가는 트래픽에 대해 패킷 검사를 수행합니다.
* Stateful 프로토콜 감지를 사용하여 사용되는 포트에 관계없이 HTTPS와 같은 프로토콜을 필터링합니다.

VPC에 대해 네트워크 방화벽을 활성화하려면 Amazon VPC와 네트워크 방화벽 모두에서 단계를 수행합니다.

이 워크샵에서는 다양한 디자인과 함께 Network Filrewall을 적용하는 방안들을 살펴 봅니다.



