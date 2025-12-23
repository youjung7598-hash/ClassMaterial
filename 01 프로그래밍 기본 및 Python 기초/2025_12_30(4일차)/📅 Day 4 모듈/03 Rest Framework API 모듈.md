### 🔹 Django REST Framework(DRF)
	Django로 "앱이 아닌 API"를 만들기 쉽게 도와주는 도구입니다.  
	즉, 웹페이지(HTML)를 만드는 것이 아니라,  
	모바일 앱이나 다른 웹서비스가 사용할 수 있는 '데이터만 전달하는 통로
	(API)'를 만드는 데 쓰입니다.

###### ◽ Django와 DRF의 차이점
|구분|Django|Django REST Framework|
|---|---|---|
|목적|웹페이지(HTML) 제공|데이터(API) 제공 (예: JSON)|
|사용자|웹브라우저 이용자|모바일 앱, 자바스크립트, 외부 서비스|
|응답 형태|HTML, Template|JSON, XML 등 기계가 읽는 데이터|
◽ 왜 DRF를 쓰나요?
- 복잡한 로직을 단순하게 작성할 수 있어요.
- GET/POST/PUT/DELETE 요청을 쉽게 처리할 수 있어요. 
- 모바일 앱과 통신하는 백엔드 서버를 쉽게 만들 수 있어요.
- 인증/권한/속도제한 등 보안 기능도 포함되어 있어요.
- 자동으로 API 문서 페이지도 만들어줘요.

◽ 실제로 어디에 쓰이나요?
- 모바일 앱의 서버(API) 만들기
- React, Vue 같은 프론트엔드와 연결되는 백엔드 만들기
- 회원가입/로그인/게시글 등록/댓글 관리 같은 기능을 외부에서 쓸 수 있도록 API 제공하기

◽ 쉽게 이해하기 위한 비유
	Django가 사람을 위한 웹사이트라면,  
	DRF는 기계를 위한 웹사이트(API)입니다.

---
### 🔹 Django REST Framework 주요 모듈
| 범주                 | 모듈 예시 (import 대상)                                                                                           | 설명                        |
| ------------------ | ----------------------------------------------------------------------------------------------------------- | ------------------------- |
| **Serializer**     | `from rest_framework import serializers`                                                                    | 모델/데이터를 JSON 등으로 직렬화/역직렬화 |
| **APIView**        | `from rest_framework.views import APIView`                                                                  | 클래스 기반 API 뷰 (CBV)        |
| **GenericView**    | `from rest_framework import generics`                                                                       | CRUD 자동화 클래스 뷰 제공         |
| **Mixins**         | `from rest_framework import mixins`                                                                         | CRUD 동작을 조합할 수 있는 클래스     |
| **ViewSet**        | `from rest_framework import viewsets`                                                                       | RESTful API 구조 자동화        |
| **Router**         | `from rest_framework.routers import DefaultRouter`                                                          | ViewSet을 URL과 연결          |
| **Status 코드**      | `from rest_framework import status`                                                                         | HTTP 상태 코드 정의 모음          |
| **Response**       | `from rest_framework.response import Response`                                                              | JSON 응답을 반환할 때 사용         |
| **Request**        | `from rest_framework.request import Request`                                                                | Django의 request를 확장한 객체   |
| **Permissions**    | `from rest_framework import permissions`                                                                    | 접근 권한 제어                  |
| **Authentication** | `from rest_framework.authentication import TokenAuthentication, SessionAuthentication, BasicAuthentication` | 인증 방식 정의                  |
| **Throttling**     | `from rest_framework.throttling import UserRateThrottle`                                                    | 요청 속도 제한 기능               |
| **Pagination**     | `from rest_framework.pagination import PageNumberPagination, LimitOffsetPagination`                         | 페이지네이션 기능                 |
| **Filtering**      | `from rest_framework import filters`                                                                        | 검색, 정렬 필터링 기능             |
| **Parsers**        | `from rest_framework.parsers import JSONParser, FormParser`                                                 | 요청 본문 파싱기                 |
| **Renderers**      | `from rest_framework.renderers import JSONRenderer, BrowsableAPIRenderer`                                   | 응답 형식 지정기                 |
| **Exceptions**     | `from rest_framework.exceptions import APIException, NotFound`                                              | 예외 처리 도구                  |
| **Schema/Docs**    | `from rest_framework.schemas import get_schema_view`                                                        | API 문서 자동 생성              |
| **Decorators**     | `from rest_framework.decorators import api_view, permission_classes, authentication_classes`                | 함수형 API 뷰용 데코레이터          |
| **Test**           | `from rest_framework.test import APITestCase`                                                               | API 테스트용 클래스 (단위 테스트)     |
| **Settings**       | `from rest_framework.settings import api_settings`                                                          | DRF 설정값 접근                |
| **Fields**         | `from rest_framework.fields import CharField, IntegerField`                                                 | Serializer에서 사용되는 필드 정의   |

