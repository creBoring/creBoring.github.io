---
layout: post
title: AWS Systems Manager 적극 활용하기
date: 2018-04-27 15:00:00 +0300
description: AWS Systems Manager의 구성, 사용법, 활용법에 대해 정리한 글이다. # Add post description (optional)
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

<br>
AWS Systems Manager 란 AWS에서 여러 리소스들을 **중앙 집중화**하여 인프라 구성을 논리적으로 나누고 작업을 자동화할 수 있는 **관리형 서비스**입니다.<br>
다른 타 AWS 서비스들에 비해 실제 EC2, RDS 내부 리소스 영역까지 관리할 수 있는 몇 안되는 서비스 중 하나입니다.

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

<br>
### 리소스 그룹(Resource Group)

<br>
리소스 그룹은 이름 그대로 AWS 리소스들을 묶어주는 서비스로 운영계, 개발계와 같이 서비스들을 **논리적**으로 구분지어 관리할 수 있습니다.

<img src='/assets/img/2018-04-27/tags.png'>

실제로 리소스 그룹은 **리소스 유형**과 **태그 값**을 이용한 **쿼리문**으로 생성되는 집합으로,<br>
직접 리소스들을 추가하는 것이 아닌, 위에서 설정한 쿼리문에 부합한 리소스들이 그룹에 추가됩니다. 이 리소스들을 대상으로 여러가지 작업을 진행할 수 있습니다.<br>
즉, 리소스 그룹은 **정적이지 않으며 동적으로 변경됩니다**.
(예: 리소스 그룹에 포함되어 있던 인스턴스 하나가 삭제되면 자동으로 리소스 그룹에서도 제거되고, 새로 생성된 인스턴스가 리소스 그룹 쿼리에 적합한 경우 자동으로 리소스 그룹에 추가 됩니다.)

<br>
### 인사이트(Insight)

<br>
인사이트는 AWS 리소스들을 대상으로 한 여러 데이터들을 한 곳에 모아 보여주는 서비스입니다. <br>
CloudWatch와 같이 몇몇의 개별적인 AWS 서비스들 또한 인사이트 탭에서 확인할 수 있습니다.

인사이트의 주요 기능은 **인벤토리**와 **규정 준수**인데, 이는 리소스들에 **에이전트(Agent)**를 심고 이를 통해 전달받은 데이터를 기반으로 동작합니다.

**인벤토리**의 경우 더 정확하게 말하면 **인벤토리 수집** 기능으로, 인벤토리란 수집할 메타데이터들의 틀, 구성이라고 생각하면 됩니다.

<img src='/assets/img/2018-04-27/inventory.png'>

예를들면 네트워크 구성(IP, MacAddr, DNS, ...), Windows 업데이트(설치한 사람, 설치 날짜, ...), 애플리케이션(애플리케이션 이름, 버전, ...) 등과 같은 메타데이터 정보를 전달받을 인벤토리를 구성하고 구성에 맞는 데이터를 수집합니다.

**규정 준수**는 관리형 인스턴스 집합에 대해 패치 규정을 준수했는지 체크할 수 있으며, 구성의 일관성을 검사할 수 있습니다.<br><br>
인스턴스 집합의 데이터들을 수집한 후 규정에 맞지 않은 특정 인스턴스로 **드릴 다운(Drill down)**할 수 있습니다.<br>
기본적으로 구성 규정 준수는 *Systems Manager Patch Manager* 와 *Systems Manager State Manager* 에 대한 규정 준수 데이터를 표시하는데, Patch Manager를 통해 패치 규정을 준수했는지 체크할 수 있으며, State Manager를 통해 리소스의 연결 규정을 준수 했는지 검사합니다.

> *관리형 인스턴스는 "**공유 리소스**" 단계에서 더 자세히 다룹니다.*

<br>
### Actions

<br>
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

