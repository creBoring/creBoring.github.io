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

## Systems Manager

<br>
우선은 현 게시글은 AWS Systems Manager가 어떤 기능인지 안다는 전제하에 작성되었기 때문에,<br>
현 게시글을 다루기 전에 Systems Manager가 어떤 서비스인지 모르는 분들은 다음 게시글을 먼저 읽고 오시는 것을 권해드립니다.

> AWS Systems Manager 를 파헤쳐 보자
<br>https://creboring.github.io/AWS-Systems-Manager-를-파헤쳐-보자/

---

## VPC 엔드포인트

<br>
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

<br>
#### 게이트웨이 VPC 엔드포인트



---

## Systems Manager에 대한 VPC 엔드포인트 설정

<br>


---

## 테스트

<br>


---

## 제약 조건

<br>

---

### 참고 문헌


### 이미지 참고

> AWS icon<br>
https://aws.amazon.com/ko/architecture/icons/