---
##### ◽ Serializer
	데이터(모델, 딕셔너리 등)를 JSON 같은 포맷으로 변환하거나, 역으로 JSON을 파이썬 객체로 변환할 때 사용됩니다.

</> 예시코드: 
```python
from rest_framework import serializers

class UserSerializer(serializers.Serializer):
    username = serializers.CharField(max_length=100)
    email = serializers.EmailField()
```

✨ 적용범위:
- `serializers.py`, `views.py` 등 데이터 직렬화/역직렬화 시 사용

---
##### ◽ APIView
	클래스 기반 뷰로 HTTP 메서드(GET, POST 등)를 메서드로 나눠 처리할 수 있습니다.

</> 예시코드: 
```python
from rest_framework.views import APIView
from rest_framework.response import Response

class HelloAPIView(APIView):
    def get(self, request):
        return Response({"message": "Hello, world!"})
```

✨ 적용범위:
- `views.py`에서 기본 클래스형 API 뷰를 작성할 때 사용

---
##### ◽ GenericView
	CRUD 동작을 자동 처리해주는 클래스형 뷰 세트입니다. (`ListAPIView`, `CreateAPIView` 등)

</> 예시코드: 
```python
from rest_framework import generics
from .models import User
from .serializers import UserSerializer

class UserListCreateView(generics.ListCreateAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

✨ 적용범위:
- `views.py`에서 CRUD 기능을 손쉽게 구현할 때 사용
---
##### ◽ Mixins
	`GenericAPIView`와 함께 사용되어 CRUD 기능을 조합할 수 있는 클래스입니다.

</> 예시코드: 
```python
from rest_framework import mixins, generics
from .models import User
from .serializers import UserSerializer

