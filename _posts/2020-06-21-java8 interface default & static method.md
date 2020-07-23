---
title: "Java 8 interface default/static method"
permalink: /java/interface
categories: java8 interface default static
---

# Java 8 interface default/static method

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

    > ###### Oracle java docs
    >
    > If they add them as static methods, then programmers would regard them as utility methods, not as essential, core methods.

  * baeldung 에서도 유틸리티 메서드로서 개념을 가진다고 한다.

    * static method 는 객체에 포함되지 않으며, 해당 interface 를 상속받는 구현체의 API 의 일부가 아니다. 그냥 interfaceName.method 로 호출할 뿐이다.
    * interface static method 의 main idea 는 해당 interface 와 관련된 utils 성 메서드를 인터페이스에 작성하여 응집도를 높이는 효과를 제공한다고 한다.

    > ###### Baeldung
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

