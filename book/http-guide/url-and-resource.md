---
title: URL과 리소스
tags:
  - http
  - url
  - resource
categories: HTTP
layout: post
---

# 02. URL과 리소스

## URL

* URL은 브라우저가 정보를 찾는데 필요한 리소스의 위치를 가리킴
* 리소스가 옮겨지면 URL을 더는 사용할 수 없음

## URL의 구조

`스킴://서버위치/경로`

* [http://www.haky.com](http://www.haky.com)
* mailto:haky2@haky.com스
* ftp://ftp.haky.com/data/http-study.xls
* rtsp://www.haky.com:554/dialog/today

## URL 문법

`<스킴>://<사용자이름>:<비밀번호>@<호스트>:<포트>/<경로>;<파라미터>?<질의>#<프래그먼트>`

* URL의 가장 중요한 세 가지 컴포넌트는 스킴, 호스트, 경로 이다
* 이 모든 컴포넌트를 가지는 URL은 거의 없음

