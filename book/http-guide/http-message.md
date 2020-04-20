---
title: HTTP 메시지
tags:
  - http
  - http-message
categories: HTTP
layout: post
---

# 03. HTTP 메시지

## 메시지의 흐름

* 메시지는 원 서버 방향을 인바운드로 하여 송신된다
* 모든 처리가 끝난 뒤에 메시지가 사용자 에이전트로 돌아오는 것은 아웃바운드로 이동하는 것이다.
* 모든 메시지는 다운스트림으로 흐른다

## 메시지 구조

### 요청 메시지

![&#xC694;&#xCCAD; &#xBA54;&#xC2DC;&#xC9C0;](https://user-images.githubusercontent.com/35331310/58430619-0cac5f00-80e5-11e9-8870-fcf3e7a7fe37.png)

* 메서드 : 클라이언트 측에서 서버거 리소스에 대해 수행해주길 바라는 동작
* 요청URL : 요청 대상이 되는 리소스를 지칭하는 url
* 버전 : 메시지에서 사용 중인 HTTP의 버전

### 응답 메시지

![&#xC751;&#xB2F5; &#xBA54;&#xC2DC;&#xC9C0;](https://user-images.githubusercontent.com/35331310/58430650-2ea5e180-80e5-11e9-8223-05814a8d3806.png)

* 상태코드 : 요청 중에 무엇이 일어났는지 설명하는 숫자
* 사유구절 : 상태 코드의 의미를 사람이 이해할 수 있게 설명해주는 짧은 문구

### HTTP 메소드

![HTTP &#xBA54;&#xC18C;&#xB4DC;](https://user-images.githubusercontent.com/35331310/58430911-6b260d00-80e6-11e9-88a6-1fa42d7a30f8.png)

### HTTP 상태코드

![HTTP &#xC0C1;&#xD0DC;&#xCF54;&#xB4DC;](https://user-images.githubusercontent.com/35331310/58430974-c0621e80-80e6-11e9-9be4-e4eec65fc5b3.png)

참고 사이트

* [HTTP 메시지](https://m.blog.naver.com/PostView.nhn?blogId=yswon72&logNo=221210025791&proxyReferer=https%3A%2F%2Fwww.google.com.au%2F)
* [HTTP 메소드](https://javaplant.tistory.com/18)
* [HTTP 상태코드](https://blog.naver.com/yswon72/221209525370)
* [HTTP 헤더](https://www.slideshare.net/choong83/http-3-http)

