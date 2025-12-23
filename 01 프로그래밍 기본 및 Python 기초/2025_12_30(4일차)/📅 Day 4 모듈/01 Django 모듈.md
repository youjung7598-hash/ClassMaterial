### 🔹 모듈이란?
	모듈은 함수, 변수, 클래스 등 관련된 파이썬 코드를 하나의 파일(.py)
	에 모아 놓은 것입니다.  
	즉, .py 확장자를 가진 파이썬 파일 하나가 바로 모듈입니다.

◽ 모듈의 핵심 개념:
- `도구 상자 역할` 
	자주 쓰는 코드를 미리 저장해 두고 필요할 때마다 꺼내 쓸 수 있어요.
- `재사용 가능`
	한 번 만든 모듈은 `import` 한 줄로 여러 곳에서 사용할 수 있어요.
- `코드 정리`
	기능별로 파일을 나누면 프로그램이 더 깔끔하고 관리하기 쉬워져요.
- `외부 모듈 사용 가능`
	다른 사람이 만든 `math`, `random`, `pandas` 같은 모듈도 설치해서 쓸 수 있어요.

---
### 🔹 모듈 불러오는 방법

◽ 모듈 전체 불러오기

</> 예시코드: import 모듈명
```python
# math 모듈 전체 불러오기
import math

print("루트:", math.sqrt(16))  # math.을 붙여야 함
print("원 넓이:", math.pi * math.pow(5, 2))
```
---
◽ 원하는 항목만 불러오기

</> 예시코드: from 모듈 import 항목
```python
# math 모듈에서 필요한 항목만 불러오기
from math import sqrt, pi, pow

print("루트:", sqrt(16))  # 바로 사용 가능
print("원 넓이:", pi * pow(5, 2))
```
---
◽ 모듈에서 특정 함수만 불러오기

</> 예시코드: 
```python
# 첫번째 방법
from math import factorial

print(factorial(5))

# 두번째 방법
import math

print(math.factorial(5))
```
---
◽ as 키워드
	`as`는 import한 모듈이나 함수, 클래스에 대해 새로운 이름(별칭)을 붙일 때 사용합니다.

📖 문법, 구문(syntax): 
```python
import 모듈명 as 별칭

또는
from 모듈명 import 항목 as 별칭
```

</> 예시코드: 모듈에 별칭 붙이기
```python
import math as m

print(m.sqrt(25))  # math.sqrt 대신 m.sqrt
print(m.pi)        # math.pi 대신 m.pi
```

</> 예시코드: 함수에 별칭 붙이기
```python
from math import factorial as fact

print(fact(5))  # factorial 대신 fact
```
- 긴 함수 이름을 짧고 쉽게 바꿔 쓸 수 있음
---
###### ✨ as 실무에서 많이 쓰는 예
| 라이브러리                        | 별칭             | 사용 목적                                   |
| ---------------------------- | -------------- | --------------------------------------- |
| `numpy`                      | `np`           | 수치 계산, 배열 처리                            |
| `pandas`                     | `pd`           | 표 형식 데이터 처리 (CSV, Excel 등)              |
| `matplotlib.pyplot`          | `plt`          | 그래프 시각화                                 |
| `seaborn`                    | `sns`          | 통계 시각화 (matplotlib 기반)                  |
| `scikit-learn`               | `sk`           | 머신러닝 도구 묶음 (보통 별칭은 안 쓰지만 예외적으로 `sk` 사용) |
| `tensorflow`                 | `tf`           | 딥러닝 프레임워크                               |
| `torch` (PyTorch)            | `torch` 또는 `t` | 딥러닝 프레임워크                               |
| `django.urls`                | `urls`         | URL 라우팅 관련 도구 접근                        |
| `django.urls.reverse`        | `r`            | URL 이름 기반 동적 생성                         |
| `django.shortcuts`           | `short`        | 뷰에서 자주 쓰는 렌더링/리다이렉트 함수                  |
| `django.contrib.auth`        | `auth`         | 로그인/사용자 인증 관련                           |
| `django.http`                | `http`         | 요청 및 응답 관련 객체                           |
| `django.conf`                | `conf`         | 전역 설정 (settings) 접근                     |
| `rest_framework.serializers` | `serializers`  | DRF 직렬화 처리 도구                           |
| `rest_framework.views`       | `views`        | DRF용 뷰 클래스                              |
| `rest_framework.response`    | `response`     | API 응답 처리                               |
| `cv2` (OpenCV)               | `cv2`          | 컴퓨터 비전, 이미지 처리                          |
- Django는 대부분 `from ~ import ~` 방식으로 필요한 항목만 가져오는 것이 일반적이지만,  관리와 유지보수를 위해 별칭을 쓰는 경우도 많습니다, 특히 확장성을 고려한 설계 시.
    
