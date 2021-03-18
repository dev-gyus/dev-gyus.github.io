---
layout: post
title:  "[Spring-Security] 계정 정지 기능 구현 - 1"
subtitle:   "특정 사용자 로그인시 로그인을 차단하는 기능 구현"
date: 2021-03-16
categories: Spring
tags: Spring-Security
comments: true
---

# 계정잠금기능 설정하기
---
>운영하는 서비스에서 운영원칙에 어긋나거나 정상적인 이용을 하지 않는 유저를 대상으로 서비스 이용권한을 제한해야할 필요성이 있다.

서비스를 이용하는 유저가 단기간에 너무 많은 요청을 비정상적으로 한다거나, 특정 값을 입력하도록 제한했는데, 자바스크립트로 해킹하여 비정상적인 값을 대입하여 접근한다거나 등등 공개적인 서비스에서는 정상적인 이용을 하는사람이 대부분이지만, 소수의 비정상적인 이용을 하는사람에 의해 서버가 뻗어버리거나 DB가 엉망이 되어버리는 경우가 있을수있다.


계정 정지로는 사실 이를 원천적으로 막기는 어렵지만, 특이한 시도를 과도하게 하는것에 대한 방어수단으로서 특정 계정을 정지하여 더이상 사이트의 서비스를 이용할 수 없도록 하는건 어느 서비스에서나 흔히 볼 수 있는 제재정책이다.  

물론 이 페이지에서 설명하는 기능이 많이 부족한건 사실이나, 개념 공부차원에서는 충분히 매리트가 있을듯싶다.
***

구현에는 Spring-Secrity를 사용하였고, 1편의 기능 구현의 설명을 하자면  
>1. 계정 Entity에 계정이 정지되었는지 여부 판단을 위한 필드 생성
>2. 관리자가 정지시킬 계정의 1번에서 생성한 필드값 조정 및 DB저장
>3. 해당 유저 로그인시 계정 정지여부 확인 및 정지계정일경우 로그인정보가 일치해도 실패하도록 구현

구현코드는 아래와 같다.
- 멤버Entity
<pre>
<code>@Getter
public class Member{
  ... 생략
  private boolean locked;
}
</code>
</pre>

실제 로그인시 UserDetailsService의 loadUserByUsername메소드에서 UserDetails로 반환되어 Authentication객체에 저장될 로그인 전용 DTO클래스를 정의한다.

<pre>
<code>@Data
public class MemberLogin{
  private Member member;
}
</code>
</pre>

그리고, Spring-Security에서 로그인 성공시 후속작업을 진행하기 위해  
AuthenticationSuccessHandler를 상속한 클래스를 정의한다.

<pre>
<code>@Component
public class CustomSuccessHandler implements AuthenticationSuccessHandler{
  private RequestCache requestCache = new HttpSessionRequestCache(); // 로그인페이지 이동 전 요청 저장되어있음
  private RedirectStrategy redirectStrategy = new DefaultRedirectStrategy(); // 로그인 진행 후 리다이렉트전략

  @Override
  public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse, Authentication authentication){
    MemberLogin memberLogin = (MemberLogin)authentication.getPrincipal();
    // 만약 계정이 정지되었다면
    if(memberLogin.getMember().isLocked()){
      // 계정정지 예외 발생
      throw new LockedException("계정이 정지되었습니다. 관리자에게 문의하세요.");
    }


    SavedRequest savedRequest = requestCache.getRequest(request, redirect);
    // 직접 로그인페이지로 접속한게 아니라면,
    if(savedRequest != null){
      // 이전 요청으로 리다이렉트
      redirectStrategy.sendRedirect(request, response, savedRequest.getRequestUrl());
    }else{
      // 직접 로그인 페이지로 접근한거라면,
      redirectStrategy.sendRedirect(request, response,
        "/"); // 메인페이지로 리다이렉트
    }

  }
}
</code>
</pre>

이 기능을 이용하여 관리자가 런타임중 특정 유저의 locked 필드만 true로 변경하는 것으로 계정을 정지시키고, 정지해제 시킬수 있도록 구현했다.  
실제로 쓰려면 좀 더 손을 봐야겠지만, 그래도 안해두는것보단 훨 나을듯싶다.  
더 낫게 쓰는방법에 대해선 추후 알게되면 다시 정리해야겠다.
