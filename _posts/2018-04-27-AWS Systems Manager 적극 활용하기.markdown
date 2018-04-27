---
layout: post
title: AWS Systems Manager 적극 활용하기
date: 2018-04-27 15:00:00 +0300
description: AWS Systems Manager에 대해서와 사용법, 활용법에 대해 정리한 글이다. # Add post description (optional)
img: 2018-04-27/AWS-system-manager.png # Add image post (optional)
tags: [AWS, Systems Manager, 사용법]
---

# AWS Systems Manager

고객 리소스 관리의 효율을 높이기 위해 **AWS Systems Manager** 를 검토해 보았습니다.<br>
하기 내용들은 작성자의 주관적인 의견으로 게시글 하단에 참조링크를 남깁니다.

### 목차

 - **[AWS Systems Manager 란?](#AWS-Systems-Manager-란?)**

 - **[AWS Systems Manager 사용법](#AWS-Systems-Manager-사용법)**

 - **[AWS Systems Manager 활용하기](#AWS-Systems-Manager-활용하기)**

---

## AWS Systems Manager 란?

AWS Systems Manager 란 AWS에서 여러 리소스들을 **중앙 집중화**하여 인프라 구성을 논리적으로 나누고 작업을 자동화할 수 있는 **관리형 서비스**입니다.

<img src='/assets/img/2018-04-27/AWS-system-manager-icon.png'>

이전에는 Amazon EC2 Systems Manager 라는 이름으로 EC2 에서만 사용할 수 있던 서비스였지만, <br>
현재는 EC2 뿐만아니라 RDS, CloudFormation, 등.. 더 넓은 범위의 리소스 관리가 가능해졌습니다.<br>
(하지만 역시 EC2 리소스 관리 측면이 더 많이 존재합니다)

 ※*AWS Systems Manager는 리전별로 격리된 서비스입니다.*

AWS Systems Manager의 기능은 아래의 대표적인 4가지 기능이 존재합니다.
 - **리소스 그룹**(Resource Group)
 - **인사이트**(Insight)
 - **Actions**
 - **공유 리소스**(Shared Resource)

각 기능들에 대해 소개드립니다.

### 리소스 그룹(Resource Group)

리소스 그룹은 이름 그대로 AWS 리소스들을 묶어주는 서비스로 운영계, 개발계와 같이 서비스들을 **논리적**으로 구분지어 관리할 수 있습니다.

<img src='/assets/img/2018-04-27/tags.png'>

실제로 리소스 그룹은 **리소스 유형**과 **태그 값**을 이용한 **쿼리문**으로 구성되며,<br>
직접 리소스들을 수동적으로 추가하는 것이 아닌, 위에서 설정한 쿼리문에 적합한 리소스들을 대상으로 여러가지 작업을 진행할 수 있습니다.<br>
즉, 리소스 그룹은 **정적이지 않으며 동적으로 변경됩니다**.
(예: 리소스 그룹에 포함되어 있던 인스턴스 하나가 삭제되면 자동으로 리소스 그룹에서도 제거되고, 리소스 그룹 쿼리에 적합한 인스턴스가 새로 생성되면 자동으로 리소스 그룹에 추가 됩니다.)

### 인사이트(Insight)

### Actions

### 공유 리소스(Shared Resource)

---

## AWS Systems Manager 사용법


---


## AWS Systems Manager 활용하기


### 참고 문헌

> AWS Docs<br>
https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/what-is-systems-manager.html
