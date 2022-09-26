---
layout: single
title: "Spring_Security 환경설정"
categories: SpringSecurity
toc: true
author_profile: true
sidebar:
    nav: "docs"
---

# Spring_Security 
- SpringWeb
- SpringSecurity
- Spring Data JPA
- Spring Boot DevTools
- MySQL Driver
- Mustache
- Lombok

<br>

# MySql DB 및 사용자 생성
````java
create user 'cos'@'%' identified by 'cos1234';
GRANT ALL PRIVILEGES ON *.* TO 'cos'@'%';
create database security;
use security;
````

<br>

# 컨트롤러 생성
````java
package security.security12.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller // view를 리턴
public class  indexController {

    @GetMapping({"","/"})
    public String index(){
        return "index.html";
    }
}
````

<br>

# 인덱스 파일 생성
````java
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>인덱스 페이지</title>
</head>
<body>
<h1>인덱스 페이지입니다</h1>
</body>
</html>
````

<br>

# Config 생성
````java
package security.config;

import org.springframework.boot.web.servlet.view.MustacheViewResolver;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewResolverRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        MustacheViewResolver resolver = new MustacheViewResolver();
        resolver.setCharset("UTF-8");
        resolver.setContentType("text/html; charset=UTF-8");
        resolver.setPrefix("classpath:/templates/");
        resolver.setSuffix(".html");

        registry.viewResolver(resolver);
    }
}
````