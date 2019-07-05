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

언제나 책을 보는것이 더 정확하고 이해하기에 좋습니다.  

### Chapter 1: C++에 왔으면 C++의 법을 따릅시다.

1.  C++은 **다중패러다임 프로그래밍 언어**(multiparadigm programming language)라고 불린다. 

   1. C언어
   2. 객체 지향 개념의 C++
   3. 템플릿 C++
   4. STL

2. #define보다는 const, enum, inline이 더 좋다.

3. const 상수

   1. 상수성

      1. **비트수준 상수성**(bitwise constness): 객체를 구성하는 비트들 중 어떤 것도 바꾸면 안된다.
      2. **논리적 상수성**(logical constness): 일부 몇 비트를 바꿀 수는 있지만 그것을 사용자측에서 알아채지 못하면 된다.

   2. 비상수 함수와 상수함수의 **코드 중복 현상 피하기**

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

4. 객체를 사용하기 전에는 반드시 그 객체를 **초기화** 해야한다.

   1. **비지역 정적 객체**의 초기화 순서는 **개별 번역 단위**에서 정해진다.
      1. 용어 설명
         1. 비지역 정적 객체: 함수안에 없는 static 객체
         2. 번역단위: 컴파일을 통해 하나의 목적파일이 되는 소스코드 (c++파일)
      2. c++파일이 두개로 나누어진 라이브러리가 존재하고, 한 class에서 다른 class의 비지역 정적 객체를 사용한다면, 사용되는 쪽이 먼저 초기화가 되어 있어야 하지만, c++에서는 이 다른 번역단위에서의 초기화 순서를 모르기에 먼저 초기화가 되는것을 보장할 수가 없다.
      3. 따라서 비지역 정적 객체를 지역 정적 객체로 바꾸어서 해결해야 한다.

### Chapter 2: 생성자, 소멸자 및 대입 연산자

5.  컴파일러가 **암시적**으로 만드는 함수
   1. 기본 생성자
   2. 복사 생성자
   3. 소멸자
   4. 복사 대입 연산자
   
6. 컴파일러에서 자동으로 제공하는 기능이 필요없어서 막을때

   1. private로 선언 (정의x)

      ```c++
      class block {
      private:
          block(const block&);
          block& operator=(const block&);
      };
      ```

   2. Uncopyable (boost 라이브러리의 noncopyable) 클래스 상속

      ```c++
      class Uncopyable() {
      protected:
          Uncopyable() { }	// 생성과 소멸 허용
          ~Uncopyable() { }
      private:
          Uncopyable(const Uncopyable&);		// 복사와 복사대입 방지
          Uncopyable& operator=(const Uncopyable&);
      };
      
      class block: private Uncopyable {
          ...
      };
      ```

   3. `C++11`  delete 사용

      ```c++
      class block {
      public:
          block(const block&) = delete;
          block& operator=(const block&) = delete;
      };
      ```

7. 다형성을 가진 __기본클래스의 소멸자__는 반드시 **가상 소멸자**로 선언하기

8. 소멸자에서 예외가 빠져나가면 안된다. 
   만약 예외의 가능성이 있다면 그 함수에서 `try~catch`로 해결하던지 프로그램을 끝내던지 해야한다.

9. 객체의 __생성과 소멸과정__ 중에는 절대로 가상함수를 호출하면 안된다.
   생성과 소멸과정에서는 가상함수가 파생클래스의 함수로 호출되는게 아니기 때문이다. 

10. 대입연산자의 반환값은 `*this`의 참조자를 반환하게 하자
    일반적으로 `x = y = z`, `cout << 1 << 2` 와 같은 연속적으로 사용 가능한 연산자에선 `*this`의 참조자를 반환하는게 일반적인 관례 및 사용에 좋다.

11. 대입연산자에서는 자기 자신에게 대입하는 경우를 반드시 처리해주도록 하자.

12. 객체의 복사에서 반드시 모든 데이터 멤버와 모든 기본 클래스 부분을 빠뜨리지 말자.
    복사대입과 복사 생성자의 코드가 중복되면 이 부분을 따로 떼어내 `private: init`으로 분리할 수는 있다.

    ```c++
    class childBlock : public block {
        // 복사 생성자
        childBlock(const childBlock& rhs)
            : block(rhs)
            , text(rhs.text)
        {
            ...
        }
        
        // 복사대입 연산자
        childBlock& operator=(const childBlock& rhs) {
            ...
            block::operator=(rhs);
            text = rhs.text;
            
            return *this;
        }
    }
    ```

### Chapter 3: 자원 관리

13. 자원 누출을 막기 위해 자원관리를 하는 객체를 사용하자
    1. RAII[^1]객체 사용 (생성자에서 자원을 흭득하고 소멸자에서 그것을 해제)
    2. RCSP[^2]를 사용 → e.g. `C++11` shared_ptr를 사용
14. 
15. 
16. 
17. 

### Chapter 4: 설계 및 선언

18. 
19.  
20.  
21.  
22.  
23.  
24.  
25.  

### Chapter 5: 구현

26. 
27.  
28.  
29.  
30.  
31.  

### Chapter 6: 상속, 그리고 객체 지향 설계

32. 
33.  
34.  
35.  
36.  
37.  
38.  
39.  
40.  

### Chapter 7: 템플릿과 일반화 프로그래밍

41. 
42.  
43.  
44.  
45.  
46.  
47.  
48.  

### Chapter 8: new와 delete를 내 맘대로

49. 
50.  
51.  
52.  

### Chapter 9: 그 밖의 이야기들

53. 
54.  
55.  



### 각주

[^1]: 자원 흭득 즉 초기화(Resource Acquisition Is Initialization)의 약자
[^2]: 참조 카운팅 방식 스마트 포인터(Reference-Counting Smart Pointer)의 약자