<br>
**AWS Systems Manager 자동화** : 일반적인 인스턴스나 시스템의 **유지 관리**와 **배포 작업**을 간소화하는 작업을 처리합니다. <br>
대표적으로 EC2, RDS 인스턴스 재시작이나 AMI, Snapshot 생성 등을 처리할 수 있으며 특정 인스턴스가 아니라 위에서 다룬 '리소스 그룹'을 대상으로 여러 리소스들에 한 번에 작업을 요청할 수도 있습니다.

<br>
**AWS Systems Manager Run Command** : 인스턴스 구성을 **원격**으로 관리하게 해주는 서비스입니다.<br>
AWS Systems Manager 자동화가 인스턴스, 시스템과 같은 인프라 수준의 작업이였다면, Run Command는 인스턴스 내부의 **Command line 수준**의 작업을 행할 수 있습니다.<br>
실제로 Run Command를 통해 Console 화면에서 Shell Script를 실행할 수도 있으며, 윈도우 업데이트, RunDockerAction 등 다양한 작업이 가능합니다.

<br>
**AWS Systems Manager Session Manager (NEW)** : Session Manager는 최근에 업데이트 된 기능으로, EC2 인스턴스의 **Shell**에 접근할 수 있는 기능을 제공합니다.
실제 공식문서에서 소개할 때도 **Bastion host 를 대체하는 기능**이라고 언급할 만큼 접근성이 용이하며, sudo 권한이 있어 admin 권한으로의 작업도 가능합니다.
다만, 한가지 아쉬운 점은 Windows 서버에 접근할 때도 GUI가 아닌 Power Shell 로만 접근이 가능하기 때문에 완전히 Bastion Host의 역할을 다해주기는 힘들 것으로 보입니다.

<img src='/assets/img/2018-04-27/session-manager.png'>
(위 캡처화면은 실제 Session Manager를 통해 shell에 접근한 모습입니다)

<br>
**AWS Systems Manager Patch Manager** : Patch Manager는 보안 관련 업데이트나 AWS OS 패치와 같은 패치들을 자동화 해줍니다. 위에서 다룬 인사이트의 규정 준수에서 사용된 Patch Manager가 바로 이 AWS Systems Manager Patch Manager 입니다.

하지만, Patch Manager는 생각보다 위에서 설명한 것보다 제한적으로 사용됩니다.<br>
패치 기준과 패치가 적용될 그룹을 지정할 수 있을 뿐, 실제 적용되는 패치는 임의로 지정이 불가능하며, AWS에서 제공하는 AMI에 문제점이 발견되었거나, 보안상 문제가 있어 새로운 패치를 적용하는 용도 등으로만 사용됩니다.

Patch Manager 콘솔 웹 페이지에 접속하여 실제 패치 목록들을 확인해보면 현재까지 제공되었던 패치 및 이슈 내역들을 확인하실 수 있습니다.

<br>
**AWS Systems Manager Maintenance Windows** : 한국어로 번역했을 때 '유지 관리 기간'으로, Actions의 다른 기능들인 'System Manager 자동화', 'System Manager Run Command' 와 같은 단발성 작업들을 **스케줄러를 통해 주기적으로 실행시켜주는 기능**입니다.

스케줄러의 일정은 cron 일정 작성법을 사용하거나 rate 일정 작성법을 사용할 수 있으며 시작과 종료날자를 지정할 수도 있습니다.

<img src='/assets/img/2018-04-27/update-image.jpg'>

<br>
**AWS Systems Manager State Manager** : State Manager란 관리형 인스턴스를 정의한 상태(state)로 유지하는 프로세스를 자동화합니다. 이 State Manager는 자체적으로 동작 방식이 Maintenance Windows와 유사합니다.<br>
어떠한 작업을 주기적으로 실행시켜주고, 실행했던 작업에 대해 재사용 및 관리가 편리합니다.

실제로 사용했을 때 Maintenance Windows와 큰 차이점을 느끼지 못해 AWS에 문의한 결과, Maintenance Windows와 State Manager 의 use-case에 대한 답변을 전달받았습니다.<br>
내용 아래 공유드립니다.

