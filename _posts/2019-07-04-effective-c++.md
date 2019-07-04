---
title: "Effective C++ 정리"
date: 2019-07-04 17:00 +09:00
categories:
  - C++
tags:
  - Effective C++
toc: true
---

### 개요

Effective C++ 제 3판 번역본을 보고 공부하면서 잘 까먹을 수 있을 부분을 짧게 정리해 놓았음  

### Chapter 1: C++에 왔으면 C++의 법을 따릅시다.

1. C++은 다중패러다임 프로그래밍 언어(multiparadigm programming language)라고 불린다. 

   1. C언어
   2. 객체 지향 개념의 C++
   3. 템플릿 C++
   4. STL

2. #define보다는 const, enum, inline이 더 좋다.

3. const 상수

   1. 상수성

      1. 비트수준 상수성(bitwise constness): 객체를 구성하는 비트들 중 어떤 것도 바꾸면 안된다.
      2. 논리적 상수성(logical constness): 일부 몇 비트를 바꿀 수는 있지만 그것을 사용자측에서 알아채지 못하면 된다.

   2. 비상수 함수와 상수함수의 코드 중복 현상 피하기

      1. 비상수 멤버함수에서 상수 멤버함수를 호출하고 그 결과에서  const_cast를 이용하여 const를 떼어냄으로써 중복을 피할 수 있다.

      ```c++
      template <typename T>
      class block {
          const T& operator[](size_t pos) const {
              ...
              return text[pos];
          }
          T& operator[](size_t pos) {
              return const_cast<const block&> (
              	static_cast<const block&>(*this)[pos]
              );
          }   
      };
      ```

4. 객체를 사용하기 전에는 반드시 그 객체를 초기화 해야한다.

   1. 비지역 정적 객체의 초기화 순서는 개별 번역 단위에서 정해진다.
      1. 용어 설명
         1. 비지역 정적 객체: 함수안에 없는 static 객체
         2. 번역단위: 컴파일을 통해 하나의 목적파일이 되는 소스코드 (c++파일)
      2. c++파일이 두개로 나누어진 라이브러리가 존재하고, 한 class에서 다른 class의 비지역 정적 객체를 사용한다면, 사용되는 쪽이 먼저 초기화가 되어 있어야 하지만, c++에서는 이 다른 번역단위에서의 초기화 순서를 모르기에 먼저 초기화가 되는것을 보장할 수가 없다.
      3. 따라서 비지역 정적 객체를 지역 정적 객체로 바꾸어서 해결해야 한다.

### Chapter 2: 생성자, 소멸자 및 대입 연산자

공부중

### Chapter 3: 자원 관리

공부중

### Chapter 4: 설계 및 선언

공부중

### Chapter 5: 구현

공부중

### Chapter 6: 상속, 그리고 객체 지향 설계

공부중

### Chapter 7: 템플릿과 일반화 프로그래밍

공부중

### Chapter 8: new와 delete를 내 맘대로

공부중

### Chapter 9: 그 밖의 이야기들

공부중