- AI 분야는 별칭이 거의 표준처럼 통용됩니다. (`np`, `pd`, `plt`, `sns`, `tf`, `torch` 등은 사실상 규칙처럼 고정됨)

---
### 🔹 import 속성 확인하기

◽ 모듈 안에 어떤 속성(함수, 클래스, 변수 등)이 있는지 확인하는 방법

1️⃣ `dir()` 함수로 확인하기 : 
```python
import math

print(dir(math))
```

🖨️ 출력결과:
```python
['__doc__', '__loader__', '__name__', '__package__', '__spec__', 'acos', 'acosh', 'asin', 'asinh', 'atan', 'atan2', 'atanh', 'cbrt', 'ceil', 'comb', 'copysign', 'cos', 'cosh', 'degrees', 'dist', 'e', 'erf', 'erfc', 'exp', 'exp2', 'expm1', 'fabs', 'factorial', 'floor', 'fma', 'fmod', 'frexp', 'fsum', 'gamma', 'gcd', 'hypot', 'inf', 'isclose', 'isfinite', 'isinf', 'isnan', 'isqrt', 'lcm', 'ldexp', 'lgamma', 'log', 'log10', 'log1p', 'log2', 'modf', 'nan', 'nextafter', 'perm', 'pi', 'pow', 'prod', 'radians', 'remainder', 'sin', 'sinh', 'sqrt', 'sumprod', 'tan', 'tanh', 'tau', 'trunc', 'ulp']
```
---
2️⃣ `help()` 함수 사용 설명까지 보고 싶을 때
```python
import math

help(math)
```

