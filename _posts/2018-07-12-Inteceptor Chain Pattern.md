---
layout: page
title: Interceptor Chain Pattern And Event Model
author: "Daeho Han"
permalink: /Archives
tags: pattern
---

> - 참고 : http://wildpup.cafe24.com/archives/596

## 1. Intercepting Filter Pattern
Intercepting Filter Pattern은 J2EE기반 기술에서 적용된 Pattern으로 요청이나 응답의 전/후로 어떤 작업을 수행 하고자 할 때 사용할 수 있다.

 Interception Filter Pattern은 다음과 같은 요소로 이루어져 있다. 

 > - Filter : 요청이나 응답 전/후에 처리해야할 작업
 > - Filter Chain : 여러개의 Filter를 가지고 연속적으로 처리하게 할 수 있으며 Target 설정 및 Filter를 추가
 > - Target : 실제 요청 대상
 > - Filter Manager : Filter들과 FilterChain을 관리 한다.

![Intercepting Filter Pattern Diagram](https://opendevelopergroup.github.io/assets/daeho/netty/intercepting_filter_pattern_diagram.png)

 code는 다음과 같다
 
 ```java
 public interface Filter {
     public void execute(String request);
 }
 ```

 ```java
 public class AuthenticationFilter Implements Filter {
     @Override
     public void execute(String request) {
         // TODO Authenticate
     }
 }
 public class LoggingFilter Implements Filter {
     @Override
     public void execute(String request) {
         // TODO Logging
     }
 }
 ```

 ```java
 public class Target {
     public void execute(String request) {
         System.out.println("Executing request" + request);
     }
 }
 ```

 ```java
 public class FilterChain {
     private List<Filter> filters = new ArraList<Filter>();
     private Target target;
     
     public void addFilter(Filter filter) {
         filters.add(filter);
     }
     public void execute(String request) {
         for(Filter filter : filters) {
             filter.execute(request);
         }
         target.execute(request);
     }
     public void setTarget(Target target) {
         this.target = target;
     }
 }
 ```

 ```java
 public class FilterManager {
     FilterChain filterChain;

     public FilterManager(Target target) {
         filterChain = new FilterChain();
         filterChain.setTarget(target);
     }

     public void setFilter(Filter filter) {
         filterChain.addFilter(filter);
     }

     public void filterRequest(String request) {
         filterChain.execute(request);
     }
 }
 ```

 ```java
 public class Client {
     FilterManager filterManager;

     public void setFilterManager(FilterManager filterManager) {
         this.filterManager = filterManager;
     }

     public void sendRequest(String request) {
         filterManager.filterRequest(request);
     }
 }
 ```

 ```java
 public class InterceptingFilter {
     public static void main(String[] args) {
         FilterManager filterManager = new FilterManager(new Target());
         filterManager.setFilter(new AuthenticationFilter());
         filterManager.setFilter(new DebugFilter());

         Client client = new Client();
         client.setFilterManager(filterManager);
         clinet.sendRequest("INTECEPTING FILTER EXAM!");
     }
 }
 ```