class UserRetrieveUpdateView(mixins.RetrieveModelMixin,
                              mixins.UpdateModelMixin,
                              generics.GenericAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

✨ 적용범위:
- 사용자 정의 동작만 필요한 경우 `GenericAPIView`와 함께 조합
---
##### ◽ ViewSet
	여러 CRUD 메서드를 하나의 클래스에 통합하여 자동으로 라우팅할 수 있습니다.

</> 예시코드: 
```python
from rest_framework import viewsets
from .models import User
from .serializers import UserSerializer

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

✨ 적용범위:
- `views.py`에서 CRUD API를 ViewSet 하나로 구성할 때 사용

---
##### ◽ Router
	ViewSet과 URLConf를 자동으로 연결해주는 라우터입니다.

</> 예시코드: 
```python
from rest_framework.routers import DefaultRouter
from .views import UserViewSet

router = DefaultRouter()
router.register(r'users', UserViewSet)
```

✨ 적용범위:
- `urls.py`에서 ViewSet 라우팅 자동화 시 사용
---
##### ◽ Status 코드
	HTTP 상태 코드를 상수로 제공하여 코드의 가독성을 높입니다.

</> 예시코드: 
```python
from rest_framework import status
from rest_framework.response import Response

return Response({"error": "Unauthorized"}, status=status.HTTP_401_UNAUTHORIZED)
```

✨ 적용범위:
- `views.py`, 예외/응답 반환 시 상태 코드 사용
---
##### ◽ Response
	Django의 `HttpResponse` 대신 JSON 형태 응답을 반환할 수 있게 해줍니다.

</> 예시코드: 
```python
from rest_framework.response import Response

def example_view(request):
    return Response({"message": "Hello"})
```

✨ 적용범위:
- 모든 DRF 뷰에서 API 응답으로 JSON을 반환할 때 사용
---
##### ◽ Request
	Django의 `HttpRequest`를 확장한 객체로, `.data`, `.query_params` 등 다양한 속성 제공

</> 예시코드: 
```python
from rest_framework.views import APIView

class ExampleView(APIView):
    def post(self, request):
        name = request.data.get("name")
        return Response({"greet": f"Hi, {name}"})
```

✨ 적용범위:
- `APIView`, `ViewSet` 등에서 클라이언트 요청 데이터를 읽을 때 사용
---
##### ◽ Permissions
	접근 제어(로그인, 관리자 여부 등)를 정의하는 클래스입니다.

</> 예시코드: 
```python
from rest_framework import permissions

class IsAdminUser(permissions.BasePermission):
    def has_permission(self, request, view):
        return request.user and request.user.is_staff
```

✨ 적용범위:
- `views.py`, `APIView`, `ViewSet`에서 접근 권한 제어
---
##### ◽ Authentication
	사용자의 인증 방식을 정의합니다. (Token, Session, Basic 등)

</> 예시코드: 
```python
from rest_framework.authentication import TokenAuthentication

class ExampleView(APIView):
    authentication_classes = [TokenAuthentication]
```

✨ 적용범위:
- `views.py`, 인증 설정이 필요한 APIView/ViewSet에 적용
---
##### ◽ Throttling
	API 요청 빈도 제한을 설정합니다.

</> 예시코드: 
```python
from rest_framework.throttling import UserRateThrottle

class CustomThrottle(UserRateThrottle):
    rate = '5/min'
```

✨ 적용범위:
- 과도한 요청을 제한하고 API 보호 시 사용 (`settings.py` 또는 뷰에 설정)
---
##### ◽ Pagination
	리스트 응답을 페이지 단위로 나누어 반환합니다.

</> 예시코드: 
```python
from rest_framework.pagination import PageNumberPagination

class StandardResultsSetPagination(PageNumberPagination):
    page_size = 10
```

✨ 적용범위:
- `views.py`에서 페이지네이션이 필요한 리스트 API에 적용
---
##### ◽ Filtering
	검색, 정렬 등 필터 기능을 제공하는 모듈입니다.

</> 예시코드: 
```python
from rest_framework import filters

class UserListView(generics.ListAPIView):
    filter_backends = [filters.SearchFilter]
    search_fields = ['username']
```

✨ 적용범위:
- `views.py`: 검색 기능이 필요한 리스트 API에 적용
---
##### ◽ Parsers
	요청 본문을 다양한 형식(JSON, form 등)으로 파싱합니다.

</> 예시코드: 
```python
from rest_framework.parsers import JSONParser

class ExampleView(APIView):
    parser_classes = [JSONParser]
```

✨ 적용범위:
- `APIView`, `ViewSet`에서 요청 데이터 형식 지정 시
---
##### ◽ Renderers
	응답을 JSON 또는 웹브라우저에 맞게 렌더링합니다.

</> 예시코드: 
```python
from rest_framework.renderers import JSONRenderer

class ExampleView(APIView):
    renderer_classes = [JSONRenderer]
```

✨ 적용범위:
- API 응답을 사용자 또는 개발자 친화적으로 보여줄 때 사용
---
##### ◽ Exceptions
	API 예외 처리를 위한 도구들입니다.

</> 예시코드: 
```python
from rest_framework.exceptions import NotFound

def get_user_or_404(pk):
    raise NotFound("사용자를 찾을 수 없습니다.")
```

✨ 적용범위:
- 뷰에서 API 전용 예외 처리 시 사용
---
##### ◽ Schema/Docs
	자동 API 문서 생성을 위한 도구입니다.

</> 예시코드: 
```python
from rest_framework.schemas import get_schema_view

schema_view = get_schema_view(title='My API')
```

✨ 적용범위:
- Swagger, Redoc, CoreAPI 등의 API 문서화 도구와 연동
---
##### ◽ Decorators
	함수형 뷰에서도 DRF 기능을 적용할 수 있게 하는 데코레이터입니다.

</> 예시코드: 
```python
rom rest_framework.decorators import api_view

@api_view(['GET'])
def hello(request):
    return Response({"message": "Hi"})
```

✨ 적용범위:
- `views.py`: 함수형 뷰에 DRF 기능을 적용할 때 사용
---
##### ◽ Test
	DRF의 API 테스트를 위한 도구입니다.

</> 예시코드: 
```python
from rest_framework.test import APITestCase

class UserTestCase(APITestCase):
    def test_example(self):
        response = self.client.get('/api/users/')
        self.assertEqual(response.status_code, 200)
```

✨ 적용범위:
- `tests.py`에서 API 단위 테스트 수행 시 사용
---
##### ◽ Settings
	DRF 관련 설정을 가져오거나 커스터마이징할 수 있습니다.

</> 예시코드: 
```python
from rest_framework.settings import api_settings

print(api_settings.DEFAULT_RENDERER_CLASSES)
```

✨ 적용범위:
- 전역 설정 접근 또는 커스터마이징이 필요한 경우
---
##### ◽ Fields
	Serializer에서 사용하는 필드들을 정의합니다.

</> 예시코드: 
```python
from rest_framework.fields import CharField, IntegerField

class ExampleSerializer(serializers.Serializer):
    name = CharField()
    age = IntegerField()
```

✨ 적용범위:
- `serializers.py`: 직접 필드 정의가 필요한 경우




