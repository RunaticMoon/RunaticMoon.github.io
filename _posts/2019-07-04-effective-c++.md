---
title: "Effective C++ 정리"
date: 2019-07-04 17:00 +09:00
categories:
  - C++
tags:
  - Effective C++
  - 정리
toc: true
---

### 개요

Effective C++ 제 3판 번역본을 보고 공부하면서 잘 까먹을 수 있을 부분을 짧게 정리해 놓았음  
언제나 책을 보는것이 더 정확하고 이해하기에 좋습니다.

### Chapter 1: C++에 왔으면 C++의 법을 따릅시다.

1.  C++은 **다중패러다임 프로그래밍 언어**(multiparadigm programming language)라고 불린다. 
   1.  C언어
   2.  객체 지향 개념의 C++
   3.  템플릿 C++
   4.  STL

2. #define보다는 const, enum, inline이 더 좋다.

3. const 상수
   1. 상수성

      1.  **비트수준 상수성**(bitwise constness): 객체를 구성하는 비트들 중 어떤 것도 바꾸면 안된다.
      2.  **논리적 상수성**(logical constness): 일부 몇 비트를 바꿀 수는 있지만 그것을 사용자측에서 알아채지 못하면 된다.

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
   1.  **비지역 정적 객체**의 초기화 순서는 **개별 번역 단위**에서 정해진다.
      1.  용어 설명
         1.  비지역 정적 객체: 함수안에 없는 static 객체
         2.  번역단위: 컴파일을 통해 하나의 목적파일이 되는 소스코드 (c++파일)
      2.  C++파일이 두개로 나누어진 라이브러리가 존재하고, 한 class에서 다른 class의 비지역 정적 객체를 사용한다면, 사용되는 쪽이 먼저 초기화가 되어 있어야 하지만, C++에서는 이 다른 번역단위에서의 초기화 순서를 모르기에 먼저 초기화가 되는것을 보장할 수가 없다.
      3.  따라서 비지역 정적 객체를 지역 정적 객체로 바꾸어서 해결해야 한다.

### Chapter 2: 생성자, 소멸자 및 대입 연산자

5.  컴파일러가 **암시적**으로 만드는 함수
    1.  기본 생성자
    2.  복사 생성자
    3.  소멸자
    4.  복사 대입 연산자

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
14. RAII 객체의 복사 동작
    1. 복사를 금지 (Uncopyable 등을 사용)
    2. 관리하고 있는 자원에 대해 참조 카운팅을 수행 (shared_ptr)
    3. 관리하고 있는 지원을 깊은 복사 (포인터가 가르키는 메모리까지 복사)
    4. 관리하고 있는 자원의 소유권을 옮김 (auto_ptr)
15. 기존의 API에는 실제 자원을 직접 접근하는 경우가 많기에 자원흭득이 가능해야 한다.
    1. 명시적 변환 (.get() 등을 사용 / 안전성)
    2. 암시적 변환 (operator REAL_RESOURCE[^3] 를 사용 / 편의성)
16. 동적할당(new, delete)를 쓸때는 서로의 []의 형태를 맞추자
17. new로 생성한 객체를 스마트 포인터로 넣는 코드는 별도의 한 문장으로 만들자
    컴파일러의 설계에 따라 한 줄로 만들시 자원누출의 우려가 있다.

### Chapter 4: 설계 및 선언

18. 제대로 쓰긴 쉽고, 엉터리로 쓰기엔 어려운 인터페이스를 설계하자

19. 클래스 설계는 타입 설계와 같다.
    1. 생성 및 소멸
    2. 초기화와 대입의 차이점은?
    3. call by value의 의미는? (복사 생성자)
    4. 값에 대한 제약은? (클래스의 불변속성)
    5. 클래스 상속 계통망은?
    6. 타입 변환을 허용할 것인가? (암시적, 명시적)
    7. 어떤 연산자와 함수를 두고 무엇을 private로 할것인가
    
20. call by value보다는 call by const reference가 좋다.
    복사손실 문제(slicing problem): 파생클래스를 기본클래스로 값에 의한 전달시
                                                           파생클래스의 부분이 잘려 나간다.
    
21. 참조자를 반환하지 말자

22. 데이터 멤버의 선언은 private에 하자

23. 멤버함수보다는 비멤버 비프렌드 함수가 좋다
    비멤버 비프렌드 함수와 클래스를 같은 namespace에 넣어서 패키징하면 좋다.

24. 타입변환이 모든 매개변수에 적용되야 한다면 비멤버 함수로 선언하자

    1. 멤버함수의 반대는 비멤버 함수이다.
    2. 타입변환순서 (A * B)
       1. 멤버함수 연산자 오버로딩 (B)
       2. 비멤버 연산자 오버로딩 (A * B)
       3. 암시적 변환 - 비멤버 연산자 오버로딩 (A * A(B))

25. 예외를 던지지 않는 swap에 대해 생각해보자

    1. 표준 swap함수

       ```c++
       namespace std {
         template<typename T>
         void swap(T& a, T& b) {
           T temp(a);
           a = b;
           b = temp;
         }
       }
       ```

    2. 완전 템플릿 특수화(total template specialization)

       ```c++
       namespace std {
         template<>
         void swap<Widget>(Widget& a, Widget& b) {
           // pImpl이 private이기에 컴파일이 안된다.
       		// swap(a.pImpl, b.pImpl);
           
           // Widget안에 using std::swap을 하고서 swap을 만든다.
           a.swap(b);
         }
       }
       ```

    3. C++의 이름탐색 규칙(인자 기반 탐색(argument-dependent lookup / ADL) 혹은 쾨니그 탐색(Koenig lookup))

       1. 인자가 위치한 네임스페이스 내부의 이름부터 탐색

    4. swap을 호출할 때 using std::swap을 포함시키고, swap호출에 한정자(std::)를 붙이지 않는다.

### Chapter 5: 구현

26. 변수의 정의는 가능한 늦추자
    1. 변수가 사용되기 직전에 복사생성자로 정의하자
    2. 반복문에서의 변수 정의
       1. 루프 밖: 비용(대입 > 생성자-소멸자)
       2. 루프 안: 비용(대입 < 생성자-소멸자), 수행성능에 민감한 부분
27. 캐스팅은 절약하자
    1. 구형 스타일의 캐스트
       1. C스타일 캐스트: `(T)표현식`
       2. 함수방식 캐스트: `T(표현식)`
    2. 신형 스타일의 캐스트
       1. 상수성제거 캐스트: `const_cast<T>(표현식)`
       2. 안전한 다운캐스팅: `dynamic_cast<T>(표현식)`
       3. 하부수준 캐스트(이식성X): `reinterpret_cast<T>(표현식)`
       4. 암시적 변환 강제: `static_cast<T>(표현식)`
    3. 캐스팅이 일어날때 사본이 만들어져 원하지 않는 결과가 생길 수 있다.
    4. 캐스팅이 필요하다면 함수안에 숨겨봐라
28. 내부 객체에 대한 `핸들`을 반환하는건 피하자
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
[^3]: 여기서 REAL_RESOURCE는 각 RAII객체가 관리하는 실제 자원의 이름이다.

