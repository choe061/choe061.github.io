---
title: "DDD - Domain model, Aggregate"
permalink: /ddd/domainmodel
categories: ddd DomainModel Aggregate


---

# DDD - Domain model, Aggregate

## Domain Model

#### Domain ?

* 도메인은 소프트웨어로 해결하고자 하는 문제의 영역이다. 그래서 해결하고자 하는 문제가 다르면, 도메인이 다르게 해석될 수 있다. 예를 들어 주문개발팀의 Order 는 다른 개발팀의 Order 보다 더 세분화된 케이스, 더 많은 기능을 포함하고 있을 것이다.
* 하나의 큰 도메인은 작은 하위 도메인으로 분리할 수 있다. 하나의 큰 Shop 에서 Member, Order, Product 등등의 도메인으로 분리할 수 있는 것 처럼.

#### Domain 의 핵심

* 도메인이 제공하는 `기능`
* 도메인의 주요 데이터 구성

## Aggregate

#### Aggregate ?

* 관련 Entity 와 Value Object 를 하나로 묶은 것이고, 가장 상위 도메인을 Aggregate Root 라고 한다. 관련 모델을 Aggregate 에 묶음으로써 확인할 수 있는 특징이 두 가지가 있다.
  * 상위 수준에서 모델을 조망할 수 있다.
  * Aggregate 에 속한 객체들은 유사하거나 동일한 life-cycle 을 가진다.
* Aggregate 들의 관계만 파악해도 도메인 모델에서 전체 구조를 이해하는데 도움이 된다.

#### Aggregate Root 의 핵심 역할

* 관련 Entity 와 Value Object 에 대한 일관성 유지
* 도메인 기능 제공
* Aggregate Root 를 통해서만 도메인 로직을 구현한다. 상태 변경을 별도 클래스로 위임하기도 한다.

#### Aggregate 주의점

* (주의) 다른 Aggregate Root 객체를 변경/참조하면 안된다.
  * 한 Aggregate 에서 다른 Aggregate 를 수정하면 안된다. 애그리거트는 자신의 책임 범위를 확실히 지켜야 한다. 책임 범위를 넘어 간섭하면 안된다. 즉, Aggregate 는 서로 최대한 독립적이어야 한다.
  * 객체를 참조해서도 안된다. Aggregate Root 를 직접 참조하게 되면, 해당 Aggregate 의 상태를 변경할 수 있게되고, domain 로직이 분산되는 문제점이 있다. 로직이 분산되면 영향 범위를 파악하기 힘들고, 잘못 설계하면 순환 참조도 발생할 수 있다.
  * (JPA 를 사용하면 성능 문제도 발생, N + 1 문제)
  * 변경과 확장에 대한 어려움도 커진다.
* 위 문제에 대한 해결 방안은 Aggregate Root 간에는 ID 값을 이용한 참조를 하여 해결할 수 있다.

#### 도메인 모델을 위한 적용 규칙

* public setter 금지
  * 중요 도메인의 의미와 의도를 표현하지 못한다.
  * 도메인의 값을 변경하는 로직이 다른 응용 영역이나 표현 영역으로 분산될 수 있다.
* public getter 경고
  * 단순 getter 로 데이터를 제공하여 로직이 밖으로 분산되는 것을 막아야 한다.
  * 관련 데이터에 대한 로직은 도메인 모델에 작성하고, 의미있는 네이밍으로 메서드를 제공해야 한다.
* Value Object 는 불변으로 구현
  * public setter 의 연장선... 
  * 불변으로 구현하면, 외부에서 내부 state 를 변경하지 못하기 때문에 Aggregate 의 일관성이 깨질 가능성이 줄어든다. 즉, side-effect 가 줄어든다.

#### Repository 와 Aggregate

* Aggregate 는 개념상 완전한 한 개의 도메인 모델을 표현.
* 객체의 영속성을 처리하는 Repository
  * Repository 는 Aggregate 단위로 존재한다. Aggregate Root 에 관련된 Entity, Value Object 는 동일하거나 유사한 life cycle 을 가지기 때문에 영속화도 함께.

