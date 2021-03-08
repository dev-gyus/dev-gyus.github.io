---
layout: post
title:  "[Spring-Boot] @ConfigurationProperties에 대하여"
subtitle:   "Properties에 설정한 값을 특정 클래스 필드에 바인딩해주는 애노테이션"
date: 2021-03-09
categories: Spring
tags: Spring-Boot
comments: true
---

# @ConfigurationProperties
---
>특정 클래스에 선언하여 어플리케이션의 Properties 혹은 YAML에 설정해둔 프로퍼티값을 필드에 바인딩 시켜주는 애노테이션.

무언가 공통으로 사용해야하는 값들을 보통 Properties나 YAML에 설정해두고 MessageSource를 이용해 사용하는것과 달리 설정파일에 정의해둔 prefix를 애노테이션의 prefix로 설정해두면 properties파일에서의 .을 기준으로 다음에 오는 문자열을 애노테이션을 선언한 클래스의 필드로 자동 바인딩해주는 기능이다.
***

즉, 아래와 같이 properties파일에 설정을 해뒀다면,
<pre>
<code>app.host=http://localhost:8080
app.name="예제"
</code>
</pre>

<pre>
<code>@ConfigurationProperties("app")
@Getter @Setter
public class exBindClass{
  private String host;
  private String name;
}
</code>
</pre>

위 처럼 바인딩 받을 클래스에 .을 기준으로 왼쪽에 있는 값을 prefix로 주면, 스프링 의존 주입시 .다음에 오는 문자열과 같은 필드명을 가진 멤버에 자동으로 바인딩을 해주는 것이다. ( Setter 선언을 해줘야된다.)


> 이제 해당 클래스를 빈 설정만 해주면 의존 주입 받아 바인딩된 값을 가진 객체로 사용할 수 있다.

스프링부트에서 설정하는 방법은 아래 2가지가 존재한다.

* @EnableConfigurationProperties(exBindClass.class)를 설정클래스에 선언
* @Configuration 이 선언된 클래스에 빈 등

***

Properties값은 여러 클래스에 중복 사용되는 필드값을 바인딩된채 주입받아 손쉽게 이용할 수 있다는 장점이 있으므로 애플리케이션의 유지/보수 측면에서 꽤 유용하게 사용되는데, 이 값을 애노테이션 2개로 바인딩 설정하여 사용할수있다는것은 굉장한 매리트가 있어보인다.
결국 또 배웠으니 까먹지 말아야겠다.


>코드나, 설명에 잘못된 점이 있거나, 추가해주실 사항이 있다면 댓글로 남겨주세요.
