---
layout: post
title: "Java 8 함수형 인터페이스"
date: 2018-11-15 00:13:00
description:  # Add post description (optional)
img:  # Add image post (optional)
---

Java 8에 추가된 함수형 인터페이스 대한 정리를 해보겠습니다.


함수형 인터페이스란? (Functional Interface)
~~~
    @FunctionalInterface
    public <T> interface Consumer {
        public void accept();
    }
~~~

@FunctionalInterface 로 Bulit-in-Annotation 선언한 인터페이스는 하나의 추상메서드만 가질 수 있습니다.

이후 람다 표현식 이나 메소드 래퍼런스를 이용해서 정의하여 사용할 수 있습니다.

JAVA8에서 지원하는 함수형 인터페이스는 아래와 같습니다.

BiConsumer<T,I>
~~~
        BiConsumer<String, String> concat = (str1, str2) -> System.out.println(str1 + " " + str2);
        concat.accept("Hello", "BiConsumer");
~~~
BIConsumer는 두 개의 파라미터를 메서드로 전달을 해주는 인터페이스입니다.
위의 예제는 두개의 파라미터를 받아 System.out.println()로 두 값을 공백으로 이어지도록 한다.

BIFunction<T,U,R>
~~~
        BiFunction<Object, Object, Boolean> typeCheckedIsString = (object1, object2) -> {
            return (object1 instanceof String && object2 instanceof String);
        };

        System.out.println(typeCheckedIsString.apply("A", 2));
~~~

BIFunction 인터페이스의 경우에 두 가지를 인자를 입력받아서 한가지를 리턴해주는 함수형 인터페이스로서
위의 예제는 두 개의 인자 모두 String Type인지 검증하는 로직이다.

Supplier<String> getter = () ->  "Hello Suppend";
System.out.println(getter.get());

Supplier 인터페이스의 경우엔 인자를 받지않으며 리턴타입만 존재하는 메소드가 있습니다.
위의 예제는 Hello Suppend라는 String을 getter로 부터 받아 출력하는 메소드 입니다.

Consumer<T>
~~~
Consumer emit =  e -> System.out.println(e);
emit.accept("Hello Consumer");
~~~

Consumer 인터페이스는 인자를 입력받아 소비하는 인터페이스다.

~~~
출력결과
Hello Consumer
~~~

Function<T,R>
~~~
Function<Integer, String> casting = e -> e.toString() + "Sec";
System.out.println(casting.apply(10));
~~~

Function 인터페이스는 T라는 제너릭 자료형을 입력받아 R의 제너릭 자료형으로 값을 리턴한다.
위의 예제는 Integer 형으로 10을 주었을때 뒤에 Sec를 붙이는 간단한 예제이다.

~~~
출력결과 
10Sec
~~~

BiPredicate<T,U>

~~~
    BiPredicate<String, String> compare = (str1, str2) -> str1.equals(str2);
	System.out.println(compare.test("s","s"));
~~~

BiPredicate는 두 개의 인자를 받아 Boolean형으로 값을 리턴해주는 함수형 인터페이스입니다.

위의 예제는 두개의 String형 변수의 값이 같은지 boolean으로 알려주는 것입니다.

Predicate
~~~
		Predicate<String> compare = str1 -> str1.isEmpty();
		System.out.println(compare.test("s"));
~~~

위의 소스를 보신봐와 Predicate, BiPredicate를 제외한 대부분의 인터페이스 제너릭으로 타입을 지정하는 이외에도
IntConsumer,longConsumer 처럼 타입별로 구현된 상세 인터페이스도 있으니 자세한 사항은 공식 문서를 직접 찾아보시는 게 좋을 거 같습니다.


참고한 사이트
https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html 
https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html