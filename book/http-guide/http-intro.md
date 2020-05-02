---
title: HTTP 웹의 기초
tags:
  - http
categories: HTTP
layout: post
---

# 01. HTTP 개관

## 리소스

### 미디어 타입

MIME 타입\(Multipurpose Internet Mail Extensions\) : 데이터 포맷 라벨, 모든 HTTP 객체 데이터에 MIME 타입을 붙인다. MIME 타입을 통하여 객체 확인. 주 타입 / 부 타입  
ex\) text/html, text/plain, image/jpeg, application/vnd.ms-powerpoint 등

### **URI**

통합 자원 식별자\(Uniform Resource Identifier\), 서버의 리소스 이름, 리소스의 주소라고 보면 됨

### **URL**

통합 자원 지시자\(Uniform Resorce Locator\), 한 리소스에 대한 구체적인 위치. 오늘날 대부분의 URI는 URL

![url](https://user-images.githubusercontent.com/35331310/56505874-c731ca80-6557-11e9-839b-eed86d7ac668.png)

* 첫 번째 부분 : Scheme. 리소스에 접근하기 위해 사용되는 프로토콜. 보통 HTTP 프로토콜 \(https://\)
* 두 번째 부분 : 서버의 인터넷 주소를 제공 \(github.com\)
* 세 번째 부분 : 웹 서버의 리소스 \(/haky2/let-us-spring/wiki/HTTP-웹의-기초/\_edit\)

### **URN**

통합 자원 이름\(Uniform Resource Name\), 리소스의 위치에 영향 받지 않는 유일한 이름

![urn](https://user-images.githubusercontent.com/35331310/56505931-f5170f00-6557-11e9-8e2c-d400ddcd7bc0.png)

## 트랜잭션

### 요청 메시지

명령과 URI를 포함. 클라이언트에서 서버로

### 응답 메시지

결과와 상태코드 포함. 서버가 클라이언트로

### **메서드**

모든 HTTP 요청 메시지는 한 개의 메서드를 갖는다. GET, PUT, DELETE, POST, HEAD ....

### **상태코드**

모든 HTTP 응답 메시지는 상태 코드와 함꼐 반환된다. 200, 404, 500 ....

## 메시지

![http message](https://user-images.githubusercontent.com/35331310/56506417-6acfaa80-6559-11e9-99b1-ac6105a41ea5.png)

* 시작줄 : 명령, 상태코드 등을 나타냄
* 헤더 : MIME 타입, 응답 본문 길이 등을 나타냄
* 본문 : 데이터, 이진 데이터\(이미지, 비디오, 오디오 등\) 포함 가능

## TCP

* 전송 제어 프로토콜\(Transmission Control Protocol\)
* TCP : 오류 없는 데이터 전송, 순서에 맞는 전달, 조각나지 않는 데이터 스트림
* TCP/IP : 패킷 교환 네트워크 프로토콜의 집합. 어떤 종류의 컴퓨터나 네트워크든 서로 신뢰성 있는 의사소통이 가능하도록 해주는 역할
* HTTP 클라이언트가 서버에 메시지를 전송할 수 있게하기 위해, IP와 포트번호를 사용해 클라이언트와 서버 사이에 TCP/IP 커넥션을 맺어야 함
* 호스트명 : 도메인 이름, IP 주소에 대한 별명, DNS \(Domain Name Service\)를 통해 IP로 변환
* URL의 기본 포트번호는 80

  ![tcp/ip](https://user-images.githubusercontent.com/35331310/56507507-7a9cbe00-655c-11e9-92a9-75f31a3c768d.png)

### 프로토콜 버전

많은 서비스들은 HTTP/1.1만 지원하고 있지만, 현재 [HTTP/2.0](https://www.popit.kr/나만-모르고-있던-http2/) 프로토콜도 나와있는 상태이다.

## 웹의 구성요소

### 프락시

클라이언트와 서버 사이에 위치한 HTTP 중개자. 주로 보안을 위해 사용되며, 요청과 응답을 필터링 함

![proxy](https://user-images.githubusercontent.com/35331310/56507260-b4b99000-655b-11e9-9646-f7997019cadf.png)

### **캐시**

많이 찾는 웹페이지를 클라이언트 가까이에 보관하는 HTTP 창고

![cache](https://user-images.githubusercontent.com/35331310/56507329-e6caf200-655b-11e9-9e6d-d7ca51f4a0cf.png)\*\*\*\*

### **게이트웨이**

다른 애플리케이션과 연결된 특별한 웹 서버. 주로 HTTP 트래픽을 다른 프로토콜로 변환하기 위해 사용. HTTP/FTP 등

![gateway](https://user-images.githubusercontent.com/35331310/56507371-08c47480-655c-11e9-9d24-2c72edf6fce2.png)

### **터널**

HTTP 통신을 전달하기만 하는 특별한 프락시. 비 HTTP 데이터를 하나 이상의 HTTP 연결을 통해 그대로 전송해주기 위해 사용. SSL 등

![tunnel](https://user-images.githubusercontent.com/35331310/56507263-b5eabd00-655b-11e9-9d3f-bcf4d8de5371.png)

### **에이전트**

자동화된 HTTP 요청을 만드는 웹클라이언트. 웹브라우져, 웹로봇 등

