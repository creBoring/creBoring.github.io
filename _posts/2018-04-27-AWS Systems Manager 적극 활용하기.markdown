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

 - **[AWS Systems Manager 란?](#aws-systems-manager-란)**

 - **[AWS Systems Manager 사용법](#aws-systems-manager-사용법)**

 - **[AWS Systems Manager 활용하기](#aws-systems-manager-활용하기)**

---

## AWS Systems Manager 란?

AWS Systems Manager 란 AWS에서 여러 리소스들을 **중앙 집중화**하여 인프라 구성을 논리적으로 나누고 작업을 자동화할 수 있는 **관리형 서비스 모음**입니다.

<img src='/assets/img/2018-04-27/AWS-system-manager-icon.png'>

이전에는 Amazon EC2 Systems Manager 라는 이름으로 EC2 에서만 사용할 수 있던 서비스였지만, <br>
현재는 EC2 뿐만아니라 RDS, CloudFormation, 등.. 더 넓은 범위의 리소스 관리가 가능해졌습니다.<br>
(하지만 역시 EC2 리소스 관리 측면이 더 많이 존재합니다)

 ※*AWS Systems Manager는 리전별로 격리된 서비스입니다.*

AWS Systems Manager는 아래의 4가지 기능을 제공합니다.
 - **리소스 그룹**(Resource Group)
 - **인사이트**(Insight)
 - **Actions**
 - **공유 리소스**(Shared Resource)

각 기능들에 대해 간단히 안내드립니다.

### 리소스 그룹(Resource Group)

리소스 그룹은 이름 그대로 AWS 리소스들을 묶어주는 서비스로 운영계, 개발계와 같이 서비스들을 **논리적**으로 구분지어 관리할 수 있습니다.

<img src='/assets/img/2018-04-27/tags.png'>

실제로 리소스 그룹은 **리소스 유형**과 **태그 값**을 이용한 **쿼리문**으로 생성되는 집합으로,<br>
직접 리소스들을 추가하는 것이 아닌, 위에서 설정한 쿼리문에 부합한 리소스들이 그룹에 추가됩니다. 이 리소스들을 대상으로 여러가지 작업을 진행할 수 있습니다.<br>
즉, 리소스 그룹은 **정적이지 않으며 동적으로 변경됩니다**.
(예: 리소스 그룹에 포함되어 있던 인스턴스 하나가 삭제되면 자동으로 리소스 그룹에서도 제거되고, 새로 생성된 인스턴스가 리소스 그룹 쿼리에 적합한 경우 자동으로 리소스 그룹에 추가 됩니다.)

### 인사이트(Insight)

인사이트는 AWS 리소스들을 대상으로 한 여러 데이터들을 한 곳에 모아 보여주는 서비스입니다. <br>
CloudWatch와 같이 몇몇의 개별적인 AWS 서비스들 또한 인사이트 탭에서 확인할 수 있습니다.

인사이트의 주요 기능은 **인벤토리**와 **규정 준수**인데, 이는 리소스들에 **에이전트(Agent)**를 심고 이를 통해 전달받은 데이터를 기반으로 동작합니다.

**인벤토리**의 경우 더 정확하게 말하면 **인벤토리 수집** 기능으로, 인벤토리란 수집할 메타데이터들의 틀, 구성이라고 생각하면 됩니다.

<img src='/assets/img/2018-04-27/inventory.png'>

예를들면 네트워크 구성(IP, MacAddr, DNS, ...), Windows 업데이트(설치한 사람, 설치 날짜, ...), 애플리케이션(애플리케이션 이름, 버전, ...) 등과 같은 메타데이터 정보를 전달받을 인벤토리를 구성하고 구성에 맞는 데이터를 수집합니다.

**규정 준수**는 관리형 인스턴스 집합에 대해 패치 규정을 준수했는지 체크할 수 있으며, 구성의 일관성을 검사할 수 있습니다.<br><br>
인스턴스 집합의 데이터들을 수집한 후 규정에 맞지 않은 특정 인스턴스로 **드릴 다운(Drill down)**할 수 있습니다.<br>
기본적으로 구성 규정 준수는 *Systems Manager Patch Manager* 와 *Systems Manager State Manager* 에 대한 규정 준수 데이터를 표시하는데, Patch Manager를 통해 패치 규정을 준수했는지 체크할 수 있으며, State Manager를 통해 리소스의 연결 규정을 준수 했는지 검사합니다.

> *관리형 인스턴스는 "**공유 리소스**" 단계에서 자세히 다룹니다.*


### Actions

Actions 에는 실제 AWS 리소스에 직접 행 할 수 있는 작업들을 제공합니다.
(여러 실제 리소스에 직접 작업을 진행하는 서비스들을 모아둔 탭입니다.)

Actions 에는 아래와 같은 작업들이 존재합니다.

- AWS Systems Manager 자동화
- AWS Systems Manager Run Command
- AWS Systems Manager Session Manager (**NEW**)
- AWS Systems Manager Patch Manager
- AWS Systems Manager Maintenance Windows
- AWS Systems Manager State Manager

실제 이 작업들은 위에서 다루었던 **리소스 그룹**, **인사이트** 와 연계하여 효율적으로 작업할 수 있습니다.<br>
(실제 Actions의 Patch Manager와 State Manager도 인사이트의 규정 준수에서 설명드렸던 기능입니다)

간략하게 하나씩 살펴보면 아래와 같습니다.

**AWS Systems Manager 자동화** : 일반적인 인스턴스나 시스템의 **유지 관리**와 **배포 작업**을 간소화하는 작업을 처리합니다. <br>
대표적으로 EC2, RDS 인스턴스 재시작이나 AMI, Snapshot 생성 등을 처리할 수 있으며 특정 인스턴스가 아니라 위에서 다룬 '리소스 그룹'을 대상으로 여러 리소스들에 한 번에 작업을 요청할 수도 있습니다.

**AWS Systems Manager Run Command** : 인스턴스 구성을 **원격**으로 관리하게 해주는 서비스입니다.<br>
AWS Systems Manager 자동화가 인스턴스, 시스템과 같은 인프라 수준의 작업이였다면, Run Command는 인스턴스 내부의 **Command line 수준**의 작업을 행할 수 있습니다.<br>
실제로 Run Command를 통해 Console 화면에서 Shell Script를 실행할 수도 있으며, 윈도우 업데이트, RunDockerAction 등 다양한 작업이 가능합니다.

<img src="/assets/img/2018-04-27/run-command.png">
(Run Command로 Sheel Script 실행하는 Console 화면의 일부분..)

**AWS Systems Manager Session Manager (NEW)** : -아직 업데이트되지 않았습니다-

**AWS Systems Manager Patch Manager** : Patch Manager는 보안 관련 업데이트와 같은 패치를 관리형 인스턴스에 자동화 해줍니다. 위에서 다룬 인사이트의 규정 준수에서 사용된 Patch Manager가 바로 이 AWS Systems Manager Patch Manager 입니다.

**AWS Systems Manager Maintenance Windows** :

**AWS Systems Manager State Manager** :

### 공유 리소스(Shared Resource)

---

## AWS Systems Manager 사용법


---


## AWS Systems Manager 활용하기


---

### 참고 문헌

> AWS Docs<br>
https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/what-is-systems-manager.html

### 이미지 참고

> AWS<br>
https://aws.amazon.com/ko/systems-manager/

> tag<br>
https://tealium.com/what-is-tag-management/

> inventory<br>
https://www.ispsystem.com/software/dcimanager/features/extended-inventory-management
