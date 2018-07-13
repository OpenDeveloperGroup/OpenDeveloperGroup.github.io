---
layout: page
title: Interceptor Chain Pattern And Event Model
author: "Daeho Han"
permalink: /Archives
tags: pattern
---

# Interceptor Chain Pattern And Event Model

참고 : http://wildpup.cafe24.com/archives/596

## 1. Intercepting Filter Pattern
Intercepting Filter Pattern은 J2EE기반 기술에서 적용된 Pattern으로 요청이나 응답의 전/후로 어떤 작업을 수행 하고자 할 때 사용할 수 있다.

 Interception Filter Pattern은 다음과 같은 요소로 이루어져 있다. 

 > - Filter : 요청이나 응답 전/후에 처리해야할 작업
 > - Filter Chain : 여러개의 Filter를 가지고 연속적으로 처리하게 할 수 있으며 Target 설정 및 Filter를 추가
 > - Target : 실제 요청 대상
 > - Filter Manager : Filter들과 FilterChain을 관리 한다.