🖨️ 출력결과:
```python
Help on built-in module math:

NAME
    math

DESCRIPTION
    This module provides access to the mathematical functions...

FUNCTIONS
    acos(...)
    ceil(...)
    sqrt(...)
```
- `help(math)`는 모듈에 대한 자세한 문서(도움말)를 보여줍니다.
- 함수의 사용법, 설명, 매개변수까지 포함되어 있어 매우 유용합니다.
---
3️⃣ 모듈 문서 확인하기:  공식 문서
![[Pasted image 20250522210154.png]]
- 공식 문서는 [https://docs.python.org/3/library/math.html](https://docs.python.org/ko/3/)에서 확인 가능
---
### 🔹 Django `import` 모듈
	Django에서 import 모듈은 다른 파일이나 라이브러리의 기능을 현재 
	코드에서 사용할 수 있도록 불러오는 문법입니다.  
	파이썬의 import는 외부 모듈, 함수, 클래스, 변수 등을 가져와 재사용
	할 수 있게 해주며, Django에서는 뷰, 모델, 설정, 유틸리티 등을 구성할
	때 필수적으로 사용됩니다.

###### ◽ 모듈들
| 분류         | 예시                                                          | 설명                  |
| ---------- | ----------------------------------------------------------- | ------------------- |
| 기본 설정      | `from django.conf import settings`                          | 전역 설정 파일 접근         |
| URL 처리     | `from django.urls import path, include`                     | URL 라우팅 등록          |
|            | `from django.urls import reverse, reverse_lazy`             | URL 이름으로 경로 생성      |
| HTTP 요청/응답 | `from django.http import HttpResponse, JsonResponse`        | 응답 객체 반환            |
|            | `from django.shortcuts import render, redirect`             | 템플릿 렌더링 또는 리다이렉트 처리 |
| 뷰(View)    | `from django.views import View`                             | 클래스형 뷰 기반           |
|            | `from django.views.generic import ListView, DetailView`     | 제네릭 클래스 기반 뷰        |
| 폼 처리       | `from django import forms`                                  | 폼 클래스 정의            |
| 모델(Model)  | `from django.db import models`                              | 모델 필드 및 구조 정의       |
| ORM 쿼리     | `from django.db.models import Q, F`                         | 복잡한 조건 검색           |
| 관리자(admin) | `from django.contrib import admin`                          | 관리자 등록 처리           |
| 사용자 인증/권한  | `from django.contrib.auth.models import User`               | 기본 사용자 모델           |
|            | `from django.contrib.auth.decorators import login_required` | 로그인 필요 뷰 제한         |
|            | `from django.contrib.auth.mixins import LoginRequiredMixin` | 클래스형 로그인 제한         |
| 세션/쿠키      | `from django.contrib.sessions.models import Session`        | 세션 접근               |
| 미들웨어       | `from django.utils.deprecation import MiddlewareMixin`      | 사용자 정의 미들웨어 작성 시 사용 |
| 시간/시간대 처리  | `from django.utils import timezone`                         | 현재 시간, 타임존 관련 도구    |

###### ◽ 부가 도구 및 기타
| 상황         | 예시                                                        |
| ---------- | --------------------------------------------------------- |
| 메시지 처리     | `from django.contrib import messages`                     |
| 파일 업로드     | `from django.core.files.storage import FileSystemStorage` |
| 캐시 사용      | `from django.core.cache import cache`                     |
| 이메일 발송     | `from django.core.mail import send_mail`                  |
| 커스텀 에러 페이지 | `from django.http import Http404`                         |
| CSRF 처리    | `from django.views.decorators.csrf import csrf_exempt`    |

---
### 🔹 Django 모듈 종류

###### ◽ 장고에서 import 실습을 위한 사전 준비
✅ Windows용 (VSCode)
```python
# 1. 새 폴더 생성 및 이동
mkdir my_django_project
cd my_django_project

# 2. 가상환경 생성
python -m venv venv

# 3. 가상환경 활성화
venv\Scripts\activate

# 4. pip 최신화
python -m pip install --upgrade pip

# 5. Django 설치
pip install django
```

✅ macOS용 (VSCode)
```python
# 1. 새 폴더 생성 및 이동 (필요시)
mkdir my_django_project
cd my_django_project

# 2. 가상환경 생성
python3 -m venv venv

# 3. 가상환경 활성화
source venv/bin/activate

# 4. pip 최신화
pip install --upgrade pip

# 5. Django 설치
pip install django
```

✅ Ubuntu용 (VSCode)
```python
# 1. 새 폴더 생성 및 이동 (필요시)
mkdir my_django_project
cd my_django_project

# 2. 가상환경 생성
python3 -m venv venv

# 3. 가상환경 활성화
source venv/bin/activate

# 4. pip 최신화
pip install --upgrade pip

# 5. Django 설치
pip install django
```
---
🖥️ macOS vs Windows 개발 환경 개요: 
- macOS는 기본적으로 유닉스(Unix) 기반 OS이기 때문에, Linux와 거의 동일한 명령어 체계를 사용합니다.
- Windows는 PowerShell이나 CMD 기반이라서 경로 표기나 가상환경 실행 방식이 다릅니다.
- macOS vs Linux의 차이점은 거의 없지만, 시스템에 따라 기본 설치된 Python 버전이나 경로 구조에는 약간의 차이가 있을 수 있습니다.
---
1️⃣ macOS는 이미 리눅스와 거의 동일한 개발 환경:
- macOS는 Darwin 커널을 기반으로 한 유닉스 계열 OS입니다.
- 그래서 리눅스와 거의 모든 터미널 명령어와 개발 환경이 일치합니다.
- 예: `vim`, `nano`, `ls`, `rm`, `source`, `grep`, `ssh`, `curl` 등 대부분 동일.

2️⃣ WSL은 Windows 환경에서 리눅스 개발을 하기 위한 도구:
- Windows는 기본적으로 리눅스와 명령어 구조가 전혀 다릅니다.
- 그래서 리눅스 기반 서버 배포, Docker, Python 등 개발을 제대로 하려면 WSL이 사실상 필수입니다.

3️⃣ macOS는 서버 환경과 유사하여 바로 배포 준비 가능:
- macOS에서 Django + Nginx + Gunicorn 등 실습이 어렵지 않습니다.
- Docker도 원활하게 작동하고, SSH 등 원격 접속도 바로 사용할 수 있습니다.
---
`장고 설치확인`
```bash
python -m django --version
```

`장고 앱 플랫폼 생성`
```bash
django-admin startproject mysite[전체 플랫폼 또는 서비스의 이름]
cd mysite # 생성된 플랫폼 폴더로 이동
```

`장고 설치 결과 확인:`
```python
python manage.py runserver
```

---
📖 문법, 구문(syntax):  `path()` 함수 
```python
path(route, view, kwargs=None, name=None)
```

- `route` (필수)
	사용자가 브라우저 주소창에 입력하는 URL 경로
```python
'hello/'        → http://localhost:8000/hello/
'post/<int:id>/' → 동적 경로 (id는 정수형 파라미터)
```

- `view` (필수)
	URL로 접속했을 때 실행될 뷰 함수 또는 클래스형 뷰
```python
path('hello/', views.hello_view)  # views.py의 hello_view 함수 실행
```

- `kwargs` (선택)
	거의 사용되지 않음, view 함수에 넘겨줄 기본 키워드 인자
```python
path('about/', views.about_view, kwargs={'author': 'admin'})
```

- `name` (선택) html에 연결하려면 필수
	URL 경로에 이름을 부여해서, 나중에 `reverse()` 함수나 템플릿 `{% url %}` 태그에서 사용할 수 있음
```python
path('login/', views.login_view, name='login')
```

템플릿에서는:
```python
<a href="{% url 'login' %}">로그인</a>
```

- 웹구조 예시:
![[Pasted image 20250523212544.png]]
`슬러그(slug)`: 사람이 읽을 수 있는 URL을 만들기 위해 사용됨
	예: /posts/hello-world/ , /article/django-slug-usage/
`쿼리스트링(Query String)` : URL 끝에 붙는 `?` 이후의 문자열을 의미합니다.
	예: `https://example.com/search?keyword=django&page=2`
	`?` → 쿼리스트링의 시작
	`keyword=django&page=2` → key-value 쌍으로 된 파라미터들

---
###### ◽ 전역 설정
	settings.py에 정의된 설정값(예: DEBUG, MEDIA_URL, DATABASES 등)
	을 다른 파일에서 사용할 때 필요합니다.
	예: settings.MEDIA_ROOT를 통해 업로드 파일의 저장 경로에 접근할 
	수 있음.

</> 예시코드: 필요한 항목만 불러오기
```python
# views.py
from django.conf import settings
from django.http import HttpResponse
import os

def show_upload_path(request):
    path = settings.MEDIA_ROOT
    return HttpResponse(f"업로드 경로: {path}")
```

`django.conf` 
	Django의 설정 파일(config)과 관련된 하위 모듈입니다. 여기서 `conf`는 configuration(설정)의 줄임말입니다.
`settings` 
	프로젝트의 `settings.py` 내용을 코드에서 접근할 수 있도록 하는 전역 설정 객체입니다.
`{path}` 변수설정
	`path = settings.MEDIA_ROOT` 이 줄로 인해 변수 `path`가 메모리에 저장되고 `f"..."` 안에서 `{path}`를 쓰면 → 그 값이 문자열 안에 들어갑니다 
`settings.MEDIA_ROOT`
	사용자가 업로드한 이미지나 파일이 저장될 실제 폴더 경로를 지정하는 설정
```python
settings.DEBUG         # settings.py에 설정된 DEBUG 값 가져오기
settings.MEDIA_ROOT    # 업로드 파일 저장 경로 가져오기
```

이렇게 설정하면, 프로젝트 폴더 안의 `media/` 디렉토리에 업로드 파일이 저장됩니다.
```python
# settings.py
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

# /home/사용자명/MY_DJANGO_PROJECT/media/
MEDIA_ROOT = BASE_DIR / "media"     # 업로드 파일 저장 위치
MEDIA_URL = "/media/"               # URL 접근 경로
```

![[Pasted image 20250523194345.png|300]]

`폴더구조`
```python
my_django_project/
│
├── import_test/       ← Django 앱
├── media/             ← 업로드된 이미지 저장될 폴더
├── db.sqlite3
├── manage.py
└── venv/              ← 가상환경
```
---
이 구조에서 `urls.py`에서 `views.py`를 import 할 때:
```python
from . import views 
```
- 같은 디렉토리에서 `views.py` 불러오기
```python
my_django_project/
├── app/
│   ├── __init__.py
│   ├── views.py
│   └── urls.py
```

`urls.py`
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.show_upload_path),
]
```

🖨️ 출력결과:
```python
업로드 경로: /home/사용자명/my_project/media
```

---
</> 실습해보기1 (views.py): `DEBUG` 모드 확인하기
```python
from django.conf import settings
from django.http import HttpResponse

