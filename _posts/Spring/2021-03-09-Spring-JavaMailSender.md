---
layout: post
title:  "[Spring-Boot] JavaMailSender에 대하여"
subtitle:   "Application에서 Mail을 보내기위한 API"
date: 2021-03-08
categories: Spring
tags: Spring-Boot
comments: true
---

# JavaMailSender에 대하여
---
>스프링에서 제공하는 웹 어플리케이션에서 메일을 보다 손쉽게 보낼 수 있도록 해주는 API

웹 어플리케이션에서 회원의 이메일이 실제로 존재하는 이메일인지, 회원의 이메일인지를 파악하기 위해 이메일 인증을 하게된다.  
그러면 당연히 웹 어플리케이션에서 인증을 할 수 있는 메일을 보내야 하는데, 이때 보다 편리하게 이메일을 작성해서 보낼 수 있도록 도와주는 API이다.


> MailSender 인터페이스를 상속받아 본문에 HTML 메일을 작성할 수 있도록 기능 추가된 인터페이스

<strong>MailSender</strong>는 SimpleMailMessage를 정의해 텍스트 메일을 발송 할 수있는 반면,  
<strong>JavaMailSender</strong>에선 MimeMessage를 정의해 본문이 HTML로 이루어진 메일을 발송할 수 있다.

> HTML작성은 어떻게하지?

타임리프를 사용중이라면 타임리프에서 제공하는 TemplateEngine 클래스의 process()메소드를 이용하면 된다.

이때, 타임리프에서 사용하는 파라미터값을 제공하기위해 스프링의 Model과 비슷한 역할을 하는 Context 객체에 뷰페이지에서 사용하는 파라미터를 정의하여 타임리프로만든 HTML파일과 Context객체를 인자로 주면 알아서 HTML로 변환해주며, 이 변환된 문자열을 <strong>MimeMessage</strong>의 setText() 메소드 인자로 할당하면 HTML메일이 보내진다는 것이다.

MimeMessage에는 Multipart기능도 제공하기 때문에, 메일에 특정 첨부파일을 같이 보낼수도 있다.

<strong>LegacySpringFramework</strong> 사용중이라면 Spring 빈 설정파일에 빈 설정 후 의존 주입받아서 사용하면되고,  
<strong>SpringBoot</strong> 는 바로 의존 주입받아서 사용하면된다.



예시코드  
#### SMTP정보
<pre>
  <code>
spring.mail.host=smtp.gmail.com
spring.mail.port=587
# 본인 gmail 계정을 입력하세요.
spring.mail.username= ${발신자 계정 메일 주소}
# 본인 앱 패스워드로 변경하세요.
spring.mail.password= ${발신자 계정 비밀번호}
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.timeout=5000
spring.mail.properties.mail.smtp.starttls.enable=true

  </code>
</pre>


#### 타임리프
ex-email.html
~~~
<pre>
  <code>
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <div>
        <p>안녕하세요. <span th:text="${nickname}"></span>님</p>

        <h2 th:text="${message}">메시지</h2>

        <div>
            <a th:href="${host + link}" th:text="${linkName}">Link</a>
        </div>
    </div>
</body>
</html>
</code>
</pre>
~~~

#### Mail Sender Service
<pre>
  <code>
@Service
@RequiredArgsConstructor
public String mailSender() throws MessagingException{
  private final JavaMailSender javaMailSender;
  private final TemplateEngine templateEngine;

  public void sendMail(){
    Context context = new Context();
    context.setVariable("nickname", "규스");
    context.setVariable("message", "회원 가입을 진행하시려면 아래 링크를 클릭해 이메일 인증을 시도해주세요.");
    context.setVariable("host", "http://localhost:8080/"); // 메시지소스로 설정해두고 받아쓰면 참 편하다.
    context.setVariable("link", "email-authentication"); // 인증을 진행할 링크
    context.setVariable("linkName", "여기를 클릭해주세요"); // 위 링크를 덧씌울 텍스트

    String message = templateEngine.process("ex-email.html", context);

    MimeMessage mail = javaMailSender.createMimeMessage();
    MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mail, false, "UTF-8"); // 2번째 인자는 Multipart여부 결정
    mimeMessageHelper.setTo(<받는사람 이메일 주소>);
    mimeMessageHelper.setSubject(<이메일 제목>);
    mimeMessageHelper.setText(message, true); // 2번째 인자는 HTML여부 결정
    javaMailSender.send(mimeMessage);
  }
}
  </code>
</pre>

>값 세팅때문에 코드 가독성이 떨어지므로 되도록 별도의 유틸성 메소드를 만들어 사용하거나,  
별도의 클래스를 생성해 관리하는것을 추천한다.

이번에 공부하면서 타임리프와 스프링의 상호작용이 상당히 좋다는걸 알게되었다. 역시 밀어주는데는 이유가 있는것 같다.  
한번 배운거 까먹지말고 잘 기억해둬야겠다.

>코드나, 설명에 잘못된 점이 있거나, 추가해주실 사항이 있다면 댓글로 남겨주세요.
