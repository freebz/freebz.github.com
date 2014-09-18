---
layout: post
title: "Spring Security CSRF 방어"
description: "Spring Security CSRF 사용하기"
category: Spring Security
tags: [Spring Security, CSRF]
author: FreeBz
---

스프링 3.2에서 CSRF 방어를 지원한다.

http://spring.io/blog/2013/08/21/spring-security-3-2-0-rc1-highlights-csrf-protection/

비교적 간단한 설정으로 CSRF 를 방어할 수 있어, 상당히 편리하다.

Spring Security CSRF 를 사용하면서 사소한 문제가 조금 있었다.

첫번째는 한글 깨짐이었다.

Encoding Filter를 이용하여 UTF-8로 처리하였는데,

Encoding Filter를 Spring Security 뒤에 설정하면,
CSRF 필터가 적용될 경우, 한글이 깨지는 문제가 발생하였다.

보편적으로 인증에 실패하면 굳히 인코딩을 한 필요가 없다고 판단하여,
Spring Security 필터를 우선으로 적용하였는데, 

CSRF 필터를 사용하고자 한다면, 인코딩 필터를 우선 적용하면,
한글 깨짐을 방지할 수 있다.


두번째는 멀티파트 파일전송이었다.
CSRF 필터가 멀티파트로 전송된 요청에서 CSRF 토큰을 가져오지 못하여
항상 요청이 실패하였다.

쉬운 방법으로는 CSRF 토큰을 URL에 포함시켜 전송하면 cSRF 필터가 값을 가져와서 정상적으로 처리가 된다.

좀 더 세련된 방법으로는 MultipartResolver Filter를 적용시키는 방법이다.

	<filter>
    	<filter-name>multipartFilter</filter-name>
    	<filter-class>org.springframework.web.multipart.support.MultipartFilter
    	</filter-class>
    	<init-param>
    		<param-name>multipartResolverBeanName</param-name>
    		<param-value>multipartResolver</param-value>
    	</init-param>
	</filter>

이렇게 필터를 추가하면 CSRF 필터에서 요청이 MultipartResolver에서 처리되어 파라메터를 읽어와서 정상처리가 된다.
init-param의 값은 스프링에서 설정된 multipartResolver 빈의 ID 이다.

하지만, 실제 컨트롤러에서 파일을 처리할 수 없는 문제가 발생하였다.
필터쪽에서 스트림을 이미 사용하였기 때문에, 컨트롤러에서 스트림을 사용할 수가 없었다.

보통 스프링 멀티파트 리졸버의 id는 multipartResolver로 적용해야 한다. multipartResolver라는 ID로 MultipartResolver를 찾기 때문이다.

그러나 Multipart Filter를 적용할 때는 filterMultipartResolver라는 ID를 적용해야 한다. 그러면 필터에서 처리된 Multipart요청이 Controller까지 전달 된다.

필터 설정은 다음과 같이 하고,

	<filter>
		<filter-name>multipartFilter</filter-name>
		<filter-class>org.springframework.web.multipart.support.MultipartFilter</filter-class>
	</filter>
    
Multipart Resolver의 ID를 filterMultipartResolver로 변경하면,
CSRF의 요청과 Controller의 요청을 모두 잘 처리할 수 있다.
