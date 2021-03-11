---
layout: post
title:  "[Java] @Retention, @Target에 대하여"
subtitle:   "@Retention, @Target애노테이션 분석"
date: 2021-03-11
categories: Java
tags: false
comments: true
---
> 커스텀 애노테이션을 만들때 애노테이션의 메모리유지 범위, 대상을 설정하는 애노테이션

## @Retention
***
커스텀 애노테이션의 생성에서 해당 애노테이션이 선언된 대상(@Target의 속성값)의 메모리를 언제까지  
유지 할 것인지 결정하는 애노테이션.
@Retention은 RetentionPolicy라는 enum타입값을 인자로 받아 해당 policy값으로 메모리 유지타임을 결정한다.  
RetentionPolicy에는 3가지 정책이 있으며, 각각의 설명은 다음과 같다

> RetentionPolicy.SOURCE : javac(자바 컴파일러)가 자바코드를 컴파일할때 해당 애노테이션의 메모리를 버림 = 사실상 주석

> RetentionPolicy.CLASS : 컴파일러가 해당 애노테이션의 메모리를 유지하여 컴파일하지만, 실질적으로 JVM이 바이트코드를 해석해서 동작하는 Runtime단계에선 해당 메모리를 버린다 = 클래스파일에는 코드가 존재한다.

> RetentionPolicy.RUNTIME : JVM이 실제 클래스의 자바 바이트코드를 해석해서 동작하는 Runtime단계에서 메모리가 유지되고, Runtime이 종료되면 메모리도 사라진다.

즉, 요약하면
>1. SOURCE = 실제 동작할땐 영향안줌
>2. CLASS = 역시 어플리케이션 동작할땐 영향안줌
>3. RUNTIME = 어플리케이션 동작하는동안 항상 영향을 미침

실제 커스텀애노테이션을 만들어 사용하는건 결국 어플리케이션의 실행동안 개발자가 특정 목적을 가지고 사용하기위해 만드는것이므로, RUNTIME을 가장 많이 사용하고, 거의 이것만 사용한다고 생각해도 될것같다.
## @Target
***
커스텀 애노테이션이 적용될 대상을 지정하는 애노테이션으로 선언할 수 있는 enum값은 아래와 같다.

    클래스, 인터페이스에 선언
    TYPE,

    enum, 상수 포함 객체 필드에 선언
    FIELD,

    메소드에 선언
    METHOD,

    일반적인 파라미터에 선언
    PARAMETER,

    생성자에 선언
    CONSTRUCTOR,

    지역변수에 선언
    LOCAL_VARIABLE,

    애노테이션에 선언
    ANNOTATION_TYPE,

    패키지에 선언
    PACKAGE,

    매개변수의 타입에 선언
    TYPE_PARAMETER,

    매개변수 사용시 선언
    TYPE_USE  

나는 @Target의 속성으론 보편적인 파라미터에 다 붙일수 있는 PARAMETER를 많이 사용했던것같다.  
(SpringSecurity의 Authentication 바인딩해올때 아주 편하다.)  
역시 잊지말고 필히 기억해두자.
