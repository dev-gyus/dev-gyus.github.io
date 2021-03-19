---
layout: post
title:  "[Spring-Boot] 정적자원 외부 경로 지정"
subtitle:   "이미지, JS, CSS등의 정적자원의 외부 경로 지정"
date: 2021-03-19
categories: Spring
tags: Spring-Boot
comments: true
---

# SpringBoot 정적자원 외부 경로 지정  
***
> 스프링부트에서 정적자원(이미지,JS,CSS 등)을 기본 디렉토리로 저장해두면 추후 웹어플리케이션이 새로 업데이트될경우 모두 지워져버리기 때문에 반드시 외부 경로를 지정해서 저장해야한다.

legacy spring framework에선 이미지를 저장하기위해 WebMvcConfigurer의 addResourceHandler에서 경로를 설정해서 사용하는게 일반적이기때문에 크게 신경쓰지 않아도 될 수 있으나, spring boot의 경우 기본 설정에서 WAS의 프로젝트경로를 자동으로 사용하기때문에, 결국 설정해줘야 된다.


설정법은 다음과 같다.
<pre>
<code>
@Configuration
public class WebConfig implements WebCofigurer{
  ... 생략
  @Override
  public void addResourceHandler(ResourceHandlerRegistry registry){
    registry.addResourceHandler("/upload/**") // 정적자원이 참조하는링크 ex) <img src="rootContext/upload/이미지파일이름" />  
    .addResourceLocation("file:///실제경로") // file:// + /경로
  }
}
</code>
</pre>  


유의해야할 점은 addResourceLocation("url") url에 file://을 prefix로 반드시 선언해주고, 그다음에 정적자원이 저장될 위치를 절대경로로 지정해주면 된다.

간단하지만 굉장히 중요하고, 또 막상 잘 안되면 안됨의 무한루프가 반복되어 시간손실이 어마해지니.. 꼭 잊지말고 기억해두자.
