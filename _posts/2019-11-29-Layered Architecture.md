---
title: "Layered Architecture"
permalink: /architecture/layered-architecture
categories: layered-architecture
---

https://dzone.com/articles/layered-architecture-is-good 번역 및 정리

>  layered architecture 란 `관심사의 분리(SOC, Separation Of Concern)`를 위해 코드를 계층화하는 것

Layered Architecture 의 프로젝트의 구조는 4개의 패키지로 구성된다. 각 Layer 마다 주요 관심사가 구분되어 있고, 관련된 객체들만 가지기로 한다.

#### Presentation (또는 Interface Layer)

최종적으로 User 에게 전달되는 **UI** 또는 Client 에게 Response 를 되돌려주는 것에 대한 책임을 가지는 클래스를 포함한다.

#### Application

애플리케이션의 **기능적인 요구사항**을 충족시키기 위한 로직을 포함한다. 그리고 이것은 Domain 의 역할과는 다르다. 대부분의 시스템에서 Application Layer 는 Domain Objects 를 조작하는 Services 로 구성되어 **Use Case 시나리오를 수행**한다.

#### Domain

거의 **Domain entity** 들로 구성되고, <u>Service 도 일부</u> 가지고 있다. **비즈니스 규칙**들은 Domain Layer 에 있어야 한다.

#### Infrastructure (또는 Persistence Layer)

Database 에 데이터를 persisting 하거나, DAO, Repository 와 같은 **기술적인 작업을 담당하는 책임**을 가진다.

#### Layered Architecture 의 중요한 규칙

Layered Architecture 를 올바로 구현하기 위해서는 두 가지 중요한 규칙이 있다.

1. 모든 참조는 **단 방향**으로 가야 한다. 

   - presentation → application → domain → infrastructure

   ![Image title](https://dzone.com/storage/temp/4277164-layered-architecture-overview.png)

2. **관심사**를 지켜야 한다. 각 Layer 의 개념과 관련없는 로직은 위치해서는 안된다. 예를 들어 도메인 로직 또는 DB query 가 UI(Presentation Layer) 에 있으면 안된다.

## Layered Architecture

Layered Architecture 의 main idea 는 `관심사의 분리(SOC, Separation Of Concerns)` 이다. 위에서 말했지만, UI 작업에 domain 또는 db code 가 섞이는 것을 피해야 한다. Layered Architecture 는 프로젝트의 설계나 구현에 대해서는 알려주지 않는다. 이를 DDD(도메인 주도 개발)와 같은 다른 아키텍쳐 프로세스로 보완해야 한다. DDD 와 같은 옵션을 선택하지 않아도, Layered Architecture 는 소스 코드를 어떻게 구성하는지 가이드를 제공해준다.

## Layering Spring Pet Clinic

* Spring 기반의 예제 프로젝트 : https://github.com/spring-projects/spring-petclinic

![Image title](https://dzone.com/storage/temp/4277165-architecture-comparison.png)

왼쪽은 Spring 팀에서 sample 프로젝트로 제공하는 Spring Pet Clinic 이다. 프로젝트 패키지 아래 도메인별로 상위 도메인이 펼쳐져 있다. 반면, 오른쪽은 Layered Architecture 로 구성된 Pet Clinic 이다. (application service 에 해당하는 service 클래스가 빠져있는 예제이지만...)

위 이미지에서는 Repository interface 가 infrastructure 에 존재한다. 그런데 다른 Layered Architecture 나 DDD start 책에서는 repository interface 는 Domain Layer 에 있어야하고, Infrastructure Layer 에서 그 인터페이스의 구현체가 위치한다.

DDD start! 책을 보면 domain 레이어 하위에 아래 세 가지 기능의 객체들이 있다.

* domain
  1. model
  2. service : 도메인 서비스는 한 애그리거트로 기능을 구현하기 힘들 때, DomainService 를 통해 구현하기도 한다. 도메인에 대한 상태 값 없이 로직만 구현한다.
  3. repository

> ###### DDD start!
>
> 도메인 로직을 외부 시스템이나 RDBMS 와 같은 엔진을 이용해서 구현해야 할 경우 인터페이스와 클래스를 분리하게 된다. 도메인 영영에 도메인 서비스 인터페이스, 인프라스트럭처 영역에 실제 구현체를 둔다. DIP 방식으로 분리하면 결합도를 낮추어서, 도메인 영역이 특정 구현에 종속되는 것을 방지할 수 있고 도메인 영역에 대한 테스트가 수월해진다.

책에서 나온 내용으로는 특정 기술 구현에 종속되는 것을 방지한다는데, JPA 를 사용하면 인터페이스 자체가 특정 기술에 종속되어서 어디에 두어야 좋을지 헷갈린다...

> 하지만, 원본 글 중 좋아요가 많은.. 댓글을 보니까 Spring Pet Clinic 또한 Layered Architecture 라고 할 수 있다고 한다. Layer 를 나누는 것이 단지 패키지를 나누는 것과 100% 같다고 할 수 없다. 라고 하는 것 같다. 패키지를 4개로 나누는 것은 구현 방법 중 하나 일 뿐.

#### Layered Architecture 의 장점

- 간결성
- 일관성
- SOC 보장
  - 각 패키지의 관심사가 명확함

#### Layered Architecture 의 단점

- 확장성 없음
  - 프로젝트의 규모가 커지면 Layered Architecture 는 도움이 되지 못한다. 다른 아키텍쳐를 찾아야 한다.
- use case 가 숨겨진다?
  - 코드 구성 만 살펴보면 프로젝트가 무엇을 하고 있는지 알 수 없다. 최소한 클래스 이름을 읽거나, 구현까지도 읽어야 할 수도 있다.
  - 음...?
- 낮은 응집성
  - 일반적인 시나리오와 비즈니스 개념에 기여하는 클래스가 프로젝트의 기술적인 문제를 분리하도록 구성되어 있기 때문에 하나의 도메인 개념이 서로 멀리 떨어져 있을 수 있다.

#### 언제 Layered Architecture 를 적용하면 좋을까

* 간단함
  * 러닝 커브가 적다.
* 일관성
  * 여러 소규모의 프로젝트, microservices 에서 내부 아키텍쳐로 사용하기 좋다.
* SOC(Separation Of Concerns)
  * 경험이 적은 팀에 좋다. 
  * 음...? 명시적인 패키지 분리를 통해 강제로 관심사의 분리를 수행하기 때문에 ?
* 확장성 필요없는 경우 ?
  * microservices 와 같은 작은 프로젝트를 지원하는데 좋다. 또는 큰 프로젝트를 작은 조각으로 만들때 좋다.
* Hidden use cases and low-cohesion (숨겨진 유스케이스 및 낮은 응집도 ?)
  * 복잡한 비즈니스 로직 없이 단순 CRUD 연산 또는 간단한 REST API service 의 경우 Layered Architecture 는 좋다.
* Dependency Inversion 이 없는 경우
  * 음...? 현재 DIP 를 지키지 않는 경우, DIP 를 쉽게 지키기 위해 Layered Architecture 를 기반으로 개발해보라는건가 ?