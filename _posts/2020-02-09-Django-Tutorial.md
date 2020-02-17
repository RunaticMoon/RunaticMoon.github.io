---
title: "Django Tutorial 정리"
date: 2020-02-09 14:00 +09:00
categories:
  - Python
tags:
  - Django
  - 정리
toc: true

---

>  출처: [튜토리얼 사이트](https://gcamp.tistory.com/572)

# 1장

### client와 server

client -> server: 요청(request)
client <- server: 응답(response)



### HTTP 프로토콜의 특징

요청(request)에 대한 응답(response)을 한 후 연결(connection)을 해제한다.
연결에 대한 상태(state)를 유지하기 위해서 쿠키(cookie) 또는 세션(session)을 사용한다.



### URL 맵핑 (urls.py)

쿼리스트링 방식이 아닌 url에 파라미터를 넣어 url에 따라 Django의 view에서 서로 다른 함수를 실행하게 한다.



### 서버 구성

client -> `웹 서버` -> `애플리케이션 서버` -> DB

1. 웹 서버: 정적인 데이터 요청 처리
2. 애플리케이션 서버: 동적인 데이터 요청 처리



# 2장

### Django 설치

```python
# 설치
pip install django

# 업그레이드
pip install django --upgrade
```



### 프로젝트 생성

```python
django-admin startproject <project_name>
```



### 애플리케이션 생성

```python
python manage.py startapp <application_name>
```



### MVT 패턴

1. MVC 패턴
   ![image-20200209185245391](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20200209185245391.png)
2. MVT 패턴
   ![image-20200209185256744](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20200209185256744.png)



Django에서는 Control을 View로, View를 Template라는 이름으로 사용한다.
Model은 DB와 통신하고, View에서는 적절한 Template를 찾아서 client에게 넘겨준다.

1. Application
   1. Model: models.py
   2. View: views.py
   3. Template: 따로 사용자가 만들어야 함
2. Project
   1. 설정: settings.py
   2. URL 맵핑: urls.py



# 3장

### Django 흐름도

![image-20200209185920831](C:\Users\user\AppData\Roaming\Typora\typora-user-images\image-20200209185920831.png)



### URLConf

```python
urlpatterns = [
  # path(URL(클라이언트 요청 URL), View(함수 또는 메서드))
  path('student/register/', views.student_register),
  ...
]
```



### View

```python
def registerStudent(request):
  return HttpResponse()
```

클라이언트 요청에 따른 애플리케이션 실행 결과를
Template(html), 여러 메시지 등을 이용해서 클라이언트한테 response한다.



### Model

```python
class Student(models.Model):
  s_name = models.CharField(max_length=30)
  s_major = models.CharField(max_length=30)
```

ORM(Object Relational Mapping)을 사용하여 Django내의 SQLite로 저장된다.
이때 Table의 이름은 `<애플리케이션 이름(소문자)>_<클래스명(소문자)>`이 된다.



### Setting.py

1. Application 생성 후

   ```python
   INSTALLED_APPS = [
     ...
     # 애플리케이션의 apps.py에 있는 설정클래스를 등록해야 한다.
     'students.apps.StudentsConfig'
   ]
   ```

2. 개발모드와 운영모드 설정

   ```python
   # 개발모드
   DEBUG = True
   ALLOWED_HOSTS = []	# 입력하지 않아도 localhost, 127.0.0.1 정의
   
   # 운영모드
   DEBUG = False
   ALLOWED_HOSTS = [<서버의 IP주소>]	# 운영모드시엔 서버의 IP주소를 입력해야함
   ```



### 기본 사용자 및 그룹 테이블 생성

```python
# 사용자 및 그룹 테이블 생성
python manage.py migrate

# 데이터베이스 변경사항 반영
python manage.py makemigrations
python manage.py migrate

# 관리자 계정 생성
python manage.py createsuperuser
```



### 서버 구동

```python
python manage.py runserver 0.0.0.0:8000
```



# 4장

Django에서는 기본적으로 SQLite를 이용한 ORM을 사용한다.



### 테이블 생성

```python
# models.py
from django.db import models

class Student(models.Model):
  s_name = models.CharField(max_length=100)
  s_major = models.CharField(max_length=100)
  s_age = models.IntegerField(default=0)
  s_grade = models.IntegerField(default=0)
  s_gender = models.CharField(max_length=30)

  # __str__을 정의해야 출력등으로 구별하기 쉽다
  def __str__(self):
    return self.s_name
```

> models.CharField: 문자열 필드
> models.IntegerField: 정수 필드



### 장고 shell모드 실행

```python
python manage.py shell
```



### 레코드 추가

```python
# Django Shell
from students.models import Student
qs = Student(...)
qs.save()
```



### 레코드 읽기

```python
# Django Shell
# 데이터 전체 가져오기
qs = Student.obejcts.all()		# QuerySet으로 반환된다.
print(qs[0]) 									# 각 객체는 []을 이용해서 가져올 수 있다.

# 데이터 한개 가져오기
qs = Student.get(...)					# Student객체로 반환된다.
print(qs.s_name)							# 각 속성은 .을 이용해서 가져올 수 있다.
```



### 레코드 필터

```python
# Django Shell
qs = Student.objects.filter(s_age__lt=22)	# s_age가 22이하인 값 필터링
```

| (속성)__<필터> | 설명                             |
| -------------- | -------------------------------- |
| lt             | ~보다 작다                       |
| lte            | ~보다 작거나 같다                |
| gt             | ~보다 크다                       |
| gte            | ~보다 크거나 같다                |
| isnull         | ~ null인 자료 검색               |
| contains       | 특정 문자열을 포함하는 자료 검색 |
| startwith      | 특정 문자열로 시작하는 자료 검색 |
| endwith        | 특정 문자열로 끝나는 자료 검색   |



### 레코드 정렬

```python
# Django Shell
qs = Student.objects.order_by('s_age')	# 오름차순
qs = Student.objects.order_by('-s_age')	# 내림차순
```



### 레코드 수정

```python
# Django Shell
qs.s_major = 'mathmatics'
qs.save()
```



### 레코드 삭제

```python
# Django Shell
qs.delete()
```



# 5장

### TimeZone 수정

```python
# settings.py
# TIME_ZONE = 'UTC'
TIME_ZONE = 'Asia/Seoul'
```



# 6장

### url 설정

```python
# project urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('students/', include('students.urls')),
]

# application urls.py
from django.urls import path
from . import views

app_name = 'students'
urlpatterns = [
    path('reg/', views.regStudent, name='reg'),
  	path('<str:name>/det', views.detStudent, name='stuDet')
]
```

`django.urls의 include`: 다른 어플리케이션내의 파일을 포함할때 사용
`from . import views`: 파일이 있는 어플리케이션의 views를 import
`app_name`: 참조에 사용될 어플리케이션의 이름을 정한다.
`path의 name`: 참조에 사용될 path의 이름을 정한다.
`<str:name>`: path내의 parameter의 타입과 이름을 설정한다.



### view 설정

```python
# application views.py
from django.shortcuts import render

# Create your views here.
def regStudent(request):
  return render(request, 'students/registerStudent.html')
```

`urls.py`에서 정의한 함수를 만들고, 어떤 html파일을 렌더링해서 보낼지 선택한다.
render()의 두번 째 매개변수는 application내의 templates 아래의 경로이다.



### template 설정

```django
<!-- form내부에 정의하며 hidden input으로 
		 CSRF 공격을 막기위한 토큰이 추가된다. -->
{% csrf_token %}

<!-- 프로젝트 내의 다른 url을 사용하는 곳에 삽입 -->
{% url 'students:regCon' %}

<!-- 프로젝트 내의 다른 url을 사용하는 곳에 삽입 + 매개변수 넘기기 -->
{% url 'students:stuMod' student_info.s_name %}
```

어플리케이션내에 `templates\student` 폴더를 만들어 html파일을 저장한다.



### html Redirect

```python
# views.py
from django.http import HttpResponseRedirect
from django.urls import reverse

def regConStudent(request):
  return HttpResponseRedirect(reverse('students:stuAll'))
```

`django.urls의 reverse()`는 django참조형태의 url을 `all/`과 같은 path에서 사용되는 url로 변경해준다.



### Form

```python
# views.py
def regConStudent(request):
  name = request.POST['name']	
```

input의 name태그의 값을 이용하여 form이 보낸 값을 읽어들인다.



### Context

```python
# views.py
def readStudentAll(request):
  qs = Student.objects.all()
  context = {'student_list': qs}
  return render(request, 'students/readStudents.html', context)
```

template에서 사용될 변수를 context에 담아서 render의 세번 째 매개변수로 보낼 수 있다.



### Django Script

```django
<!-- if문 -->
{% if student_list %}
{% endif %}

<!-- if-else문 -->
{% if student_list %}
{% else %}
{% endif %}

<!-- for문 -->
{% for s in student_list %}
	<!-- 변수에 포함된 값 읽어오기 -->
	{{ s.s_name }} 
{% endfor %}
```