> **AWS 답변**<br><br>
As I understand, you would like to know the difference between using  the SSM 'Maintenance Window' and 'State Manager' as they both do a similar task of running functions periodically with Run Command/Patch Manager. Please let me know if this needs any correction.<br><br>
To directly answer your query, 'Maintenance Windows' will let you define a schedule for when to perform an action. Whereas, 'State Manager' automates the process of keeping your managed instances(EC2/Hybrid) in a defined state.<br><br>
- Like you rightly mentioned, there are some tasks/use-cases of 'State Manager' that can be achieved using 'Maintenance Windows'. However, there are some use-cases that can only be achieved or achieved easily with 'State Manager':<br>
**Example1**: For example, an association can specify that anti-virus software must be installed and running on your instances, or that certain ports must be closed. The association specifies a schedule for when the configuration is reapplied. The association also specifies actions to take when applying the configuration. For example, an association for anti-virus software might run once a day. If the software is not installed, then State Manager installs it. If the software is installed, but the service is not running, then the association might instruct State Manager to start the service.<br><br>
- Maintenance window lets you to schedule a task during non-business hours, whereas State manager ensures that a defined state is maintained for your application environment by running SSM documents periodically.<br><br>
>
> I hope you find this information useful.

즉, Maintenance Windows 만으로도 State Manager의 목적을 도달할 수 있지만, State Manager를 사용함으로써 조금 더 쉽게 도달할 수 있는 use-case 들이 있습니다.<br>
두 서비스는 각각 아래와 같은 상황에 사용됩니다.<br>
- **Maintenance Windows**는 업무 시간 이외의 시간에 작업을 예약할 수 있습니다.<br>
- **State Manager**는 SSM 문서(Action)을 주기적으로 실행하여 응용 프로그램 환경에 대해 정의한 상태가 유지되도록합니다.

> 여기서 **SSM 문서**는 위 Run Command 에서도 사용되었으며, System Manager에 정의된 작업을 의미합니다.<br>
'공유 리소스' 단계에서 더 상세히 알아보겠습니다.

<br>
### 공유 리소스(Shared Resource)

<br>
공유 리소스(Shared Resource)는 AWS Systems Manager에서 여러 동작에 필요한 자원들이나 리소스를 관리하는 서비스들입니다.

공유 리소스는 아래와 같은 리소스들이 존재합니다.

- AWS Systems Manager 관리형 인스턴스
- AWS Systems Manager 정품 인증
- AWS Systems Manager 문서(설명서)
- AWS Systems Manager Parameter Store

<br>
**AWS Systems Manager 관리형 인스턴스** : 관리형 인스턴스는 System Manager를 위해 구성된 EC2 인스턴스를 의미합니다.<br>
지금까지 쭉 설명해왔던 기능들은 대부분이 이 관리형 인스턴스를 대상으로 진행되는 작업들이며, System Manager를 통해 관리받고 있는 인스턴스를 의미합니다.

System Manager는 SSM Agent라고 하는 Agent를 EC2 내부에 구성한 후 이를 통해 통신하는 방식으로 작동합니다.

기본적으로 AWS에서 제공하는 **최신 AMI로 구성하는 인스턴스들은 기본적으로 SSM Agent가 탑재되어 있습니다.**<br>
다만 추가적으로 **IAM 권한** 설정 및 **Outbound Internet** 설정을 끝마쳐야만 실제 System Manager를 통해 관리할 수 있습니다.

해당 방법에 대해서는 'AWS Systems Manager 사용법' 에서 안내드리겠습니다.

> 기본적으로 SSM Agent가 구성되어 있는 AMI 리스트는 다음 AWS 공식문서를 참고해주세요
https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/systems-manager-prereqs.html

<br>
**AWS Systems Manager 정품 인증** :

<br>
**AWS Systems Manager 문서(설명서)** :

<br>
**AWS Systems Manager Parameter Store** :

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

> update<br>
https://pixabay.com/photo-1672385/