def check_debug_mode(request):
    if settings.DEBUG:
        return HttpResponse("현재 DEBUG 모드입니다. (개발 중)")
    else:
        return HttpResponse("현재 운영 모드입니다. (DEBUG=False)")
```

`Django내에 settings.py 설정`
```python
# settings.py

# 개발 중에는 True, 배포 시에는 반드시 False로 설정
DEBUG = True  # 또는 False로 바꿔서 실습

# 반드시 추가되어야 하는 항목 (로컬 개발 테스트를 위해)
ALLOWED_HOSTS = ['*']
```

`urls.py`에 URL 경로 등록:
```python
from django.urls import path
from . import views

urlpatterns = [
    path('check-debug/', views.check_debug_mode),
]
```

`서버 실행 및 결과 확인:`
```python
python manage.py runserver
```

- [결과확인] [http://localhost:8000/check-debug/](http://localhost:8000/check-debug/)

🔍 해설:
- `settings.DEBUG`는 `settings.py`에서 설정된 `DEBUG = True` 또는 `False` 값을 가져옵니다.
- 개발 단계에서는 보통 `True`, 배포 시에는 `False`로 설정됩니다.
- 실습 시 `settings.py`의 `DEBUG` 값을 `True/False`로 바꿔보고 결과를 확인해 보세요.
---
</> 실습해보기2 예시코드 (views.py): 오류 유발 코드로 DEBUG 모드 차이 확인하기
```python
def trigger_error(request):
    # 일부러 오류 발생시키기 (0으로 나누기 → ZeroDivisionError)
    result = 1 / 0
    return HttpResponse(f"결과: {result}")
