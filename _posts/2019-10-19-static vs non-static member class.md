---
title: "static vs non-static member class"
permalink: /java/effective/24
categories: java static-member-class non-static-member-class 
---
> effective java item 24. 멤버 클래스는 되도록 static 으로...

# Nested class : static vs non-static
* 특정 클래스 내부에서만 사용할 목적으로 클래스를 정의할 때 nested class 를 정의하곤 한다. 그런데 지금까지 나는 무의식적으로 static nested class 로만 정의했는데, outer class 와 인스턴스 생성, 메모리 공간에 이유가 있었다.
* [Oracle Java Tutorials](https://docs.oracle.com/javase/tutorial/java/javaOO/nested.html) 에 나온 설명
  > Terminology: Nested classes are divided into two categories: static and non-static. Nested classes that are declared static are called static nested classes. Non-static nested classes are called inner classes.
    ```java
    class OuterClass {
        static class StaticNestedClass {}
        class InnerClass {}
    }
    ```
  * Nested class 는 크게 static 과 non-static 으로 구분할 수 있고, static 으로 선언된 Nested class 를 static nested class 라고 부른다. 그리고 non-static nested class 를 inner class 라고 부른다.

## non-static member class, inner class
* non-static 멤버 클래스의 인스턴스는 outer class 의 인스턴스와 암묵적으로 연결된다. 그래서 non-static nested class(inner class) 에서 outer class 를 참조할 수 있다. `클래스명.this` 형태로 바깥 클래스의 인스턴스 참조를 가져올 수 있다.
* 바깥 인스턴스의 생성 없이 non-static member class 의 인스턴스도 생성할 수 없다. 즉, outer class 의 인스턴스 없이 생성할 수 없고, 바깥 클래스의 인스턴스 메소드에서 inner class 의 인스턴스를 생성하는 방법이 일반적이다. outer class 의 인스턴스를 이용하여 `outerInstance.new XClass()` 로 수동으로 생성할 수도 있지만, 외부에서 non-static nested class 를 사용한다면 이 경우는 차라리 새로운 Outer class 로 빼서 정의하는게 낫겠다.
* inner class 에서 outer class 의 멤버에 접근
  * 오라클 가이드를 보니 아래 예시를 Shadowing 이라는 단어로 설명하고 있고, static nested class 에서는 다음과 같이 접근할 수 없다. non-static nested class(inner class) 에서는 `XXXClass.this.x` 처럼 outer class 의 멤버 변수에 접근이 가능하다.
  * `Effective Java 에서는 아래 처럼 멤버 클래스에서 바깥 인스턴스에 접근할 일이 없다면 무조건 static 을 붙여서 static nested class 로 만들라고 한다.`
  ```java
  public class ShadowTest {
      public int x = 0;

      class FirstLevel {
          public int x = 1;

          void methodInFirstLevel(int x) {
              System.out.println("x = " + x);
              System.out.println("this.x = " + this.x);
              System.out.println("ShadowTest.this.x = " + ShadowTest.this.x);
          }
      }
    
      public static void main(String... args) {
          ShadowTest st = new ShadowTest();
          ShadowTest.FirstLevel fl = st.new FirstLevel();
          fl.methodInFirstLevel(23);
      }
  }
  ```
#### inner class 단점 정리
* shadowing 예시를 보면 멤버 클래스가 outer class 에 대한 숨은 참조를 가지게 된다. 이 참조를 저장하려면 시간과 공간이 소비된다. 그리고 `가비지 컬렉션이 outer class 의 인스턴스를 수거하지 못하는 메모리 누수가 생길 수 있다는 문제가 있다.`
  
## static nested class
* static nested class 는 outer class 인스턴스 생성 없이 독립적으로 존재할 수 있다. 
* 대표적인 static nested class 의 예로 Map 과 Entry 의 관계가 있다. Map 인스턴스는 key-value 을 를 직접 관리하지 않고, 키-값 쌍을 표현하는 Entry 를 이용한다. Entry 가 매번 생성될때마다 Map(Outer class) 에 대한 참조를 각각 가지게 된다면 메모리 생성 및 관리에서 비효율적인 것이다. 그래서 Entry 는 static nested class 로 정의되어 있다.
  ```java
  public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    
      static class Node<K,V> implements Map.Entry<K,V> {
      
      }
  }
  ```
