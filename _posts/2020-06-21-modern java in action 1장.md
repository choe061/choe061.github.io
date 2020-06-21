---
title: "ModernJavaInAction 1장"
permalink: /java/modern/1
categories: java ModernJavaInAction
---

# Modern Java In Action 1장 및 관련 내용 추가

1장에서는 Java 8 에서 나온 Stream, Lambda, Interface 의 default/static method 의 특징도 설명하지만, 등장한 이유도 설명한다. 사용법은 어디서든 쉽게 찾아볼 수 있는데, 이런 내용은 자세히 정리된 곳이 없어서 좋은 것 같다.

추가로 책 내용 외에 도큐먼트에서 찾아본 내용도 정리해보았다.

## Java 8 에서 나온 특징

#### 스트림

- 연속적인 데이터를 입력 스트림으로 받아서 출력 스트림으로 하나씩 기록
- 유닉스 명령어처럼 명령어 파이프라인을 만들 수 있다. 자바 8 에도 Stream API 로 일련의 데이터를 처리하기 위한 파이프라인을 쉽게 구성할 수 있다.
- 덤으로 병렬성까지

#### 동작 파라미터화

* 메서드 파라미터로 코드 전달

- 자바 8 이전에는 primitive type 값이나 객체의 인스턴스만 전달이 가능했다.
  - 자바 7 이하에서 메서드와 클래스는 이급 시민이지만, **자바 8 부터 Lambda 와 Method reference 를 사용하여 일급 시민으로 사용할 수 있게되었다.** 
  - 자바 7 이하에서는 new 연산자로 생성된 인스턴스만 전달이 가능했지만, **자바 8 부터는 메서드 자체(코드)를 메서드 인수로 넘겨줄 수 있다.** 덕분에 코드를 간결하게 작성할 수 있고, bolierplate code 가 없어지면서 가독성이 좋아졌다.

#### 병렬처리

* Stream 을 사용하여 덤으로 병렬성을 얻을 수 있는데, 제약조건??이 있다. 병렬 실행이 되기 때문에 하나의 Stream 파이프라인에서 외부에 가변 데이터를 공유하면 안된다. 코드가 정상적으로 동작하지 않을 수 있고, 어떤 side-effect 가 생길지 모른다.
* 스트림을 병렬로 처리하는 경우, CPU 가 두 개 이상이면, 각 CPU 가 작업을 나누어 처리하고 그 결과를 합친다.
  * 즉, 프로그램이 실행되는 동안 컴포넌트 간 상호작용이 일어나면 안된다. Java stream 의 각 실행, 파이프라인은 상호작용하지 않는 것을 가정하기 때문에 외부 가변 데이터를 공유해선 안되고 서로 독립적으로 수행해야 한다는 것 같다. 또한 그래서 side-effect 가 없는 함수를 스트림에서 사용해야 하는 것 같다.
* Stream 에서 사용하는 메서드는 수학적인 함수의 의미로 사용해야 한다. side effect 가 없는 함수로 만들어야 한다.

#### interface default method

* default method 를 설명하기 전에, 기존 메서드 중 Collections.sort(list) 를 잠깐 설명한다. 자바 8이전에는 기능을 추가할때 하위 호환성을 지키기 위해 Collections 와 같은 유틸성 클래스에 기능을 추가하였고, 위 메소드처럼 Collections 에 sort 가 추가되었다. 그럼 stream 도 Collections 나 다른 유틸성 클래스로 만들면 되었을텐데, 왜 default method 를 만들고 list 에 추가했을까?

* 하지만 기존 List, Map 등 인터페이스의 하위 호환성을 지키며 인터페이스를 수정하는 것은 기존의 방법으로는 불가능했다. 그래서 Java 8 에서 기존 인터페이스를 수정하면서 하위 호환성을 지키기 위해 디폴트 메서드가 추가되었다.

  * 예를들어 메소드 시그니처를 보면 **MemberUtils.move(member) 이런 형태보다 member.move() 가 훨씬 깔끔**해보인다. 그리고 **Member 를 다루는 메소드가 한 곳으로 모이면서 응집도도 올라간다.** 마찬가지로 Collections.stream(members) 는 members.stream() 이 비해 어색하고 깔끔한 코드처럼 보이지 않는다. interface 에 새로운 기능을 추가하면서 members.stream() 방식을 사용하기 위해 default method 가 생긴 것이다.

  * 자바 8의 List 인터페이스부터는 sort 디폴트 메서드가 추가되었다.

    * ```java
      default void sort(Comparator<? super E> c) {
          //...
      }
      ```

#### interface static method

* 개발을 하면서 interface 에 static method 를 작성해본적은 없었다. static method 가 생긴 이유에 대해 oracle java doc 과 bealdung 에서 한 가지 이유로 static method 를 인터페이스에 추가한다고 설명한다.

  * oracle java docs 에서 interface 의 static method 는 핵심, 필수 메서드보단 utility methods 로서 간주한다.

    > ###### Oracle java docs 설명
    >
    > If they add them as static methods, then programmers would regard them as utility methods, not as essential, core methods.

  * baeldung 에서도 유틸리티 메서드로서 개념을 가진다고 한다.

    * static method 는 객체에 포함되지 않으며, 해당 interface 를 상속받는 구현체의 API 의 일부가 아니다. 그냥 interfaceName.method 로 호출할 뿐이다.
    * interface static method 의 main idea 는 해당 interface 와 관련된 utils 성 메서드를 인터페이스에 작성하여 응집도를 높이는 효과를 제공한다고 한다.

      > ###### Baeldung 설명
      >
      > Since *static* methods don't belong to a particular object, they are not part of the API of the classes implementing the interface, and they have to be **called by using the interface name preceding the method name**.
      >
      > To understand how *static* methods work in interfaces, let's refactor the *Vehicle* interface and add to it a *static* utility method:
      >
      > The idea behind *static* interface methods is to provide a simple mechanism that allows us to **increase the degree of [cohesion](https://en.wikipedia.org/wiki/Cohesion_(computer_science))** of a design by putting together related methods in one single place without having to create an object.

* interface 의 static method 결론
  * 기존 static method 처럼 유틸성 method 를 만들때 사용한다. 그런데 차이점은 관련된 유틸리티 메서드를 한곳에 모아 응집도를 높이기 위함이다.
  * static 메서드는 특정 객체에 속하지 않기 때문에 interfacce 를 상속받는 구현체의 API 라고 볼 수 없다. 그렇기 때문에 필수 메서드보단 utility methods 로서 간주한다.
* 참고
  * oracle java docs : https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html
  * baeldung : https://www.baeldung.com/java-static-default-methods#static-interface-methods

#### Optional\<T\>

* 일반적인 함수형 언어에서는 서술형 데이터 형식을 이용해 null 회피 기법을 제공.
* Java 8 에서 NullPointerException 을 피하기 위한 Optional 클래스를 제공.

* 구조적(structural) 패턴 매칭 기법 ???
  * 자바에서 if-else, switch case 를 대체하기 위해
  * 자바 8 에서는 아직 패턴 매칭을 완벽하게 지원하지 않음