```

`urls.py`에 연결 추가:
```python
from django.urls import path
from . import views

urlpatterns = [
    path('check-debug/', views.check_debug_mode),
    path('trigger-error/', views.trigger_error),  # 오류 테스트용
]
```

1️⃣  `DEBUG = True` 상태에서 실행:
```python
http://localhost:8000/trigger-error/
```
- 개발자 친화적인 상세 오류 페이지가 뜨며, Traceback, 파일 경로, 라인 번호 등이 자세히 표시됩니다.

2️⃣ `settings.py`에서 `DEBUG = False`로 변경:
```python
DEBUG = False
ALLOWED_HOSTS = ['*']  # False 상태에서는 필수
```

3️⃣ 다시 서버 실행 → 동일 URL 접속:
```python
http://localhost:8000/trigger-error/
```
- 간단한 500 서버 오류 페이지가 뜹니다.
- 내부적으로는 오류가 발생했지만, 보안상의 이유로 상세 정보는 출력되지 않습니다.
---
###### ◽ 정적 파일(CSS, JS, 이미지 등) URL 접두어(prefix)
	Django에서 `settings.STATIC_URL`은 정적 파일(CSS, JS, 이미지 등)
	을 브라우저에서 접근할 때 사용하는 URL 접두어(prefix)입니다.

</> 실습해보기3  (views.py): `STATIC_URL` 출력하기
```python
from django.conf import settings
from django.http import HttpResponse

