---
layout: post
title:  "[Spring-Security] 계정 정지 기능 구현 - 2"
subtitle:   "특정 사용자 계정 정지 및 실시간 계정 로그아웃 기능 구현"
date: 2021-03-18
categories: Spring
tags: Spring-Security
comments: true
---

# 계정잠금기능 설정하기 - 2
---
>이번에는 계정 정지를 넘어 실시간으로 유저를 로그아웃 시키는 방법을 알아보자

1편에서 이어지는 내용이므로, 1편의 내용은 아래링크를 참조하세요.  
[클릭하시면 1편으로 이동합니다.](https://dev-gyus.github.io/spring/2021/03/16/SpringSecurity-AccountLock-2.html)  
***

사용된 개념은 다음과 같다
>1. SpringSecurity에서 사용자의 요청을 처리할때 세션에 인증정보가 있으면 해당 인증정보를 Security Context에 저장함
>2. 인증정보가 저장되어있는 객체는 SessionRegistry이며, getAllPrincipals() 메소드로 현재 인증된 모든 사용자 세션을 가져올수있음.
>3.반환된 객체를 로그인에 사용한 UserDetails 혹은 User 객체를 상속한 객체로 캐스팅 후 루프를 돌리며 특정 유저의 Unique필드와 캐스팅한 객체의 같은 필드를 대조한다.  
>4. 대조하다 정지시킬 유저의 Unique필드와 같을때 SessionRegistry의 getAllSessions()로 해당 유저의 객체로 생성된 모든 세션을 가져옴.
>5. 반환받은 객체는 SessionInfomation이란 객체의 컬렉션인데, 이 역시 루프를 돌려서 정지시킬 계정의 특정 unique값과 대조하여 해당하는 세션을 만료시켜 로그아웃 시키는 기능을 이용함.

우선, SessionRegistry를 우리가 주입받아 사용할 수 있도록 Security설정파일에 SessionRegistry 빈을 등록한뒤, SessionManagement에 해당 빈을 등록시키고, 세션이 만료되었을경우 스프링 시큐리티가 이를 감지하도록 이벤트 리스너도
빈으로 등록해둔다.  


<pre>
<code>
```
@Configuration  
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
  ... 생략
  @Override
  protected void configure(HttpSecurity http) throws Exception{
    ... 생략
    http.sessionManagement.sessionRegistry(sessionRegistry());
  }
  // SessionRegistry 빈등록 및 SessionManagement SessionManagementFilter에 등록
  @Bean
  public SessionRegistry sessionRegistry(){
    return new SessionRegistryImpl();
  }

  // 세션 만료시 SpringSecurity가 이를 감지하도록 이벤트 리스너 설정
  @Bean
  public static ServletListenerRegistrationBean<HttpSessionEventPublisher> httpSessionEventPublisher(){
    return new ServletListenerRegistrationBean<HttpSessionEventPublisher>(new HttpSessionEventPublisher());
  }
}
```
</code>
</pre>

이제 준비는 다 됐고, 유저를 정지시키는 클래스에 정지기능을 가진 메소드를 구현하면된다.  
Spring Security, Spring Data JPA를 사용하였다


<pre>
<code>
```
@Service
@RequireArgsConstructor
@Transactional
public class BlockUserService{
  private final MemberRepository memberRepository;
  private final SessionRegistry sessionRegistry;

public void blockUser(Long memberId){
  Member findMember = memberRepository.findById(memberId).orElseThrow();
  findMember.setLocked(true); // 유저 계정 잠그기
  // 모든 세션에서 인증된 객체를 가져와서 UserDetails 혹은 User객체 상속한 로그인용 DTO객체로 변환
  List<MemberLogin> principals = (MemberLogin) sessionRegistry.getAllPrincipals()
                    .stream().map(o -> (MemberLogin) o).collect(Collectors.toList());

  for(MemberLogin principal : principals){
    Member member = principal.getMember();
    // 인증된 객체들중 정지시키려는 객체의 유니크값과 루프돌던 인증객체의 유니크값이 같을경우
    if(member.getId() == memberId){
      List<SessionInformation> sessionList = sessionRegistry.getAllSessions(principal, false); // 해당 인증객체로 생성된 모든 세션을 가져옴
      for(SessionInformation session : sessionList){
        session.expireNow(); // 해당 인증객체의 현재 만료되지 않은 세션을 모두 만료시킴 -> HttpSessionEventPublisher가 세션 만료를 감지하고 로그아웃 처리시킴
      }
    }
  }
}
}
```
</code>
</pre>


이렇게 해두고, 정지관련된 리소스에는 관리자권한을 가진 유저만 접근할 수 있도록 설정한뒤, 정지 요청시 대응하는 컨트롤러에서
blockUser 메소드를 호출하도록 프로그래밍 해두면, 정지된 유저는 다음요청에 로그아웃 처리되어 인증을 필요로 하는 리소스에 접근할때 Security설정에 의해 Login Url로 이동하지만, 이미 정지처리가 되어있기 때문에 로그인해도 LockedException 발생으로 로그인이 되지않는다.  
***
이와같이 실시간으로 유저를 밴 할수있는 기능을 구현해봤는데, 이게 가장 최선의 예제라고는 생각하지 않지만, 적어도 서비스의 필수기능으로서의 역할은 잘 해주는것같다.  
결론은 까먹지말자.
