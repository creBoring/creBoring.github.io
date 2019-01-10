---
layout: post
title: AWS VPC endpoint로 Systems Manager에 접근해보자
date: 2018-12-20 15:00:00 +0300
description: AWS VPC endpoint를 통해 인터넷에 엑세스되지 않는 EC2 인스턴스들을 대상으로 Systems Manager를 사용해 보았다. # Add post description (optional)
img: 2018-12-21/AWS-VPC-systems-manager.png # Add image post (optional)
tags: [AWS, AWS Systems Manager, vpc endpoint]
---

AWS Systems Manager를 정리하며, **Outbound** 인터넷이 되어야만 SSM Agent가 SSM 과 통신할 수 있다는 점이 매우 제한적으로 느껴졌는데, **VPC 엔드포인트** 를 사용하면 인터넷이 전혀 되지 않는 인스턴스에 대해서도 SSM을 통해 관리할 수 있다는 내용을 접하게 되어 테스트를 진행하고 게시글로 정리하게 되었습니다.

글에 대한 비판과 의견은 언제든지 환영합니다 :)

### 목차

- **[Systems Manager](#systems-manager)**

- **[VPC 엔드포인트](#vpc-엔드포인트)**

- **[Systems Manager에 대한 VPC 엔드포인트 설정](#systems-manager에-대한-vpc-엔드포인트-설정)**

- **[테스트](#테스트)**

- **[제약 조건](#제약-조건)**

---

<br>
## Systems Manager

우선은 현 게시글은 AWS Systems Manager가 어떤 기능인지 안다는 전제하에 작성되었기 때문에,<br>
현 게시글을 다루기 전에 Systems Manager가 어떤 서비스인지 모르는 분들은 다음 게시글을 먼저 읽고 오시는 것을 권해드립니다.

> AWS Systems Manager 를 파헤쳐 보자
<br>https://creboring.github.io/AWS-Systems-Manager-를-파헤쳐-보자/

---

<br>
## VPC 엔드포인트

우선 **VPC 엔드포인트**란, AWS에서 제공하는 서비스들 중 VPC에 속해있지 않는, 인터넷을 통해서만 통신이 가능한 서비스들을 VPC 내부에 endpoint를 생성하여 **사설망으로 접근할 수 있도록 하는 서비스** 입니다.<br>
AWS에는 VPC 내부에 생성되는 리소스가 있는가 하면, VPC와는 상관없이 Regional 하게 서비스되는 리소스들도 존재합니다. ex) S3, dynamodb, Systems Manager, ...

VPC endpoint는 두 가지 방식이 존재합니다.<br>
첫 번째는 **인터페이스 VPC 엔드포인트**이며, 두 번째는 **게이트웨이 VPC 엔드포인트** 입니다.

두 방식은 각각 다른 작동원리를 가지고 있으며, AWS 서비스별로 지원하는 엔드포인트 방식도 다릅니다.<br>
- 인터페이스 VPC 엔드포인트 : Systems Manager, KMS, SNS, ...
- 게이트웨이 VPC 엔드포인트 : S3, dynamodb

<img src="https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/images/vpc-endpoint-s3-diagram.png">

VPC endpoint 두 가지 방식의 작동원리를 간단하게 살펴보면 다음과 같습니다.<br>

#### 인터페이스 VPC 엔드포인트

인터페이스 VPC 엔드포인트는 AWS 리소스의 엔드포인트를 ENI(Elastic Network Interface) 즉 Network Interface로써 private IP를 지정하여 사용하는 방식입니다.<br>
인터페이스 VPC 엔드포인트 방식을 사용하면 실제로 해당 리소스에 대한 privaet IP가 VPC의 Subnet 내부에 생성됩니다.<br>
사용자는 해당 private IP를 통해 해당 리소스에 접근할 수 있게 됩니다.

#### 게이트웨이 VPC 엔드포인트

게이트웨이 VPC 엔드포인트는 대상 서비스로 향할 수 있는 게이트웨이를 만들어 라우팅테이블을 통해 모든 통신을 private 하게 되도록 합니다.

<img src="/assets/img/2018-12-21/gateway-vpc-endpoint.png"/>

> VPC 엔드포인트 공식문서<br>
https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpc-endpoints.html

<br>
이번 게시글에서 다룰 Systems Manager는 **인터페이스 형식**의 VPC 엔드포인트를 지원합니다.

---

<br>
## Systems Manager에 대한 VPC 엔드포인트 설정

본격적으로 인터넷에 연결되지 않은 VPC Subnet을 구성하고 EC2를 생성한 후에 VPC Endpoint를 이용해 리소스들을 관리해보겠습니다.

<br>
### VPC Private(인터넷에 연결 되지 않은) Subnet 생성

System Manager를 사용하기 위해서는 반드시 EC2 인스턴스의 Outbound 통신이 이루어져야 합니다.<br>
VPC 엔드포인트를 사용하면 이러한 제약조건을 신경쓰지 않아도 됩니다.<br>
VPC 엔드포인트를 생성하여 인터넷이 되지 않는 서브넷에 존재한 인스턴스를 System Manager로 관리해보겠습니다.

우선 VPC에 서브넷 하나와 라우팅테이블 하나를 생성하여 인터넷과 연결되지 않도록 설정하겠습니다.<br>
*(인터넷에 연결되면 당연히 잘 작동합니다) *

 1. AWS VPC 콘솔 페이지에 접속합니다. https://console.aws.amazon.com/vpc/home
 2. 좌측 탭에서 **라우팅테이블** 탭을 선택합니다.
 3. **[Create route table]** 버튼을 눌러 라우팅 테이블 하나를 만들어줍니다.
 4. 방금 만든 라우팅테이블에는 어떠한 게이트웨이도 연결하지 않았으므로 인터넷과 연결되지 않습니다.<br><br> 이제 새로운 서브넷을 만들어 방금 만든 라우팅테이블을 붙여줍시다.
 5. 좌측 탭에서 **서브넷** 탭을 선택합니다.
 6. **[서브넷 생성]** 버튼을 눌러 새로운 서브넷을 하나 만들어줍니다. CIDR 블록은 현재 VPC CIDR 내에서 남는 대역대로 잡아서 생성하시면 됩니다.
 7. 생성된 서브넷을 선택한 후 아래 상세 정보가 표시되면, 상세 정보에서 **라우팅 테이블 탭**을 선택합니다.
 8. **[라우팅 테이블 수정]** 버튼을 누른 후, 라우팅테이블을 방금 만든 라우팅테이블로 바꿔줍니다.

<br>
### System Manager VPC 엔드포인트 생성

이제 실제로 VPC에 연결시킬 VPC 엔드포인트를 생성해 보겠습니다.<br>
System Manager를 Private한 환경에서 사용하기 위해서는 3~4가지 엔드포인트를 생성해야합니다.<br>
기본적으로는 아래 세가지 VPC 엔드포인트가 필요하며, System Manager의 Session Manager 기능을 사용하시려는 경우 한 가지 엔드포인트가 추가로 필요합니다.<br>
- **com.amazonaws.region.ssm**
- **com.amazonaws.region.ec2messages**
- **com.amazonaws.region.ec2**
- com.amazonaws.region.ssmmessages (Session Manager를 사용할 경우)

자세한 내용은 아래 공식문서를 참고해주세요.

> 생성해야될 VPC 엔드포인트들에 대한 공식문서<br>
https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/sysman-setting-up-vpc.html#sysman-setting-up-vpc-create

1. AWS VPC 콘솔 페이지에 접속합니다. https://console.aws.amazon.com/vpc/home
2. 좌측 탭에서 **엔드포인트** 탭을 선택합니다.
3. **[엔드포인트 생성]** 버튼을 누른 뒤 4가지 엔드포인트를 생성해줍니다. 내부 항목은 아래와 같이 선택합니다.
  - 서비스 범주 = AWS 서비스
  - 서비스 이름 = 위에 언급한 4가지 서비스명을 하나씩 각각 선택하여 만들어줍니다.
  - VPC & 서브넷 = 앞에서 만들었던 서브넷과 서브넷이 속해있는 VPC를 선택해줍니다.<br>
  *(서브넷은 VPC 내부에 어떤 서브넷을 선택해도 상관없습니다.)*
  - 보안그룹 = VPC 엔드포인트에 연결될 보안그룹에는 프라이빗 서브넷에서 443 포트로 들어오는 연결을 허용해야 합니다.

> VPC Endpoint에 연결될 보안그룹 지정 시 주의사항은 아래 공식문서의 **'수신 연결'** 을 확인해주세요<br>
https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/sysman-setting-up-vpc.html#vpc-requirements-and-limitations

<img src="/assets/img/2018-12-21/system-manager-endpoint.png"/>

엔드포인트를 모두 만들 경우 각각의 엔드포인트는 네트워크 인터페이스를 할당받게 됩니다.<br>
엔드포인트들의 상태가 모두 ***'사용 가능'*** 이 되면 다음 스텝으로 넘어갑니다.

<br>
### SSM 관리형 인스턴스 생성

이제 인터넷에 연결되지 않은 공간에서 System Manager를 사용할 환경이 모두 갖춰졌습니다.<br>
System Manager로 관리할 EC2 인스턴스를 생성해줍시다.

1. AWS EC2 콘솔 페이지에 접속합니다. https://console.aws.amazon.com/ec2/home
2. 아까 만들었던 서브넷으로 하여 인스턴스를 하나 생성해줍니다. (AMI는 System Manager Agent가 기본적으로 설치 되어 있는 Amazon Linux 2 로 해보겠습니다)
3. 생성된 인스턴스에 SSM 권한을 부여해줄 **IAM Role**을 생성해야 합니다. <br>AWS IAM 콘솔 페이지에 접속합니다. https://console.aws.amazon.com/iam/home
4. 좌측의 **역할** 탭을 선택한 후 **[역할 만들기]** 버튼을 누릅니다.
5. 역할을 사용할 서비스는 EC2로 선택해주신 후 ***'AmazonEC2RoleforSSM'*** 이라는 정책을 연결하여 역할을 생성합니다.
6. EC2 콘솔 페이지로 돌아간 후 방금 만든 역할을 EC2에 부여해줍니다.<br>
*(EC2 인스턴스를 우클릭 후 **인스턴스 설정** -> **IAM 역할 연결/바꾸기** 를 선택하면 역할을 부여할 수 있습니다)*

잠시 기다리면 System Manager 페이지에서 방금 생성한 인스턴스가 관리형 인스턴스로 표시되는 것을 확인할 수 있습니다.<br>
또한 Session Manager, Run Command 등의 작업을 진행할 수 있게 되었습니다.

---

<br>
## 테스트

VPC Endpoint를 이용해 private 환경에서 System Manager로 EC2 인스턴스에 여러 작업을 걸어보겠습니다.

<br>
### 테스트 환경

**< System Manager 관리형 인스턴스가 구성되어 있습니다. >**
<img src='/assets/img/2018-12-21/test-state1.png' />

**< 인터넷에 연결되지 않은 서브넷에 VPC Endpoint가 구성되어 있습니다. >**
<img src='/assets/img/2018-12-21/test-state2.png' />

<br>
### Session Manager

Session Manager 연결 시도 시 별다른 이상없이 접근이 가능했습니다.<br>
정상적으로 서버에 ssm-user 로 접근했으며 내부에서 명령어도 잘 작동합니다 :)

<img src='/assets/img/2018-12-21/session-manager-test.png' />

<br>
### Run Command

Run Command 를 통해 ***AWS-RunShellScript Document*** 를 실행시켜 보았습니다.<br>
파라미터로는 "ls -al" 을 주었으며 결과값이 정상적으로 출력됩니다.

<img src='/assets/img/2018-12-21/run-command-test.png' />

<br>
### Maintenance Windows

Maintenance Windows(유지 관리 기간) 또한 잘 작동합니다.<br>
**'자동화'**를 Maintenance Windows로 작동시켰으며 이 때 실행한 Document는 ***AWS-StopEC2Instance*** 입니다.

<img src='/assets/img/2018-12-21/maintenance-windows-test.png' />

---

<br>
## 제약 조건

System Manager를 Private 환경에서 VPC endpoint를 통해 사용하려면, 아무래도 public하게 인터넷으로 사용하는 것보단 제약조건이 존재합니다.<br>

- aws:domainJoin 플러그인을 사용하는 SSM 문서에서 도메인으로 Windows 인스턴스를 조인하도록 요구하는 요청이 실패합니다.
- VPC 엔드포인트는 현재 교차 리전 요청을 지원하지 않습니다.
- VPC Endpoint에 연결된 보안 그룹은 관리형 인스턴스의 프라이빗 서브넷에서 443 포트로 들어오는 연결을 허용해야 합니다.<br>
*(System Manager 통신이 443 포트로 이루어지기 때문입니다)*
- VPC 엔드포인트 정책이 Amazon S3 버킷에 대한 몇몇 액세스를 허용해야 합니다

더 자세한 내용은 아래 공식문서를 참고하시기 바랍니다.

> VPC Endpoint로 System Manager 사용 시 제약 조건<br>
https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/sysman-setting-up-vpc.html#vpc-requirements-and-limitations

---

### 참고 문헌

> AWS System Manager Docs<br>
https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/what-is-systems-manager.html

> AWS VPC Endpoint Docs<br>
https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpc-endpoints.html

### 이미지 참고

> AWS icon<br>
https://aws.amazon.com/ko/architecture/icons/

> VPC Endpoint 아키텍처 그림<br>
https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/vpce-gateway.html