def show_static_url(request):
    return HttpResponse(f"STATIC URL은: {settings.STATIC_URL}")
```

🔍 해설:
- 기본적으로 `settings.py`에서 `STATIC_URL = '/static/'` 로 지정됩니다.
- 예: 브라우저에서 `http://localhost:8000/static/style.css`처럼 접근
---
`urls.py`에 URL 경로 등록:
```python
from django.urls import path
from . import views

urlpatterns = [
    path('show-static-url/', views.show_static_url),
]
```

`settings.py`
```python
STATIC_URL = '/static/'
```

`서버 실행 및 결과 확인:`
```python
python manage.py runserver
```

- [결과확인] [http://localhost:8000/show-media-url/](http://localhost:8000/show-media-url/)

---
###### ◽ Django URL 처리
	URL 패턴을 등록하거나, URL 이름을 역으로 찾아내어 경로를 생성할 때 
	사용됩니다.  
	reverse()는 URL 이름으로 실제 경로 문자열을 얻고, path()는 
	라우팅을 구성할 때 사용됩니다.

</> 예시코드: views.py
```python
# urls.py
from django.urls import path, reverse
from django.http import HttpResponse

def redirect_example(request):
    url = reverse("hello")
    return HttpResponse(f"리디렉션 주소: {url}")
```

`urls.py`
```python
from django.urls import path
from django.http import HttpResponse
from . import views

urlpatterns = [
    path("hello/", lambda request: HttpResponse("Hello"), name="hello"),
    path("go/", views.redirect_example),
]
```

`서버 실행 및 결과 확인:`
```python
python manage.py runserver
```

- [결과확인] [http://localhost:8000/show-media-url/](http://localhost:8000/hello/) (http://localhost:8000/go/)

✨ 적용범위:
- `path` → URL 경로를 정의할 때 사용하는 함수
- `reverse` → URL 이름(name)을 기반으로 실제 경로(URL 문자열)를 생성하는 함수

---
###### ◽ 요청/응답 처리
	HTTP 요청에 대한 응답을 생성하거나, 페이지 렌더링 및 리디렉션 
	처리 시 사용합니다.
	브라우저에서 각각의 URL 요청을 통해  
	텍스트 응답, JSON 응답, 404 오류 응답을 직접 확인합니다.

</> 예시코드: views.py
```python
from django.http import HttpResponse, JsonResponse, Http404
from django.shortcuts import render, redirect

def simple_response(request):
    return HttpResponse("일반 텍스트 응답")

def json_response(request):
    return JsonResponse({"message": "JSON 응답"})

def page_not_found(request):
    raise Http404("페이지 없음")
```

`urls.py`
```python
from django.urls import path
from . import views

urlpatterns = [
    path('text/', views.simple_response, name='text'),
    path('json/', views.json_response, name='json'),
    path('not-found/', views.page_not_found, name='not-found'),
]
```

- `[결과확인]`
- (http://localhost:8000/text/) 
- (http://localhost:8000/json/) 
- (http://localhost:8000/not-found/)

✨ 적용범위:
- `views.py`에서 HTTP 응답 생성 및 예외 처리 시 사용

---
###### ◽ 뷰 (클래스형 뷰 포함)
	클래스 기반 뷰(CBV)를 정의하거나, CSRF 보안 설정을 비활성화할 때 
	사용합니다.

공격이름 CSRF = Cross-Site Request Forgery : 교차 사이트 요청 위조
	사용자가 로그인된 사이트를 가장해서,  
	악의적인 요청을 자동으로 보내게 만드는 공격입니다.

마치 은행 사이트에 로그인된 상태에서,  
다른 사이트에서 만든 몰래 보내는 이체 요청 링크를 클릭하게 만드는 것과 같습니다. Django는 CSRF 토큰을 이용해 이를 자동 방지합니다.

`프로젝트 구조:`
```python
MY_DJANGO_PROJECT/
├── manage.py    ← 실행 시작점 (여기서 runserver가 시작됨)
└── import_test/
    ├── urls.py  ← 라우팅은 manage.py가 있는 루트 디렉토리를 기준으로함
    ├── views.py ← 동일 폴더
```

</> 예시코드:  `views.py`
```python
from django.views import View
from django.http import HttpResponse
from django.views.decorators.csrf import csrf_exempt  

# 클래스형 뷰
class HelloView(View):
    def get(self, request): # 클래스 내부에 정의된 메서드
        return HttpResponse("클래스형 GET 응답")

@csrf_exempt
def no_csrf_view(request): # 클래스 바깥에 정의된 일반 함수
    if request.method == "POST":
        return HttpResponse("함수형 POST 응답 (CSRF 검증 없이)")
    else:
        return HttpResponse("POST 요청이 필요합니다.", status=405)
```
`@csrf_exempt`를 사용하면 CSRF 토큰이 없어도 요청을 허용하여 보안 검사를 건너뜁니다. 내부 API 테스트용으로만 사용합니다.

`myproject/urls.py`
```python
from django.contrib import admin
from django.urls import path
from import_test.views import HelloView, no_csrf_view ← 경로설정 
# views.py에서 직접 import

urlpatterns = [
    path('admin/', admin.site.urls),
    path('get/', HelloView.as_view(), name='get'),
    path('nocrsf/', no_csrf_view, name='nocrsf'),
]
```

`서버 실행 및 테스트`
```python
python manage.py runserver
```

- http://localhost:8000/get/ : 클래스형 뷰 테스트 (GET 요청)
- http://localhost:8000/nocrsf/ : CSRF 예외 뷰 테스트 (POST도 가능)

✨ 적용범위:
- `views.py`에서 클래스 기반 뷰 정의 또는 CSRF 예외 처리할 때 사용됨
---
###### ◽ 모델 관련
	데이터베이스 테이블 구조를 정의하거나, 필터링에 사용할 조건(QuerySet)
	을 표현할 때 사용합니다.

</> 예시코드: 
```python
# models.py
from django.db import models

class Article(models.Model):
    title = models.CharField(max_length=200)
    views = models.IntegerField(default=0)

# views.py
from django.db.models import F
def increase_view_count(article):
    article.views = F("views") + 1
    article.save()
```

✨ 적용범위:
- `models.py`: 데이터베이스 모델 정의
- `views.py`: 복잡한 필터링 조건이나 연산 처리 시 `Q`, `F` 사용
---
###### ◽ 폼
	사용자 입력을 처리하거나 유효성 검사를 수행하는 폼을 생성할 때 
	사용합니다.
	
</> 예시코드: 
```python
# forms.py
from django import forms

class ContactForm(forms.Form):
    name = forms.CharField(label="이름")
    message = forms.CharField(widget=forms.Textarea)
```

✨ 적용범위:
- `forms.py`: 사용자 입력폼 생성 및 처리
- `views.py`: `form.is_valid()` 등 유효성 검사 로직에 활용
---
###### ◽ 관리자(admin)
	모델을 Django의 관리자 페이지에서 관리할 수 있도록 등록할 때 
	사용합니다.

</> 예시코드: 
```python
# admin.py
from django.contrib import admin
from .models import Article

admin.site.register(Article)
```

✨ 적용범위:
- `admin.py`: 관리자 사이트에서 모델을 등록하거나 관리 옵션을 설정할 때 사용

---
###### ◽ 사용자 인증 / 권한
	로그인 여부 확인, 접근 제한, 사용자 정보 조회 등에 사용됩니다.

</> 예시코드: 
```python
# views.py
from django.contrib.auth.decorators import login_required
from django.contrib.auth.models import User
from django.http import HttpResponse

@login_required
def user_info(request):
    user = request.user
    return HttpResponse(f"{user.username} 님 환영합니다.")
```

✨ 적용범위:
- `views.py`, `models.py`: 사용자 인증 처리, 권한 제어 시 사용
- 클래스 기반 뷰에서는 `LoginRequiredMixin` 사용
---
###### ◽ 세션
	사용자의 상태(예: 로그인 정보, 장바구니 등)를 서버에 저장할 때 
	사용됩니다.

</> 예시코드: 
```python
# views.py
def save_to_session(request):
    request.session["user_level"] = "basic"
    return HttpResponse("세션 저장 완료")
```

✨ 적용범위:
- `views.py`: 사용자별 상태 정보를 세션에 저장하거나 불러올 때 사용

---
###### ◽ 메시지 프레임워크
	사용자에게 안내 메시지를 보여주기 위해 사용됩니다. (예: 성공, 오류 
	메시지 등)

</> 예시코드: 
```python
# views.py
from django.contrib import messages
from django.shortcuts import redirect

def post_success(request):
    messages.success(request, "저장되었습니다!")
    return redirect("home")
```

✨ 적용범위:
- `views.py`: 메시지 생성
- `template`: 템플릿에서 메시지 표시 (`{% for message in messages %}`)
---
###### ◽ 미들웨어 클래스
	요청 또는 응답을 가로채기 위한 사용자 정의 미들웨어를 만들 때 
	사용됩니다.
	
</> 예시코드: 
```python
# middleware.py
from django.utils.deprecation import MiddlewareMixin

class SimpleMiddleware(MiddlewareMixin):
    def process_request(self, request):
        print("요청 시작")
```

✨ 적용범위:
- `middleware.py`: 요청/응답 처리 전후 로직 삽입 시 사용
- `settings.py`: `MIDDLEWARE`에 등록 필요
---
###### ◽ 시간 및 타임존 처리
	현재 시간, 시간 비교 등을 처리할 때 사용합니다. datetime보다 
	장고 내부에서 권장됩니다.

</> 예시코드: 
```python
# models.py
from django.utils import timezone

class Notice(models.Model):
    created_at = models.DateTimeField(default=timezone.now)
```

✨ 적용범위:
- `models.py`, `views.py`: 날짜/시간 저장 및 비교 시 사용

---
###### ◽ 파일 업로드
	파일을 서버에 저장하는 기본 로직을 사용할 때 사용됩니다.

</> 예시코드: 
```python
# views.py
from django.core.files.storage import FileSystemStorage

def upload_file(request):
    if request.method == "POST" and request.FILES["myfile"]:
        myfile = request.FILES["myfile"]
        fs = FileSystemStorage()
        filename = fs.save(myfile.name, myfile)
        return HttpResponse(f"업로드 완료: {filename}")
```

✨ 적용범위:
- `views.py`: 파일 업로드 처리 로직 구현 시 사용

---
###### ◽ 캐시
	자주 사용되는 데이터를 임시 저장하여 처리 속도를 높일 때 사용됩니다.

</> 예시코드: 
```python
# views.py
from django.core.cache import cache

def cache_example(request):
    cache.set("greeting", "안녕하세요!", timeout=60)
    return HttpResponse(cache.get("greeting"))
```

✨ 적용범위:
- `views.py`, `services.py`: 데이터 캐싱 및 빠른 응답 처리 시 사용

---
###### ◽ 이메일 발송
	회원가입 인증, 알림 등 이메일 발송 기능을 구현할 때 사용합니다.

</> 예시코드: 
```python
# views.py
from django.core.mail import send_mail

def send_test_email(request):
    send_mail("테스트 제목", "테스트 내용", "from@example.com", ["to@example.com"])
    return HttpResponse("이메일 발송 완료")
```

✨ 적용범위:
-  `views.py`, `tasks.py`: 사용자에게 메일 전송 시 사용
---
