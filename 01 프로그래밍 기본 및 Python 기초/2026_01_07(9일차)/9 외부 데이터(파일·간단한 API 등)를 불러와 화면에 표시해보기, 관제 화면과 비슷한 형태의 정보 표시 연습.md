오늘 수업을 한 문장으로
> 외부 데이터(API/CSV)를 받아서 DataFrame으로 만들고, 기준(임계치)으로 위험만 골라서 화면에 보여주는 ‘관제 화면’의 최소형을 만든다.

목표
- API가 뭔지 감으로 이해하기
    - 웹 브라우저 대신 파이썬이 서버에 요청을 보내는 것
- JSON 응답 구조를 보고 필요한 데이터 위치 찾기
- pandas.DataFrame으로 변환해서 표처럼 다루기

데이터 가져오기 → 파싱 → UI 표시 → 필터링/색상 표시 → 실시간 갱신

### 왜 이걸 해야 하나?

`1)` 데이터가 들어오는 서비스를 만들려면 필수다
	게시판/CRUD만 만들면 내 DB 안에서만 놀아요.  
하지만 실무 서비스는 거의 항상:
- 외부 API(날씨/결제/지도/공공데이터)
- 파일(CSV/엑셀)
- 센서/로그 같은 스트림 데이터를 불러와서 화면에 보여주고 판단합니다.
    

`2)` 관제(모니터링)는 전체를 다 보여주는 UI가 아니다
관제는 보통:
- 정상은 넘기고
- 이상만 강조해서
- 우선순위를 정해서 사람이 판단하게 만들어야 합니다.
    
그래서 오늘 핵심은:
> 데이터(DataFrame) + 기준(임계치) = 위험 목록

`3)` API가 항상 성공한다 라고 가정하면 운영 불가능합니다.

실무에서는 API가 자주 실패합니다. 예를 들어
- 인증키 만료/오입력
- 서버 장애(점검, 과부하)
- 응답 형식 변경(JSON → XML/HTML)
- 네트워크/타임아웃 문제가 흔하게 발생합니다.
    
그래서 에러가 발생해도 앱이 멈추지 않도록, 에러 상황을 사용자에게 이해하기 쉬운 UI 메시지로 안내하는 방어 코드까지 구현해야 완성입니다.

`4)` 이후 확장(저장/알림/자동갱신)의 기반이 됩니다.
	오늘 만든 구조는 그대로 다음 단계 기능으로 확장됩니다:
		- CSV 파일 데이터를 데이터베이스(DB)에 저장
		- 특정 수치가 기준을 넘으면 알림 전송 (Slack / Discord 등)
		- 일정 시간마다 데이터를 다시 불러오는 자동 갱신 구조 (캐시 / TTL 적용)
    
👉 즉, 오늘 만든 흐름은 실무 관제·대시보드의 출발점이 됩니다.

---

오늘의 흐름
	데이터 가져오기→파싱→DataFrame 변환→UI 표시→필터링→차트→탐색(필터/정렬)

이 흐름 하나만 몸에 들어오면, DRF/FastAPI 응답도 같은 방식으로 다룰 수 있어요.

---
### 외부 API 불러오기 기본

실습 환경 준비
> uv는 이미 설치되어 있다고 가정합니다.  
> uv는 컴퓨터(WSL)에 한 번만 설치하면 되며,  
> 실습이나 프로젝트마다 중복 설치할 필요는 없습니다.

uv 설치 여부 확인
```bash
uv --version
```
	✅ 설치되어 있는 경우: uv 0.x.x
	❌ 설치되어 있지 않은 경우 : command not found: uv
	uv는 컴퓨터(WSL)에 한 번만 설치하고, 프로젝트마다 .venv 가상환경만 새로 만든다.
- `uv` 자체를 매번 설치 ❌  
- 프로젝트용 가상환경을 매번 생성 ⭕

실습 준비 – 폴더 & 가상환경 만들기
```bash
# 1) 오늘 실습용 폴더 만들기
mkdir day9_api
cd day9_api

# 2) 가상환경 생성 (파이썬 3 기준)
uv venv

# 3) 가상환경 활성화
source .venv/bin/activate
# (프롬프트 앞에 (venv) 가 보이면 성공)
```
이 시점부터는 이 프로젝트 전용 파이썬 환경에서 작업 중 상태입니다.

필요한 라이브러리 설치
```bash
uv pip install requests pandas
```
가상환경을 사용하므로 이 설치는 이 프로젝트에서만 유효합니다.

설치 확인:
```bash
uv pip list
```
아래 항목이 보이면 정상입니다:
- `requests`
- `pandas`

---
### 파일 만들기
- `seoul_air_api.py` : 서울시 OpenAPI에 요청 보내는 메인 실습 파일
- `secrets_example.py` : API KEY를 코드 밖에서 관리하는 예시용 파일

메인 파일 만들기
```bash
code seoul_air_api.py
```
	VSCode가 열리면 아래 코드를 그대로 붙여넣습니다.

---
### 1단계 – “API에 요청 보내기”만 먼저 성공해 보기

🎯 목표는 딱 2개입니다.
1. 파이썬이 서버에 요청을 보낼 수 있다
2. 서버가 준 응답을 문자열(text)로 확인할 수 있다
    
> 아직 데이터를 분석하는 단계가 아니고, 요청/응답이 되는지 감 잡는 단계입니다.

`seoul_air_api.py` (1차 버전)
```python
import requests

# ✅ 1) 서울시 미세먼지 API 기본 URL (예시)
# - API_KEY가 없거나 틀리면: 서울시 서버가 "XML 에러 메시지"를 내려줄 수 있습니다.
API_KEY = "여기에_본인_API_키_문자열_붙이기"

# ✅ URL 구성
# 1/5/ 는 "1번째부터 5개까지 가져와라" 같은 의미로 생각하면 됩니다.
url = f"http://openapi.seoul.go.kr:8088/{API_KEY}/json/airQualityAPI/1/5/"

# ✅ 2) 서버에 GET 요청 보내기
# - timeout을 넣으면 서버가 응답이 없을 때 무한 대기하지 않습니다.
response = requests.get(url, timeout=10)

# ✅ 3) 상태 코드 확인
print("HTTP Status Code:", response.status_code)

# ✅ 4) 서버가 준 응답 원문(text) 일부 출력
print("\n응답 내용 (앞부분만):")
print(response.text[:300])

# ✅ 5) 응답이 JSON인지 XML인지 힌트 확인(중요)
print("\nContent-Type:", response.headers.get("Content-Type"))
```

실행 (가상환경 켜진 상태에서)
```bash
python seoul_air_api.py
```

터미널에 출력결과:
```
HTTP Status Code: 200

응답 내용 (앞부분만):
<RESULT><CODE>INFO-100</CODE><MESSAGE><![CDATA[인증키가 유효하지 않습니다.
인증키가 없는 경우, 열린 데이터 광장 홈페이지에서 인증키를 신청하십시오.]]></MESSAGE></RESULT>

Content-Type: application/xml;charset=UTF-8
```
✅ 이 결과를 이렇게 이해하면 됩니다
- `HTTP Status Code: 200`  
    → 통신은 성공 (서버까지 갔고, 서버가 답장을 줬다)
    
- 그런데 내용이 XML이다  
    → 데이터(JSON)가 아니라 안내문/에러 메시지(XML)를 받은 것
    
핵심:
> 200이면 다 성공이 아니라 200은 서버가 답을 줬다는 뜻이고,  
> 진짜 성공/실패는 response.text 내용을 봐야 한다
	
👉 이 첫 단계에서는 “아 파이썬이 웹에 요청을 보내는구나” 정도만 느끼면 됩니다. 브라우저 대신 requests가 일을 할수 있습니다.

---
### 브라우저 vs `requests` 비교
	같은 URL을 요청해도 브라우저는 사람이 보기 좋은 화면을, 파이썬은 가공하기 좋은 데이터 
	원문을 다룹니다.

| 비교 항목            | 브라우저(크롬, 사파리 등)            | 파이썬 + `requests`            |
| ---------------- | -------------------------- | --------------------------- |
| 누가 실행?           | 사람 손으로 클릭/입력               | 코드가 자동 실행                   |
| 어떤 형태로 봄?      . | 예쁜 웹페이지(HTML+CSS 렌더링)      | 응답 원문(text/json)을 그대로 받음    |
| 반복 작업            | 사람 손으로 계속 해야 함             | for문, 스케줄러로 무한 반복 가능        |
| 용도               | 사람 눈으로 보기, 인터랙션(로그인, 클릭 등) | 데이터 수집, 분석, 자동화, 다른 시스템과 연동 |
### 브라우저 버전 실습 (JSON을 직접 눈으로 보기)

1. 주소창에 입력:
    `https://jsonplaceholder.typicode.com/posts/1`
    
2. JSON처럼 생긴 글자가 화면에 보이면 OK
![[Pasted image 20251220180833.png]]

파이썬 버전 (같은 걸 코드로 요청해보기)

파일 생성
```bash
code browser_json.py
```

browser_json.py 파일을 생성한후 아래 코드를 작성합니다.
```python
import requests
# 웹 서버에 HTTP 요청을 보내기 위한 라이브러리

url = "https://jsonplaceholder.typicode.com/posts/1"
# 요청을 보낼 API 주소 (게시글 1개를 달라는 예시용 URL)

response = requests.get(url, timeout=10)
# 서버에 GET 요청 전송
# timeout=10 → 서버가 10초 안에 응답하지 않으면 중단

print("HTTP Status Code:", response.status_code)
# 서버가 요청을 정상적으로 받았는지 확인 (200이면 통신 성공)

print("\n응답 원문:")
print(response.text)
# 서버가 보내준 실제 응답 내용(문자열)
# JSON처럼 보이지만, 파이썬 입장에서는 아직 '글자(text)'
```

터미널에서 실행:
```bash
python browser_json.py
```

출력결과:
```
HTTP Status Code: 200

응답 원문:
{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
  "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
}
```
이건 JSONPlaceholder 서버가 정상적으로 JSON 데이터를 돌려준 것이고, 파이썬이 웹 서버에 요청 → 서버가 JSON 응답 → 그대로 출력 이 흐름이 100% 성공했다는 뜻입니다.

=> 둘 다 같은 서버, 같은 URL에 요청하지만,
- 브라우저는 화면에 예쁘게 보여주는 용도
- 파이썬은 데이터를 가져와서 코드로 바로 처리하는 용도

이 차이를 알면,
`response.json()`, `pandas.DataFrame` 다음에 이런 함수에 대해 이해하기 쉽습니다.

이 단계에서는 딥하게 API가 뭐고 HTTP가 뭔지 다 이해시키려고 하기보다,  
‘지금은 크롬이 아니라 파이썬 코드가 웹에 요청을 보내고, 서버가 응답을 돌려주면 우리가 그 데이터를 코드로 다룰 수 있다’ 이 감각만 잡아도 충분합니다.

---
### 2단계 – `.json()`으로 파이썬 dict로 변환해 보기

지금까지 우리는 파이썬 코드로 웹 서버에 요청을 보내고, 서버가 어떤 응답을 돌려주는지 확인했습니다.

이제 목표는 한 가지입니다.
> 서버가 준 텍스트(JSON)를 파이썬이 다룰 수 있는 자료구조(dict)로 바꾸기

### 🌱 먼저 꼭 알아야 할 아주 중요한 개념 3가지

이 3가지만 이해하면, **API·JSON·에러가 왜 나는지**가 한 번에 정리됩니다.

---
1️⃣ 서버가 보내주는 건 항상 글자(text)다
```python
response.text
```

이게 무슨 뜻이냐면요…
서버는 우리에게 엑셀 파일이나 딕셔너리를 직접 주는 게 아닙니다.
👉 항상 글자(문자열)로만 줍니다.

---
예를 들어 보면

서버가 주는 내용이
- JSON처럼 보이든
- XML처럼 보이든
- 웹페이지(HTML)처럼 보이든

파이썬 눈에는 전부 그냥 글자입니다.
```
"{ 'name': 'seoul', 'pm10': 30 }"
"<RESULT><CODE>INFO-100</CODE></RESULT>"
"<html><body>Hello</body></html>"
```
👉 전부 문자열(str) 그래서 처음에는 항상 이렇게 확인합니다.

```python
print(response.text)
```
	서버가 나한테 무슨 글자를 줬는지 먼저 본다

---
2️⃣ `.json()`은 JSON일 때만 쓰는 변환기다
```python
response.json()
```

이걸 이렇게 생각하면 됩니다 👇
> .json()은 “이 글자가 JSON 맞지? 그럼 파이썬 딕셔너리로 바꿔줄게”라는 변환기입니다.

---
그래서 중요한 규칙 1개
- 서버가 JSON을 줬을 때만 사용 가능
- JSON이 아니면 ❌ 바로 에러

---
비유로 설명하면
- `.json()`은
    👉 한글 문서를 엑셀로 변환하는 버튼 같은 것
    
그런데 서버가 준 게
- 엑셀 ❌
- 이미지 ❌
- PDF ❌
라면?
👉 “엑셀로 바꿀 수 없습니다” 하고 에러가 나는 것과 똑같습니다.

---
그래서 이렇게 해야 합니다
```
1️⃣ response.text 로 내용 확인
2️⃣ JSON처럼 생겼는지 확인
3️⃣ 그 다음에 response.json()
```
	무조건 순서 중요

---
3️⃣ HTTP 상태 코드 200은 데이터 성공이 아니다
```
HTTP Status Code: 200
```
	이건 초보자들이 가장 많이 착각하는 부분입니다.

200의 진짜 의미
> 요청은 잘 받았고, 서버가 뭔가를 응답으로 돌려줬다

✔️ 요청 성공
❌ 데이터 성공 아님

---
예를 들면 이런 상황입니다
- 회사 건물에 들어감 → 출입 성공 (200)
- 하지만
    - 출입증(API 키) 없음
    - 담당자 없음
    
👉 일은 못 보고 안내문만 받고 나옴

---
실제로 서버는 이렇게 말할 수도 있습니다
```
HTTP 200
"인증키가 없습니다"
"권한이 없습니다"
"잘못된 요청입니다"
```
	서버 입장에서는 정상적으로 대답했기 때문에 200

그래서 실무/관제에서는 이렇게 판단합니다
```
상태 코드 = 통신 성공 여부
응답 내용 = 진짜 성공/실패 여부
```
	항상 response.text를 봐야 하는 이유

---
#### 🧠 최종 정리 (이 4줄만 기억하세요)
1️⃣ 서버는 항상 글자(text) 로 응답한다
2️⃣ `.json()`은 JSON일 때만 써야 한다
3️⃣ HTTP 200 = 통신 성공이지, 데이터 성공은 아니다
4️⃣ 그래서 항상 response.text부터 확인한다

> API에서 에러가 났을 때 코드가 틀린 게 아니라, 서버가 JSON이 아닌 안내문을 준 경우가 대부분입니다.

---
`seoul_air_api.py` 수정
```python
import requests
import pprint  # 딕셔너리를 보기 좋게 출력하기 위한 도구

# 1️⃣ 서울시 미세먼지 API 주소
# ⚠️ 아직은 인증키를 넣지 않은 상태
API_KEY = "여기에_본인_API_키_문자열_붙이기"
url = f"http://openapi.seoul.go.kr:8088/{API_KEY}/json/airQualityAPI/1/5/"

# 2️⃣ 서버에 요청 보내기
response = requests.get(url, timeout=10)

# 3️⃣ HTTP 상태 코드 확인
print("HTTP Status Code:", response.status_code)

# 4️⃣ 서버가 준 '원본 텍스트' 확인
print("\n📦 응답 원문 미리보기:")
print(response.text[:300])   # 너무 길어서 앞부분만 출력

# 5️⃣ 응답의 데이터 형식(Content-Type) 확인
print("\n📦 Content-Type:")
print(response.headers.get("Content-Type"))

# 6️⃣ JSON일 때만 .json() 사용
if response.headers.get("Content-Type", "").startswith("application/json"):
    print("\n✅ JSON 형식 확인 → 파이썬 dict로 변환 시도")

    data = response.json()   # JSON → dict
    print("\n🔍 타입 확인:", type(data))   # dict 확인

    print("\n📦 전체 구조(일부):")
    pprint.pprint(data, width=80, depth=2)

else:
    print("\n❌ JSON이 아님")
    print("👉 현재 응답은 XML 또는 에러 메시지입니다.")
```

코드실행하기:
```bash
python seoul_air_api.py
```

출력결과:
```
HTTP Status Code: 200

📦 응답 원문 미리보기:
<RESULT><CODE>INFO-100</CODE><MESSAGE><![CDATA[인증키가 유효하지 않습니다.
인증키가 없는 경우, 열린 데이터 광장 홈페이지에서 인증키를 신청하십시오.]]></MESSAGE></RESULT>

📦 Content-Type:
application/xml;charset=UTF-8

❌ JSON이 아님 → 현재는 XML 또는 에러 메시지
(venv) youjung@DESKTOP-PJCRMMU:~/day1_console/day9_api$ python seoul_air_api.py
HTTP Status Code: 200

📦 응답 원문 미리보기:
<RESULT><CODE>INFO-100</CODE><MESSAGE><![CDATA[인증키가 유효하지 않습니다.
인증키가 없는 경우, 열린 데이터 광장 홈페이지에서 인증키를 신청하십시오.]]></MESSAGE></RESULT>

📦 Content-Type:
application/xml;charset=UTF-8

❌ JSON이 아님
👉 현재 응답은 XML 또는 에러 메시지입니다.
```

### 🧠 이 출력 결과를 이렇게 이해하면 됩니다

✔️ 1. 요청은 성공했다
```
HTTP Status Code: 200
```
	서버까지는 정상적으로 도착

---
❌ 2. 하지만 데이터는 실패
```xml
인증키가 유효하지 않습니다
```
- 서울시 API는 인증키가 없으면
    - JSON 데이터 ❌
    - XML 에러 메시지 ⭕ 를 돌려줌

---
❌ 3. 그래서 `.json()`을 쓰면 안 되는 상태
```
Content-Type: application/xml
```
- JSON 아님 → `.json()` 사용 ❌
- 지금은 에러 메시지를 확인하는 단계

---
📌 지금 단계에서 꼭 기억해야 할 핵심 정리
`1` `response.text`
> 서버가 준 원본 문자열

`2` `response.json()`
> JSON일 때만 파이썬 `dict` / `list` 로 변환

`3.` 실무/관제에서는 항상 이 순서
```
요청
 → response.status_code
 → response.text
 → Content-Type 확인
 → JSON일 때만 response.json()
```

---
## 3단계 – 필요한 위치(row)까지 “안전하게” 들어가 보기

앞 단계에서 우리는 중요한 사실 하나를 확인했습니다.
> 서버가 JSON을 주면 파이썬에서는 그 JSON이 딕셔너리(dict)와 리스트(list)로 바뀐다는 것

이제부터 할 일은 아주 단순합니다.
> 큰 박스(JSON) 안에 뭐가 들어 있는지 하나씩 열어보면서 실제 데이터가 어디 있는지 찾기

우리가 하고 싶은 건 “미세먼지 데이터”인데, 서버는 데이터를 바로 주지 않고 큰 JSON 상자 안에 넣어서 줍니다.

그래서 우리는 JSON을 이렇게 탐색합니다:
- 바깥 상자(dict) →
- 그 안 상자(dict) →
- 데이터 목록(list) →
- 데이터 1개(dict)
    
즉, 3단계는 JSON 구조를 읽는 능력을 키우는 단계입니다.

---
🎯 **3단계의 목표**

이 단계에서의 목표는 딱 3가지입니다.
- 최상위 키가 무엇인지 확인한다 (`data.keys()`)
- dict → dict → list 순서로 내려간다 (`data["airQualityAPI"]["row"]`)
- 데이터 1개(dict)를 직접 눈으로 확인한다 (`rows[0]`)

👉 즉, JSON 안에서 실제 데이터가 어디에 숨어 있는지 찾는 연습을 하는 단계입니다

---
### 🌱 먼저 그림으로 이해해보기 (개념)

API에서 받아오는 데이터는 바로 쓰기 좋은 형태가 아닙니다.

서울시 공공데이터 응답은 보통 이런 구조입니다:
```json
{
"airQualityAPI":{
	"list_total_count":50,
	"RESULT":{ ...},
	"row":[
		{"MSRSTE_NM":"강남구","PM10":20},
		{"MSRSTE_NM":"영등포구","PM10":40}
		]
	}
}
```

이걸 박스로 비유하면 👇
```
data (dict)
 └─"airQualityAPI" (dict)
     ├─"list_total_count"
     ├─"RESULT"
     └─"row" (list)
          ├─0번째 데이터 (dict)
          ├─1번째 데이터 (dict)
          └─ ...
```

👉 우리가 진짜로 쓰고 싶은 데이터는 `"row"` 안에 들어 있는 리스트(list) 입니다.

그래서 이 구조를 읽을 수 있어야
- pandas DataFrame으로 바꿀 수 있고
- 필터링, 시각화, 관제 대시보드로 확장할 수 있습니다.

---
⚠️ 반드시 알고 가야 할 전제

❗ HTTP Status Code 200 ≠ JSON 성공
```
HTTP Status Code: 200
```

이 말은 단지:
> 요청을 서버가 받았고, 무언가 답을 줬다 라는 뜻입니다.

- ✅ 정상 데이터(JSON)일 수도 있고
- ❌ 에러 메시지(XML)일 수도 있습니다
    
👉 그래서 3단계에서는 반드시

> 지금 받은 응답이 JSON인지 먼저 확인해야 합니다.

---

### 실습 코드:

`seoul_air_api.py`
```python
import requests
import pprint  # 딕셔너리를 보기 좋게 출력하기 위한 도구

API_KEY = "여기에_본인_API_키_문자열_붙이기"
url = f"http://openapi.seoul.go.kr:8088/{API_KEY}/json/airQualityAPI/1/5/"

# 1️⃣ 서버에 요청 보내기
response = requests.get(url, timeout=10)

# 2️⃣ HTTP 상태 코드 확인
print("HTTP Status Code:", response.status_code)

# 3️⃣ 서버가 준 '원본 텍스트' 확인
print("\n📦 응답 원문 미리보기:")
print(response.text[:300])

# 4️⃣ 응답의 데이터 형식(Content-Type) 확인
content_type = response.headers.get("Content-Type", "")
print("\n📦 Content-Type:")
print(content_type)

# =====================================================
# 🔧 [수정] JSON일 때만 파싱하도록 안전장치 유지
# =====================================================
is_json = content_type.lower().startswith("application/json")

if is_json:
    print("\n✅ JSON 형식 확인 → 파이썬 dict로 변환합니다.")

    data = response.json()   # JSON → dict
    print("\n🔍 타입 확인:", type(data))   # dict 확인

    # =================================================
    # 🔵 [추가] 3단계: JSON 구조 안으로 하나씩 들어가기
    # =================================================
    print("\n--- 🔍 3단계: JSON 구조 해석 시작 ---")

    # 🔵 [추가-1] 최상위 키 확인
    print("\n1️⃣ 최상위 키:", data.keys())

    # 🔵 [추가-2] 'airQualityAPI' 키 존재 여부 확인
    if "airQualityAPI" not in data:
        print("\n❌ 'airQualityAPI' 키가 없습니다.")
        print("👉 응답 구조가 예상과 다릅니다.")
        pprint.pprint(data, width=80, depth=3)

    else:
        # 🔵 [추가-3] airQualityAPI 안으로 진입
        air = data["airQualityAPI"]
        print("\n2️⃣ air 타입:", type(air))
        print("   air 키:", air.keys())

        # 🔵 [추가-4] 실제 데이터가 들어있는 'row' 확인
        if "row" not in air:
            print("\n❌ 'row' 키가 없습니다.")
            print("👉 데이터가 없거나 응답 구조가 다릅니다.")
            pprint.pprint(air, width=80, depth=3)

        else:
            rows = air["row"]
            print("\n3️⃣ row 타입:", type(rows))   # list
            print("   row 길이:", len(rows))

            # 🔵 [추가-5] 첫 번째 데이터만 출력
            if len(rows) == 0:
                print("\n⚠️ row 리스트가 비어 있습니다.")
            else:
                print("\n4️⃣ 첫 번째 측정 데이터:")
                pprint.pprint(rows[0], width=80)

# =====================================================
# 🔧 [유지] JSON이 아닐 때는 파싱하지 않음
# =====================================================
else:
    print("\n❌ JSON이 아님")
    print("👉 현재 응답은 XML 또는 에러 메시지입니다.")
    print("👉 인증키가 유효한지 확인한 뒤 다시 실행하세요.")
```

실행 방법:
```bash
python seoul_air_api.py
```

실행 결과
```
HTTP Status Code: 200

📦 응답 원문 미리보기:
<RESULT><CODE>INFO-100</CODE><MESSAGE><![CDATA[인증키가 유효하지 않습니다.
인증키가 없는 경우, 열린 데이터 광장 홈페이지에서 인증키를 신청하십시오.]]></MESSAGE></RESULT>

📦 Content-Type:
application/xml;charset=UTF-8

❌ JSON이 아님 → 현재는 XML 또는 에러 메시지
(venv) youjung@DESKTOP-PJCRMMU:~/day1_console/day9_api$ python seoul_air_api.py
HTTP Status Code: 200

📦 응답 원문 미리보기:
<RESULT><CODE>INFO-100</CODE><MESSAGE><![CDATA[인증키가 유효하지 않습니다.
인증키가 없는 경우, 열린 데이터 광장 홈페이지에서 인증키를 신청하십시오.]]></MESSAGE></RESULT>

📦 Content-Type:
application/xml;charset=UTF-8

❌ JSON이 아님
👉 현재 응답은 XML 또는 에러 메시지입니다.
(venv) youjung@DESKTOP-PJCRMMU:~/day1_console/day9_api$ python seoul_air_api.py
HTTP Status Code: 200

📦 응답 원문 미리보기:
<RESULT><CODE>INFO-100</CODE><MESSAGE><![CDATA[인증키가 유효하지 않습니다.
인증키가 없는 경우, 열린 데이터 광장 홈페이지에서 인증키를 신청하십시오.]]></MESSAGE></RESULT>

📦 Content-Type:
application/xml;charset=UTF-8

❌ JSON이 아님
👉 현재 응답은 XML 또는 에러 메시지입니다.
👉 인증키가 유효한지 확인한 뒤 다시 실행하세요.
```

🖥 실행 결과 해석

🔴 지금처럼 인증키가 없을 때 (정상 상황)
```
📦 Content-Type:
application/xml;charset=UTF-8

❌ JSON이 아님
👉 현재 응답은 XML 또는 에러 메시지입니다.
```

이 뜻은:
> ❌ 코드가 틀린 게 아님
> ❌ 3단계를 잘못 이해한 것도 아님
> 👉 서버가 JSON 데이터 대신 에러 안내문(XML)을 준 상태

그래서 3단계는 실행되지 않는 것이 맞습니다.

---

🟢 인증키가 유효할 때 (3단계 정상 실행)
```
1️⃣ 최상위 키: dict_keys(['airQualityAPI'])

2️⃣ air 타입: <class'dict'>
   air 키: dict_keys(['list_total_count','RESULT','row'])

3️⃣ row 타입: <class'list'>
   row 길이:5

4️⃣ 첫 번째 측정 데이터:
{'MSRSTE_NM':'중구','PM10':23, ...}
```

---

✔️ 우리가 3단계에서 실제로 한 일
1. JSON 전체를 **dict**로 받았다
2. 가장 바깥 키가 무엇인지 확인했다
3. 그 안의 dict(`airQualityAPI`)로 들어갔다
4. 실제 데이터 목록(list, `row`)을 찾았다
5. 데이터 1개(dict)를 직접 눈으로 확인했다

🧠 핵심 공식 (반드시 기억)
```
JSON 구조 해석 공식
dict →dict →list →dict
```

이 공식은
- 서울시 API
- 공공데이터
- 모든 REST API
- DRF 응답 구조
에서 100% 그대로 반복됩니다.

---
## 4단계 – pandas DataFrame으로 변환해서 “표(엑셀)”처럼 보기

🎯 4단계의 목적 (왜 하는가?)
3단계까지는 JSON 구조를 해석해서 row(데이터 목록)를 찾는 연습이었다면,

4단계의 목적은
> “row(list)에 들어 있는 데이터를 pandas DataFrame으로 변환해서 표(엑셀)처럼 다루는 것”

###### ✅ 왜 DataFrame이 중요한가?
| JSON(list/dict) | pandas DataFrame |
| --------------- | ---------------- |
| 구조는 명확하지만 보기 불편 | 엑셀처럼 한눈에 보기 쉬움   |
| 값 하나하나 접근해야 함   | 컬럼 단위 처리 가능      |
| 가공·분석 어려움       | 필터·정렬·통계·차트 가능   |

즉, 관제 대시보드 / 분석 / 시각화의 출발점 = DataFrame

---

🧠 4단계 핵심 개념 3가지
`1)` `rows`는 딕셔너리들이 들어있는 리스트(list)

3단계에서 찾은 이것:
```python
rows = air["row"]
```

`rows` 형태는 대략 이런 느낌:
```python
[
  {"MSRSTE_NM":"강남구","PM10":20, ...},
  {"MSRSTE_NM":"종로구","PM10":30, ...},
  ...
]
```

즉,
- 리스트 안에
- 딕셔너리 여러 개 들어있음 
- 각 딕셔너리 = 한줄(row) 데이터

---

`2)` `pd.DataFrame(rows)`는 “리스트(행들)를 표로 바꿔주는 함수”
```python
df = pd.DataFrame(rows)
```

이 한 줄로 벌어지는 일:
- 딕셔너리의 키들은 **컬럼명**이 되고
- 딕셔너리 하나가 **한 줄(row)** 즉 표의 한 행이 됩니다.

---

`3)` `.head()` / `.columns`로 “표가 잘 만들어졌는지” 확인
```python
df.head()
df.columns.tolist()
```
- `df.head()` → 상위 5줄 미리보기
- `df.columns.tolist()` → 컬럼 이름 확인

---

### 실습 코드

`seoul_air_api.py`
```python
# =====================================================
# 0️⃣ import 구문
# =====================================================
import requests
import pprint  # 딕셔너리를 보기 좋게 출력하기 위한 도구
import pandas as pd  # DataFrame 변환용


# =====================================================
# 1️⃣ API 설정
# =====================================================
API_KEY = "여기에_본인_API_키_문자열_붙이기"

# ❌ url 문자열에 < > 가 들어가 있으면 안 됨
# ❌ f"<http://...>"  ← 잘못된 형식
# ✅ 정상적인 f-string URL
url = f"http://openapi.seoul.go.kr:8088/{API_KEY}/json/airQualityAPI/1/50/"


# =====================================================
# 2️⃣ 서버에 요청 보내기
# =====================================================
response = requests.get(url, timeout=10)

# =====================================================
# 3️⃣ HTTP 상태 코드 확인
# =====================================================
print("HTTP Status Code:", response.status_code)

# =====================================================
# 4️⃣ 서버가 준 원본 텍스트 확인
# =====================================================
print("\n📦 응답 원문 미리보기:")
print(response.text[:300])

# =====================================================
# 5️⃣ 응답 데이터 형식(Content-Type) 확인
# =====================================================
content_type = response.headers.get("Content-Type", "")
print("\n📦 Content-Type:")
print(content_type)


# =====================================================
# ✅ JSON일 때만 파싱하도록 안전장치
# =====================================================
is_json = content_type.lower().startswith("application/json")

if is_json:
    # ❌ if 다음 줄에 들여쓰기 없었음
    # ✅ 반드시 들여쓰기 필요
    print("\n✅ JSON 형식 확인 → 파이썬 dict로 변환합니다.")

    data = response.json()  # JSON → dict
    print("\n🔍 타입 확인:", type(data))  # dict 확인


    # =================================================
    # ✅ 3단계: JSON 구조 안으로 하나씩 들어가기
    # =================================================
    print("\n--- 🔍 3단계: JSON 구조 해석 시작 ---")

    print("\n1️⃣ 최상위 키:", data.keys())

    # ❌ if"airQualityAPI"notin data:  (띄어쓰기 없음)
    # ✅ 파이썬 문법상 반드시 띄어쓰기 필요
    if "airQualityAPI" not in data:
        print("\n❌ 'airQualityAPI' 키가 없습니다.")
        print("👉 응답 구조가 예상과 다릅니다.")
        pprint.pprint(data, width=80, depth=3)

    else:
        air = data["airQualityAPI"]
        print("\n2️⃣ air 타입:", type(air))
        print("   air 키:", air.keys())

        if "row" not in air:
            print("\n❌ 'row' 키가 없습니다.")
            print("👉 데이터가 없거나 응답 구조가 다릅니다.")
            pprint.pprint(air, width=80, depth=3)

        else:
            rows = air["row"]
            print("\n3️⃣ row 타입:", type(rows))  # list
            print("   row 길이:", len(rows))

            # ❌ iflen(rows) ==0:  (띄어쓰기 없음)
            # ✅ 정상 문법
            if len(rows) == 0:
                print("\n⚠️ row 리스트가 비어 있습니다.")
            else:
                print("\n4️⃣ 첫 번째 측정 데이터:")
                pprint.pprint(rows[0], width=80)


            # =================================================
            # ✅ 4단계: pandas DataFrame으로 변환
            # =================================================
            print("\n--- 📊 4단계: DataFrame 변환 시작 ---")

            # ❌ df = pd.DataFrame(rows) 가 들여쓰기 틀어져 있었음
            # ✅ rows가 존재하는 블록 안에서 실행
            df = pd.DataFrame(rows)

            print("\n📊 DataFrame.head() 결과:")
            print(df.head())

            print("\n📌 컬럼 목록:")
            print(df.columns.tolist())

            print("\n📌 (선택) 자주 보는 컬럼만 출력 예시:")

            wanted_cols = ["MSRSTE_NM", "PM10", "PM25"]

            # ❌ [cfor cin wanted_colsif cin df.columns]
            # ✅ 리스트 컴프리헨션 문법 수정
            existing_cols = [c for c in wanted_cols if c in df.columns]

            if existing_cols:
                print(df[existing_cols].head())
            else:
                print("⚠️ 예시 컬럼(MSRSTE_NM, PM10, PM25)이 현재 응답에 없습니다.")
                print("👉 위의 '컬럼 목록'에서 실제 컬럼명을 확인해 주세요.")

# =====================================================
# ❌ JSON이 아닐 때
# =====================================================
else:
    print("\n❌ JSON이 아님")
    print("👉 현재 응답은 XML 또는 에러 메시지입니다.")
    print("👉 인증키가 유효한지 확인한 뒤 다시 실행하세요.")
```

실행
```bash
python seoul_air_api.py
```

---

🖥 출력 결과
```
HTTP Status Code: 200

📦 응답 원문 미리보기:
<RESULT><CODE>INFO-100</CODE><MESSAGE><![CDATA[인증키가 유효하지 않습니다.
인증키가 없는 경우, 열린 데이터 광장 홈페이지에서 인증키를 신청하십시오.]]></MESSAGE></RESULT>

📦 Content-Type:
application/xml;charset=UTF-8

❌ JSON이 아님
👉 현재 응답은 XML 또는 에러 메시지입니다.
👉 인증키가 유효한지 확인한 뒤 다시 실행하세요.
```

✅ 출력 결과 해석

❌ 이 출력은 “코드 실패”가 아닙니다
현재 상태는:
- HTTP 요청은 정상 (200)
- 하지만 **응답 내용이 JSON이 아니라 XML**
- 이유는 **인증키가 아직 유효하지 않기 때문**

그래서:
- `response.json()`을 **의도적으로 실행하지 않았고**
- 프로그램은 **에러 없이 정상 종료됨**

👉 이건 오히려 잘 만든 코드의 증거입니다

---
✅ 인증키가 정상일 경우, 달라지는 점

인증키가 유효해지면:
```
Content-Type:
application/json
```
위와 같이 이렇게 바뀌고,
그 순간부터:
- 3단계(JSON 구조 해석)
- 4단계(DataFrame 변환) 이 자동으로 실행됩니다.
---
✅ 한 줄 요약

> API(JSON)에서 row(list)를 찾았다면 → DataFrame으로 바꾸는 순간 “관제 대시보드의 재료”가 된다.

---
## 5단계 – 간단한 필터링까지 맛보기

🎯 5단계의 목적
4단계에서 `rows(list)`를 `DataFrame(df)`로 바꿨죠?

이제부터가 “관제 느낌”의 시작입니다.
> 관제 화면은 보통 “전체 데이터”를 다 보여주지 않고 **위험한 것 / 기준 이상인 것만 걸러서 보여주는 화면**이에요.

그래서 5단계 목표는 딱 하나입니다.
✅ 기준(임계치)을 정하고, 그 이상인 데이터만 골라서 출력해보기

예:
- PM10이 50 이상인 지역만 보기
- “나쁨 이상 지역만 보여줘!” 같은 요구를 코드로 구현하는 것

---

🧠 5단계 핵심 개념 3가지

`1)` 필터링은 “True인 행만 남기는 것”
```python
df["PM10"] >=50
```

이 결과는 “True/False”로 된 목록(Series)이 됩니다.

그리고 이걸 `df[ ... ]` 안에 넣으면:
```python
bad_air = df[df["PM10"] >=50]
```

✅ 조건이 True인 행만 남습니다.

---

`2)` 관제에서 중요한 건 “기준값(threshold)”
- PM10 50 이상 → 나쁨 이상
- PM10 80 이상 → 매우 나쁨
    처럼 기준이 있어야 “위험”을 판단할 수 있어요.
    
즉, 관제는 사실상:
> (데이터) + (기준) → 필터링 결과(경고 목록)

---

`3)` DataFrame이니까 “원하는 컬럼만 뽑아 보여주기”도 쉬움
```python
bad_air[["MSRSTE_NM","PM10"]]
```

- 관제 화면에서는 전체 컬럼을 다 보여주기보다 **핵심 컬럼만 뽑아서** 보여주는 경우가 많습니다.

---

### 실습 코드

seoul_air_api.py
```python
# =====================================================
# 0️⃣ import 구문
# =====================================================
import requests
import pprint  # 딕셔너리를 보기 좋게 출력하기 위한 도구
import pandas as pd  # DataFrame 변환용


# =====================================================
# 1️⃣ API 설정
# =====================================================
API_KEY = "여기에_본인_API_키_문자열_붙이기"
url = f"http://openapi.seoul.go.kr:8088/{API_KEY}/json/airQualityAPI/1/50/"


# =====================================================
# 2️⃣ 서버에 요청 보내기
# =====================================================
response = requests.get(url, timeout=10)

# =====================================================
# 3️⃣ HTTP 상태 코드 확인
# =====================================================
print("HTTP Status Code:", response.status_code)

# =====================================================
# 4️⃣ 서버가 준 원본 텍스트 확인
# =====================================================
print("\n📦 응답 원문 미리보기:")
print(response.text[:300])

# =====================================================
# 5️⃣ 응답 데이터 형식(Content-Type) 확인
# =====================================================
content_type = response.headers.get("Content-Type", "")
print("\n📦 Content-Type:")
print(content_type)


# =====================================================
# ✅ JSON일 때만 파싱하도록 안전장치
# =====================================================
is_json = content_type.lower().startswith("application/json")

if is_json:
    print("\n✅ JSON 형식 확인 → 파이썬 dict로 변환합니다.")

    data = response.json()  # JSON → dict
    print("\n🔍 타입 확인:", type(data))  # dict 확인


    # =================================================
    # ✅ 3단계: JSON 구조 안으로 하나씩 들어가기
    # =================================================
    print("\n--- 🔍 3단계: JSON 구조 해석 시작 ---")

    print("\n1️⃣ 최상위 키:", data.keys())

    if "airQualityAPI" not in data:
        print("\n❌ 'airQualityAPI' 키가 없습니다.")
        print("👉 응답 구조가 예상과 다릅니다.")
        pprint.pprint(data, width=80, depth=3)

    else:
        air = data["airQualityAPI"]
        print("\n2️⃣ air 타입:", type(air))
        print("   air 키:", air.keys())

        if "row" not in air:
            print("\n❌ 'row' 키가 없습니다.")
            print("👉 데이터가 없거나 응답 구조가 다릅니다.")
            pprint.pprint(air, width=80, depth=3)

        else:
            rows = air["row"]
            print("\n3️⃣ row 타입:", type(rows))  # list
            print("   row 길이:", len(rows))

            if len(rows) == 0:
                print("\n⚠️ row 리스트가 비어 있습니다.")
            else:
                print("\n4️⃣ 첫 번째 측정 데이터:")
                pprint.pprint(rows[0], width=80)


            # =================================================
            # ✅ 4단계: pandas DataFrame으로 변환
            # =================================================
            print("\n--- 📊 4단계: DataFrame 변환 시작 ---")

            df = pd.DataFrame(rows)

            print("\n📊 DataFrame.head() 결과:")
            print(df.head())

            print("\n📌 컬럼 목록:")
            print(df.columns.tolist())

            print("\n📌 (선택) 자주 보는 컬럼만 출력 예시:")

            wanted_cols = ["MSRSTE_NM", "PM10", "PM25"]
            existing_cols = [c for c in wanted_cols if c in df.columns]

            if existing_cols:
                print(df[existing_cols].head())
            else:
                print("⚠️ 예시 컬럼(MSRSTE_NM, PM10, PM25)이 현재 응답에 없습니다.")
                print("👉 위의 '컬럼 목록'에서 실제 컬럼명을 확인해 주세요.")


            # =================================================
            # ✅ 5단계: 간단한 필터링 (관제 느낌)
            # =================================================
            print("\n--- ⚠️ 5단계: 기준 이상 지역만 필터링 ---")

            # ✅ [5단계-1] 기준(임계치) 설정: PM10이 이 값 이상이면 "나쁨 이상"이라고 가정
            THRESHOLD_PM10 = 50
            print(f"\n✅ 필터 기준: PM10 >= {THRESHOLD_PM10}")

            # ✅ [5단계-2] 필터링: 조건을 만족하는 행만 남김
            # (df["PM10"] >= 50) 결과는 True/False의 목록이고,
            # df[ ... ] 로 감싸면 True인 행만 남습니다.
            if "PM10" in df.columns:
                bad_air = df[df["PM10"] >= THRESHOLD_PM10]

                # ✅ [5단계-3] 결과 출력: 관제에서는 핵심 컬럼만 뽑아 보여주는 게 일반적
                print("\n⚠️ 미세먼지 나쁨 이상 지역 목록:")

                # 컬럼이 실제로 존재할 때만 안전하게 출력
                cols_to_show = [c for c in ["MSRSTE_NM", "PM10"] if c in df.columns]

                if len(bad_air) == 0:
                    print("✅ 기준 이상 지역이 없습니다. (모두 양호)")
                elif cols_to_show:
                    print(bad_air[cols_to_show].to_string(index=False))
                else:
                    # 만약 컬럼명이 예상과 다르면 전체 컬럼 먼저 확인 유도
                    print("⚠️ 예상 컬럼(MSRSTE_NM, PM10)이 없습니다.")
                    print("👉 df.columns를 확인해서 실제 컬럼명으로 수정하세요.")
            else:
                print("\n❌ 'PM10' 컬럼이 없습니다.")
                print("👉 df.columns에서 실제 미세먼지 컬럼명을 확인해 주세요.")

# =====================================================
# ❌ JSON이 아닐 때
# =====================================================
else:
    print("\n❌ JSON이 아님")
    print("👉 현재 응답은 XML 또는 에러 메시지입니다.")
    print("👉 인증키가 유효한지 확인한 뒤 다시 실행하세요.")
```

실행 방법
```bash
python seoul_air_api.py
```

🖥 출력 결과
```bash
HTTP Status Code: 200

📦 응답 원문 미리보기:
<RESULT><CODE>INFO-100</CODE><MESSAGE><![CDATA[인증키가 유효하지 않습니다.
인증키가 없는 경우, 열린 데이터 광장 홈페이지에서 인증키를 신청하십시오.]]></MESSAGE></RESULT>

📦 Content-Type:
application/xml;charset=UTF-8

❌ JSON이 아님
👉 현재 응답은 XML 또는 에러 메시지입니다.
👉 인증키가 유효한지 확인한 뒤 다시 실행하세요.
(venv) youjung@DESKTOP-PJCRMMU:~/day1_console/day9_api$ python seoul_air_api.py
HTTP Status Code: 200

📦 응답 원문 미리보기:
<RESULT><CODE>INFO-100</CODE><MESSAGE><![CDATA[인증키가 유효하지 않습니다.
인증키가 없는 경우, 열린 데이터 광장 홈페이지에서 인증키를 신청하십시오.]]></MESSAGE></RESULT>

📦 Content-Type:
application/xml;charset=UTF-8

❌ JSON이 아님
👉 현재 응답은 XML 또는 에러 메시지입니다.
👉 인증키가 유효한지 확인한 뒤 다시 실행하세요.
```
이 말은 곧 👇
- 서버 요청 자체는 성공 (HTTP 200)
- ❌ 하지만 데이터 형식이 JSON이 아님
- ❌ 그래서 `response.json()`을 실행하면 안 됨
- ✅ 코드가 이를 정확히 감지해서 3~5단계를 스킵
    
즉,
> “아직 데이터가 준비되지 않았으니, 분석/필터링 단계로 들어가지 않는다”

라는 판단을 코드가 정확히 한 것입니다.
👉 이건 매우 잘 짠 코드의 증거입니다.

---
왜 5단계까지 안 나오는데도 ‘맞다’고 할 수 있나?

지금 코드 흐름은:
```
요청 →
Content-Type 확인 →
JSON일 때만
    ├─ 3단계(JSON 구조 해석)
    ├─ 4단계(DataFrame 변환)
    └─ 5단계(필터링)
```

그리고 실제 결과는:
```
Content-Type = application/xml
→ JSON 아님
→ 3~5단계 실행 안 함
```
👉 설계한 대로 정확히 동작

---

✅ 5단계 요약
- 필터링은 관제의 핵심이다.
    
- DataFrame에서
```python
df[df["PM10"] >= 기준값]
```
    이 한 줄로 위험 목록을 만들 수 있다.
    
- 이제 이걸 Streamlit UI로 연결하면 “관제 대시보드”가 됩니다.

---
## 5단계 마무리: Streamlit 화면으로 관제 대시보드 만들기

🎯 목표
- 콘솔 출력으로 끝내지 않고,
- 브라우저 화면에서
    - 전체 데이터 표
    - 임계치 슬라이더(기준값 조절)
    - “나쁨 이상” 필터 결과
    - 간단 차트  
        를 직접 보게 해서 **관제 시스템이 ‘완성’된 느낌**을 만든다.

준비
```bash
cd ~/day1_console/day9_api
pip install streamlit pandas requests
```
브라우저를 열수있는 스트림릿을 설치합니다.

VSCode에서 app.py파일을 생성합니다.
app.py
```python
import requests
import pandas as pd
import streamlit as st

# ==============================
# 0) Streamlit 기본 설정
# ==============================
st.set_page_config(page_title="서울 미세먼지 관제 대시보드", page_icon="🌫️", layout="wide")
st.title("🌫️ 서울 미세먼지 관제 대시보드 (API → DataFrame → 필터링)")
st.write("API로 데이터를 가져와서 **기준값(임계치)** 으로 위험 지역만 걸러서 보여주는 관제 화면입니다.")

# ==============================
# 1) 사이드바: 입력(설정값)
# ==============================
st.sidebar.header("⚙️ 설정")

API_KEY = st.sidebar.text_input("서울 OpenAPI KEY", value="여기에_본인_API_키_문자열_붙이기")
limit = st.sidebar.selectbox("가져올 데이터 개수", [5, 10, 20, 50], index=3)

threshold_pm10 = st.sidebar.slider("PM10 임계치 (나쁨 기준)", min_value=0, max_value=200, value=50, step=5)
only_bad = st.sidebar.checkbox("나쁨 이상만 보기", value=True)

refresh = st.sidebar.button("🔄 새로고침(재요청)")

# ==============================
# 2) API 호출 함수
# ==============================
@st.cache_data(ttl=60)  # 60초 동안 캐시
def fetch_air_data(api_key: str, limit: int):
    url = f"http://openapi.seoul.go.kr:8088/{api_key}/json/airQualityAPI/1/{limit}/"
    response = requests.get(url, timeout=10)

    content_type = response.headers.get("Content-Type", "")
    is_json = content_type.lower().startswith("application/json")

    # JSON이 아니면(XML/에러 메시지)
    if not is_json:
        return None, response.text[:500]

    data = response.json()

    if "airQualityAPI" not in data:
        return None, "응답에 airQualityAPI 키가 없습니다."

    if "row" not in data["airQualityAPI"]:
        return None, "응답에 row 데이터가 없습니다."

    rows = data["airQualityAPI"]["row"]
    df = pd.DataFrame(rows)

    return df, None


# ==============================
# 3) 데이터 가져오기
# ==============================
# 새로고침 버튼을 누르면 캐시 무시하고 다시 불러오게 처리
if refresh:
    fetch_air_data.clear()

df, err = fetch_air_data(API_KEY, limit)

if df is None:
    st.error("❌ DataFrame 생성에 실패했습니다. (df가 None)")
    if err:
        st.code(err)
    st.stop()

# ==============================
# 4) 에러 처리
# ==============================
if err is not None:
    st.error("❌ 데이터를 가져오지 못했습니다.")
    st.write("아래 메시지를 확인하세요 (대부분 인증키 문제로 XML이 내려옵니다).")
    st.code(err)
    st.stop()

# ==============================
# 5) 컬럼 확인 및 타입 정리
# ==============================
# 혹시 숫자가 문자열로 오면 비교가 안 되므로 숫자 변환
if "PM10" in df.columns:
    df["PM10"] = pd.to_numeric(df["PM10"], errors="coerce")

if "PM25" in df.columns:
    df["PM25"] = pd.to_numeric(df["PM25"], errors="coerce")

# ==============================
# 6) 필터링(관제 핵심)
# ==============================
if "PM10" not in df.columns:
    st.warning("⚠️ PM10 컬럼이 없습니다. df.columns를 확인해 주세요.")
    st.write(df.columns.tolist())
    st.stop()

bad_df = df[df["PM10"] >= threshold_pm10].copy()

# ==============================
# 7) 화면 출력(관제 마무리)
# ==============================
col1, col2 = st.columns([2, 1])

with col1:
    st.subheader("📋 전체 데이터")
    st.caption("API에서 받은 원본 데이터(DataFrame)입니다.")
    st.dataframe(df, use_container_width=True)

with col2:
    st.subheader("🚨 위험 요약")
    st.metric("총 데이터 수", len(df))
    st.metric(f"PM10 ≥ {threshold_pm10} 지역 수", len(bad_df))

    st.subheader("📌 컬럼 확인")
    st.write(df.columns.tolist())

st.divider()

st.subheader("⚠️ 나쁨 이상(필터링 결과)")
st.caption("관제 화면은 보통 **기준 이상 데이터만** 보여줍니다.")

show_cols = [c for c in ["MSRSTE_NM", "PM10", "PM25", "MSRDATE"] if c in df.columns]

if only_bad:
    if len(bad_df) == 0:
        st.success("✅ 기준 이상 지역이 없습니다. (모두 양호)")
    else:
        st.dataframe(bad_df[show_cols], use_container_width=True)
else:
    st.info("체크 해제 상태: 전체 데이터가 이미 위에 표시되어 있습니다.")

st.divider()

st.subheader("📈 간단 차트: PM10 상위 지역 TOP 10")
if len(df) > 0:
    top = df.sort_values("PM10", ascending=False).head(10)
    chart_cols = [c for c in ["MSRSTE_NM", "PM10"] if c in top.columns]

    if chart_cols == ["MSRSTE_NM", "PM10"]:
        top_chart = top.set_index("MSRSTE_NM")["PM10"]
        st.bar_chart(top_chart)
    else:
        st.write("차트에 필요한 컬럼이 부족합니다. (MSRSTE_NM, PM10)")
```

실행
```bash
streamlit run app.py
```

![[Pasted image 20251220201047.png]]
빨간 에러 박스의 의미
```
❌ 데이터를 가져오지 못했습니다.
```
그리고 아래 메시지:
```xml
<RESULT>
  <CODE>INFO-100</CODE>
  <MESSAGE>
    인증키가 유효하지 않습니다.
    인증키가 없는 경우, 열린 데이터 광장 홈페이지에서 인증키를 신청하십시오.
  </MESSAGE>
</RESULT>
```
이건 ❌ 프로그램 오류가 아닙니다.

이게 뜻하는 정확한 의미
- 서울 OpenAPI 서버가 요청을 정상적으로 받았고
- 대신 이렇게 말해준 것:
	“너가 준 API 키가 아직 유효하지 않거나, 잘못됐어”
    
그래서:
- JSON ❌
- XML(에러 메시지) ⭕ 를 내려준 겁니다.
    
👉 이건 우리가 일부러 만든 “방어 코드”가 정확히 작동했다는 증거예요.

- 화면이 안 뜨는 건 ❌ 개발 실패
- 데이터가 안 오는 이유를 화면에서 설명해주는 것 ⭕
- 이것이 바로 운영 가능한 관제 시스템

관제 대시보드는 완벽하게 만들어졌고, 현재 보이는 메시지는 외부 API 인증 실패를 정상적으로 알려주는 ‘정상 동작 화면’입니다.

---
# 6단계 – CSV 업로드를 관제 대시보드에 “자연스럽게” 붙이기 (API 실패 대비/리플레이)

✅ 왜 갑자기 CSV를 하냐? (관제 관점에서의 정의)

5단계까지 우리가 만든 관제 대시보드는 이런 구조였죠.
> API(외부 데이터) → requests.get() → JSON → rows(list) → DataFrame(df) → 필터링(임계치 이상만) → 화면 출력

그런데 지금 실제로 겪고 있는 문제가 있어요:
- 인증키가 틀리거나 만료됨
- 외부 API 서버 장애
- 네트워크 문제
- 서버에서 XML 에러를 내려줌

즉, “관제 시스템은 외부 데이터가 항상 정상이라고 가정하면 안 됩니다.”

그래서 실무 관제에서는 거의 무조건 아래가 같이 들어가요:
✅ 관제에서 CSV는 “옵션 기능”이 아니라 “운영 안전장치”

CSV를 쓰는 이유는 3가지로 정리됩니다.
1. API가 죽어도 화면은 살아있어야 함 (Fallback)
- 외부 API가 안 되면 CSV로라도 대체해서 화면과 기능을 유지
- “시스템이 죽었다”가 아니라
    “현재 실시간 데이터가 안 와서 대체 모드로 동작 중” 상태를 보여줄 수 있음
    
2. 테스트/수업용 데이터 재생(Replay)
- API가 매번 바뀌면 테스트가 어렵습니다.
- CSV는 “고정된 샘플 데이터”라서
    - 필터링 실습
    - 차트 실습
    - UI 실습을 안정적으로 할 수 있음
    
3. 운영 기록(로그) / 배치 데이터 분석
- 관제 시스템은 보통 “실시간”만 보는 게 아니라
    - 어제 데이터
    - 일주일 평균
    - 특정 지역의 추이 같은 걸 봐야 합니다.
- 이런 건 API보다 CSV/DB에 저장된 데이터가 더 유리한 경우가 많음
    
---

🎯 6단계 목표

이 단계의 목표는 딱 4가지입니다.
1. 같은 관제 화면에서 데이터 소스를 선택한다
    - `API(실시간)` 또는 `CSV(업로드)`
2. CSV 파일을 업로드하면 **DataFrame(df)** 으로 만들어 표로 보여준다
3. 5단계에서 했던 것처럼 **임계치 기반 필터링**을 똑같이 적용한다
4. 가능하면 **간단한 차트(추이)** 도 뽑아본다

즉,
> 데이터가 API든 CSV든,
> DataFrame만 만들면 관제 필터/표/차트는 똑같이 동작한다
> 이걸 몸으로 이해시키는 게 핵심입니다.

---

✅ 실습 준비
 `0)` 현재 폴더 구조 가정
	지금 관제 앱을 실행하던 폴더 `day9_api/` 안에 `app.py` 파일을 생성합니다. 

 `1)` 같은 폴더에 CSV 샘플 파일 만들기 (관제 데이터 “리플레이” 용)

터미널에서 현재 폴더에 파일 생성:
```bash
code dust_sample.csv
```

`dust_sample.csv` 내용 그대로 붙여넣기:
```
date,station,pm10,pm2_5
2025-01-01,강남구,45,22
2025-01-01,송파구,55,30
2025-01-01,강북구,32,18
2025-01-02,강남구,60,35
2025-01-02,송파구,70,40
2025-01-02,강북구,50,28
```

> 이 파일은 “API가 정상일 때의 row(list)”를 흉내 낸 데이터라고 생각하면 됩니다.
> (station = 구 이름, pm10 = 미세먼지, date = 날짜)

---
그대로 app.py에 통째로 복붙하세요.
```python
import streamlit as st
import requests
import pandas as pd


# =====================================================
# 0) 기본 페이지 설정
# =====================================================
st.set_page_config(
    page_title="서울 미세먼지 관제 대시보드",
    page_icon="🌫️",
    layout="wide"
)

st.title("🌫️ 서울 미세먼지 관제 대시보드 (API / CSV → DataFrame → 필터링)")
st.write("데이터를 가져와서 기준값(임계치)으로 위험 지역만 걸러서 보여주는 관제 화면입니다.")


# =====================================================
# 1) 사이드바: 관제 설정 UI
# =====================================================
st.sidebar.header("⚙️ 설정")

data_source = st.sidebar.radio(
    "데이터 소스 선택",
    ["API(실시간)", "CSV 업로드(브라우저)", "로컬 CSV(dust_sample.csv)"],
    index=0
)

api_key = st.sidebar.text_input(
    "서울 OpenAPI KEY (API 모드에서만 필요)",
    value="",
    placeholder="여기에 본인 API KEY 입력"
)

limit = st.sidebar.selectbox("가져올 데이터 개수(API 모드)", [5, 10, 20, 50], index=3)

threshold_pm10 = st.sidebar.slider(
    "PM10 임계치 (나쁨 기준)",
    min_value=0,
    max_value=150,
    value=50,
    step=5
)

only_bad = st.sidebar.checkbox("나쁨 이상만 보기", value=True)

st.sidebar.divider()

# CSV 업로드는 해당 모드에서만
uploaded_file = None
if data_source == "CSV 업로드(브라우저)":
    uploaded_file = st.sidebar.file_uploader("CSV 파일 업로드", type=["csv"])

refresh = st.sidebar.button("🔄 새로고침(재요청)")


# =====================================================
# 2) 데이터 로딩 함수
# =====================================================

# ✅ [권장] API 요청도 캐시하면 실무/수업에서 과도한 호출 방지에 좋음
@st.cache_data(ttl=60)
def load_from_api(api_key: str, limit: int) -> tuple[pd.DataFrame | None, str | None]:
    if not api_key.strip():
        return None, "API KEY가 비어 있습니다. 왼쪽 사이드바에 KEY를 입력하세요."

    url = f"http://openapi.seoul.go.kr:8088/{api_key}/json/airQualityAPI/1/{limit}/"

    try:
        response = requests.get(url, timeout=10)
    except Exception as e:
        return None, f"요청 실패(네트워크/timeout 가능): {e}"

    # ✅ [권장] HTTP 코드가 200이 아닐 때는 바로 안내
    if response.status_code != 200:
        return None, (
            f"HTTP 요청 실패 (status={response.status_code})\n\n"
            f"응답 미리보기:\n{response.text[:500]}"
        )

    content_type = response.headers.get("Content-Type", "")

    # ✅ [수정 - 필수] Content-Type 판별을 더 안전하게 (json; charset=utf-8 포함)
    is_json = content_type.lower().startswith("application/json")

    # 서울 OpenAPI는 인증/에러 상황에서 XML로 내려오는 경우가 많음
    if not is_json:
        preview = response.text[:500]
        return None, (
            f"JSON이 아닙니다. (Content-Type: {content_type})\n\n"
            f"응답 미리보기:\n{preview}"
        )

    try:
        data = response.json()
    except Exception as e:
        return None, f"JSON 파싱 실패: {e}"

    if "airQualityAPI" not in data or "row" not in data["airQualityAPI"]:
        return None, f"응답 구조가 예상과 다릅니다.\n\n키 목록: {list(data.keys())}"

    rows = data["airQualityAPI"]["row"]
    df = pd.DataFrame(rows)
    return df, None


def load_from_uploaded_csv(uploaded_file) -> tuple[pd.DataFrame | None, str | None]:
    if uploaded_file is None:
        return None, "CSV 파일이 업로드되지 않았습니다. 왼쪽에서 파일을 선택하세요."

    try:
        df = pd.read_csv(uploaded_file)
        return df, None
    except Exception as e:
        return None, f"CSV 읽기 실패: {e}"


def load_from_local_csv(path: str = "dust_sample.csv") -> tuple[pd.DataFrame | None, str | None]:
    try:
        df = pd.read_csv(path)
        return df, None
    except FileNotFoundError:
        return None, (
            f"로컬 CSV 파일을 찾지 못했습니다: {path}\n"
            f"👉 app.py와 같은 폴더에 {path}가 있는지 확인하세요."
        )
    except Exception as e:
        return None, f"로컬 CSV 읽기 실패: {e}"


# =====================================================
# 3) 데이터 로딩
# =====================================================
st.subheader("✅ 1) 데이터 로딩 결과")

# ✅ [권장] 새로고침 버튼 누르면 캐시 클리어
if refresh:
    load_from_api.clear()

if data_source == "API(실시간)":
    df, err = load_from_api(api_key, limit)
elif data_source == "CSV 업로드(브라우저)":
    df, err = load_from_uploaded_csv(uploaded_file)
else:
    # 로컬 CSV
    df, err = load_from_local_csv("dust_sample.csv")

if err is not None:
    st.error("❌ 데이터를 가져오지 못했습니다.")
    st.write("아래 메시지를 확인하세요 (실무 관제에서도 이런 안내가 매우 중요합니다).")
    st.code(err)
    st.stop()

# ✅ [권장] df가 None인 케이스 방어(예외적 상황 대비)
if df is None:
    st.error("❌ DataFrame 생성 실패(df=None).")
    st.stop()

st.success("✅ 데이터 로딩 성공 (DataFrame 생성 완료)")


# =====================================================
# 4) DataFrame 기본 확인
# =====================================================
st.subheader("✅ 2) DataFrame 미리보기 / 컬럼 확인")

col1, col2 = st.columns([2, 1])

with col1:
    st.write("📊 상위 5줄(df.head())")
    st.dataframe(df.head(), use_container_width=True)

with col2:
    st.write("📌 컬럼 목록")
    st.write(list(df.columns))


# =====================================================
# 5) 임계치 기반 필터링
# =====================================================
st.subheader("✅ 3) 관제 필터링 (임계치 이상만 보기)")

# API / CSV 컬럼 차이를 흡수
pm10_candidates = ["PM10", "pm10"]
name_candidates = ["MSRSTE_NM", "station"]

pm10_col = next((c for c in pm10_candidates if c in df.columns), None)
name_col = next((c for c in name_candidates if c in df.columns), None)

if pm10_col is None:
    st.warning("⚠️ PM10 컬럼을 찾지 못했습니다. df.columns를 확인해서 컬럼명을 맞춰주세요.")
    st.stop()

# 숫자 변환
df[pm10_col] = pd.to_numeric(df[pm10_col], errors="coerce")

# 필터링
if only_bad:
    filtered = df[df[pm10_col] >= threshold_pm10].copy()
else:
    filtered = df.copy()

st.write(f"- 사용 컬럼: **{pm10_col}**")
st.write(f"- 필터 기준: **{pm10_col} >= {threshold_pm10}**")
st.write(f"- 결과 행 개수: **{len(filtered)}개**")

show_cols = []
if name_col is not None:
    show_cols.append(name_col)
show_cols.append(pm10_col)

if len(filtered) == 0:
    st.info("✅ 기준 이상 지역이 없습니다. (모두 양호)")
else:
    st.dataframe(
        filtered[show_cols].sort_values(by=pm10_col, ascending=False),
        use_container_width=True
    )


# =====================================================
# 6) 차트로 보기 (관제 마무리)
# =====================================================
st.subheader("✅ 4) 차트로 보기 (관제 마무리: TOP 10)")

if len(filtered) == 0:
    st.info("표시할 데이터가 없습니다. 임계치를 낮추거나 '나쁨 이상만 보기'를 해제해보세요.")
else:
    chart_df = filtered.copy()

    if name_col is not None:
        chart_df = chart_df[[name_col, pm10_col]].dropna()
        chart_df = chart_df.sort_values(by=pm10_col, ascending=False).head(10)
        chart_df = chart_df.set_index(name_col)
        st.bar_chart(chart_df[[pm10_col]])
    else:
        chart_df = chart_df[[pm10_col]].dropna().sort_values(by=pm10_col, ascending=False).head(10)
        st.bar_chart(chart_df[[pm10_col]])

st.caption("✅ 정리: 데이터가 API든 CSV든, DataFrame만 만들면 필터링/표/차트는 동일하게 동작합니다.")
```

실행
```bash
streamlit run app.py
```

브라우저: `http://localhost:8501`
![[Pasted image 20251220204410.png]]

1️⃣ 데이터 로딩 결과 해석

상단에 초록색 메시지:
> ✅ 데이터 로딩 성공 (DataFrame 생성 완료)

의미
- CSV(`dust_sample.csv`)가 **정상적으로 읽혔고**
- Pandas `DataFrame`으로 **문제 없이 변환 완료**
- 관제 시스템 입장에서 보면  
    👉 **“센서 데이터 수신 성공”** 상태입니다.
    
이 시점까지가 관제 시스템의 가장 기본적인 생존 조건이에요.

---

2️⃣ DataFrame 미리보기 / 컬럼 확인

보이는 데이터
상위 5줄:

|date|station|pm10|pm2_5|
|---|---|---|---|
|2025-01-01|강남구|45|22|
|2025-01-01|송파구|55|30|
|2025-01-01|강북구|32|18|
|2025-01-02|강남구|60|35|
|2025-01-02|송파구|70|40|

컬럼 목록
`["date", "station", "pm10", "pm2_5"]`

의미
- ✔ 관제에 필요한 최소 컬럼이 모두 있음
    - `station` → 관측 위치 (관제 대상)
    - `pm10` → 판단 기준 수치
- ✔ 숫자 컬럼(`pm10`)도 정상 인식됨
    
👉 “이 데이터로 관제 판단 가능” 상태입니다.

---
3️⃣ 관제 필터링 결과 해석

 설정 상태
- PM10 임계치: **50**
- 옵션: **나쁨 이상만 보기 체크**
- 사용 컬럼: `pm10`
    

필터 조건
`pm10 >= 50`

결과 요약
- 결과 행 개수: **4개**
    
|station|pm10|
|---|---|
|송파구|70|
|강남구|60|
|송파구|55|
|강북구|50|

관제 관점 해석
- 총 데이터 중 **4건이 ‘나쁨 이상’ 상태**
- 특히:
    - **송파구**: 70, 55 → 반복적으로 높음
    - **강남구**: 60 → 주의 필요
    - **강북구**: 50 → 임계치 경계
        
👉 이 화면을 본 관제 담당자는 이렇게 판단합니다:

> 송파구는 지속 감시 대상, 강남구는 단기 경보 검토, 강북구는 추세 관찰 필요

이게 바로 관제 시스템의 역할이에요.  
모든 데이터를 보는 게 아니라 “문제가 되는 것만 걸러서 보여주는 것”.

---
4️⃣ 차트 (관제 마무리: TOP 10) 해석

차트 의미
- x축: 관측 지점(station)
- y축: PM10 수치
- 값이 클수록 위험
    
해석 포인트
- 가장 높은 막대 → **송파구**
- 그 다음 → 강남구, 강북구
- 숫자 크기 차이가 **한눈에 비교 가능**
    
👉 관제에서 차트의 목적은:
- 정확한 수치 ❌
- **“어디가 제일 위험한지 한눈에 파악” ⭕**
    
---
전체 결과를 한 문장으로 정리하면
> “서울 미세먼지 데이터 중 PM10 기준 50 이상인 지역만 선별하여,  
> 현재 송파구·강남구·강북구가 관제 대상이며  
> 송파구가 가장 위험도가 높다.”

---
## 7단계(필터/정렬)

실무 관제는 항상 2단계로 움직입니다.
1. **시스템 관제(자동)**
- “PM10 임계치 이상만” 자동으로 걸러서 보여줌
- 즉, **이상치 감지**

2. 사람 관제(탐색/검증) ← 오늘 추가하는 파트
- “강남구만 보면?”
- “가장 심한 순서로 정렬하면?”
- “기준을 50→70으로 바꾸면 결과가 어떻게 달라지지?”
    즉, **원인 파악/우선순위 결정/보고서 작성** 단계
    
---

개념 정리
- **필터링(filter)**: “조건에 맞는 행만 남기기”
    - 예: `df[df["station"] == "강남구"]`
- **정렬(sort)**: “남아있는 행의 순서를 바꾸기”
    - 예: `df.sort_values("pm10", ascending=False)`
- **관제에서의 의미**
    - 필터링 = “어떤 대상을 볼지 고르기”
    - 정렬 = “우선순위를 정해서 보기”

---

전체 app.py
```python
import streamlit as st
import requests
import pandas as pd

# =====================================================
# ✅ 0) Streamlit 기본 페이지 설정
# =====================================================
st.set_page_config(
    page_title="서울 미세먼지 관제 대시보드",
    page_icon="🌫️",
    layout="wide",
)

st.title("🌫️ 서울 미세먼지 관제 대시보드 (API / CSV → DataFrame → 관제 + 탐색)")
st.write(
    "데이터를 가져와서 DataFrame으로 만들고, 임계치 기준으로 위험 지역을 걸러본 뒤 "
    "필터/정렬로 관제 담당자처럼 탐색합니다."
)

# =====================================================
# ✅ 1) 사이드바: 데이터 소스 선택 + 관제 설정
# =====================================================
st.sidebar.header("⚙️ 설정")

data_source = st.sidebar.radio(
    "데이터 소스 선택",
    ["API(실시간)", "CSV 업로드(파일)", "CSV 로컬(폴더)"],
    index=1,
)

api_key = st.sidebar.text_input(
    "서울 OpenAPI KEY (API 모드에서만 사용)",
    value="",
    placeholder="여기에 본인 API KEY 입력",
)

limit = st.sidebar.selectbox("가져올 데이터 개수 (API 모드)", [5, 10, 20, 50], index=3)

threshold_pm10 = st.sidebar.slider(
    "PM10 임계치 (나쁨 기준)",
    min_value=0,
    max_value=150,
    value=50,
    step=5,
)

only_bad = st.sidebar.checkbox("나쁨 이상만 보기(PM10 임계치 이상)", value=True)

refresh = st.sidebar.button("🔄 새로고침(재요청)")

# =====================================================
# ✅ 2) 데이터 로딩 함수들
# =====================================================

# ✅ [추가] API 호출은 캐시 적용(과도한 요청 방지)
@st.cache_data(ttl=60)
def load_from_api(api_key: str, limit: int):
    """
    API에서 JSON을 가져와 DataFrame으로 변환합니다.
    반환: (df, err)
      - 성공: (DataFrame, None)
      - 실패: (None, 에러메시지 str)
    """
    if not api_key.strip():
        return None, "API KEY가 비어 있습니다. 왼쪽 사이드바에 KEY를 입력하세요."

    url = f"http://openapi.seoul.go.kr:8088/{api_key}/json/airQualityAPI/1/{limit}/"

    try:
        response = requests.get(url, timeout=10)
    except Exception as e:
        return None, f"요청 실패(네트워크/timeout 가능): {e}"

    # ✅ [추가] HTTP 상태코드 방어(200이 아니면 에러 처리)
    if response.status_code != 200:
        return None, (
            f"HTTP 요청 실패 (status={response.status_code})\n\n"
            f"응답 미리보기:\n{response.text[:800]}"
        )

    content_type = response.headers.get("Content-Type", "")

    # ✅ [수정] Content-Type 판별을 이전 단계와 동일하게(더 안전)
    is_json = content_type.lower().startswith("application/json")
    if not is_json:
        preview = response.text[:800]
        return None, (
            f"JSON이 아닙니다. (Content-Type: {content_type})\n\n"
            f"응답 미리보기(앞부분):\n{preview}"
        )

    try:
        data = response.json()
    except Exception as e:
        return None, f"JSON 파싱 실패: {e}"

    if "airQualityAPI" not in data or "row" not in data["airQualityAPI"]:
        return None, f"응답 구조가 예상과 다릅니다.\n키 목록: {list(data.keys())}"

    rows = data["airQualityAPI"]["row"]
    df = pd.DataFrame(rows)
    return df, None


def load_from_csv_upload(uploaded_file):
    if uploaded_file is None:
        return None, "CSV 파일이 업로드되지 않았습니다. 왼쪽에서 파일을 선택하세요."

    try:
        df = pd.read_csv(uploaded_file)
        return df, None
    except Exception as e:
        return None, f"CSV 읽기 실패: {e}"


def load_from_csv_local(filename: str = "dust_sample.csv"):
    try:
        df = pd.read_csv(filename)
        return df, None
    except FileNotFoundError:
        return None, f"'{filename}' 파일을 찾을 수 없습니다. app.py와 같은 폴더에 두세요."
    except Exception as e:
        return None, f"로컬 CSV 읽기 실패: {e}"


# =====================================================
# ✅ 3) 데이터 로딩 (API / CSV)
# =====================================================
st.subheader("✅ 1) 데이터 로딩 결과")

# ✅ [추가] refresh 버튼 클릭 시 캐시 삭제 + 재실행
# (API 모드일 때 의미가 큼)
if refresh:
    load_from_api.clear()
    st.rerun()

uploaded_file = None
if data_source == "CSV 업로드(파일)":
    uploaded_file = st.sidebar.file_uploader("CSV 파일 업로드", type=["csv"])

if data_source == "API(실시간)":
    df, err = load_from_api(api_key, limit)
elif data_source == "CSV 업로드(파일)":
    df, err = load_from_csv_upload(uploaded_file)
else:
    df, err = load_from_csv_local("dust_sample.csv")

if err is not None:
    st.error("❌ 데이터를 가져오지 못했습니다.")
    st.write("아래 메시지를 확인하세요. (실무 관제에서도 이런 안내가 매우 중요합니다.)")
    st.code(err)
    st.stop()

# ✅ [추가] df None 방어 (혹시 모를 예외 케이스)
if df is None:
    st.error("❌ DataFrame 생성 실패(df=None).")
    st.stop()

st.success("✅ 데이터 로딩 성공 (DataFrame 생성 완료)")

# =====================================================
# ✅ 4) DataFrame 기본 확인
# =====================================================
st.subheader("✅ 2) DataFrame 미리보기 / 컬럼 확인")

col1, col2 = st.columns([2, 1])

with col1:
    st.write("📊 상위 5줄(df.head())")
    # ✅ [수정] width="stretch" 제거 → Streamlit 표준 use_container_width 사용
    st.dataframe(df.head(), use_container_width=True)

with col2:
    st.write("📌 컬럼 목록")
    st.write(list(df.columns))

# =====================================================
# ✅ 5) 관제 핵심: 임계치 기반 필터링
# =====================================================
st.subheader("✅ 3) 관제 필터링 (임계치 이상만 보기)")

pm10_candidates = ["PM10", "pm10"]
station_candidates = ["MSRSTE_NM", "station"]

pm10_col = next((c for c in pm10_candidates if c in df.columns), None)
station_col = next((c for c in station_candidates if c in df.columns), None)

if pm10_col is None:
    st.warning("⚠️ PM10 컬럼을 찾지 못했습니다. df.columns를 확인해서 pm10 컬럼명을 맞춰주세요.")
    st.stop()

df[pm10_col] = pd.to_numeric(df[pm10_col], errors="coerce")

if only_bad:
    filtered = df[df[pm10_col] >= threshold_pm10].copy()
else:
    filtered = df.copy()

st.write(f"- 사용 컬럼: **{pm10_col}**")
st.write(f"- 필터 기준: **{pm10_col} >= {threshold_pm10}**")
st.write(f"- 결과 행 개수: **{len(filtered)}개**")

show_cols = []
if station_col is not None:
    show_cols.append(station_col)
show_cols.append(pm10_col)

if len(filtered) == 0:
    st.info("✅ 기준 이상 지역이 없습니다. (모두 양호)")
else:
    st.dataframe(
        filtered[show_cols].sort_values(by=pm10_col, ascending=False),
        use_container_width=True,  # ✅ [수정]
    )

# =====================================================
# ✅ 6) 관제 마무리: 간단 차트 TOP N
# =====================================================
st.subheader("✅ 4) 차트로 보기 (관제 마무리: TOP 10)")

if len(filtered) == 0:
    st.info("표시할 데이터가 없습니다. 임계치를 낮추거나 '나쁨 이상만 보기'를 해제해보세요.")
else:
    chart_df = filtered.copy()

    if station_col is not None:
        chart_df = chart_df[[station_col, pm10_col]].dropna()
        chart_df = chart_df.sort_values(by=pm10_col, ascending=False).head(10)
        chart_df = chart_df.set_index(station_col)
        st.bar_chart(chart_df[[pm10_col]])
    else:
        chart_df = chart_df[[pm10_col]].dropna().sort_values(by=pm10_col, ascending=False).head(10)
        st.bar_chart(chart_df[[pm10_col]])

st.caption("✅ 정리: 데이터가 API든 CSV든, DataFrame만 만들면 필터링/표/차트는 동일하게 동작합니다.")

# =====================================================================
# ✅ 7) 관제 탐색: 필터링/정렬 실습 파트
# =====================================================================
st.divider()
st.header("🔍 5) 관제 데이터 탐색 (필터링 / 정렬)")

st.write(
    """
**왜 이 단계가 필요한가?**

- 앞에서는 시스템이 임계치로 **자동 감지(관제)** 를 했습니다.
- 이제는 관제 담당자처럼 **직접 질문하며 확인(탐색/검증)** 합니다.

✅ 필터링 = “조건에 맞는 행만 남긴다”  
✅ 정렬 = “남은 행을 위험도/우선순위 순서로 본다”
"""
)

explore_df = df.copy()

# -------------------------------------------------
# 7-1) 지역 선택 필터링
# -------------------------------------------------
if station_col is None:
    st.warning("⚠️ 지역(station) 컬럼이 없어 지역 기반 탐색(필터)이 제한됩니다. CSV 컬럼명을 확인하세요.")
else:
    st.subheader("① 선택한 지역만 보기 (필터링)")

    stations = sorted(explore_df[station_col].dropna().unique())
    selected_station = st.selectbox("지역 선택", stations)

    station_only = explore_df[explore_df[station_col] == selected_station].copy()
    st.write(f"✅ **{selected_station}** 데이터만 표시합니다.")
    st.dataframe(station_only, use_container_width=True)  # ✅ [수정]

# -------------------------------------------------
# 7-2) PM10 기준 이상 필터링(탐색용)
# -------------------------------------------------
st.subheader("② PM10 기준값을 바꿔가며 보기 (필터링)")

pm10_series = explore_df[pm10_col].dropna()
min_pm10 = int(pm10_series.min()) if len(pm10_series) else 0
max_pm10 = int(pm10_series.max()) if len(pm10_series) else 150

threshold_test = st.slider(
    "PM10 기준 값 선택(탐색용)",
    min_value=min_pm10,
    max_value=max_pm10,
    value=min(50, max_pm10),
)

high_pm10 = explore_df[explore_df[pm10_col] >= threshold_test].copy()
st.write(f"✅ PM10이 **{threshold_test} 이상**인 행만 표시합니다.")
st.dataframe(high_pm10, use_container_width=True)  # ✅ [수정]

# -------------------------------------------------
# 7-3) 정렬: 위험도 순으로 보기
# -------------------------------------------------
st.subheader("③ 위험도 순서로 정렬 (Sort)")

sort_order = st.radio(
    "정렬 순서 선택",
    ("PM10 높은 순", "PM10 낮은 순"),
    horizontal=True,
)

if sort_order == "PM10 높은 순":
    sorted_df = explore_df.sort_values(pm10_col, ascending=False)
else:
    sorted_df = explore_df.sort_values(pm10_col, ascending=True)

st.dataframe(sorted_df, use_container_width=True)  # ✅ [수정]

# -------------------------------------------------
# 7-4) 지역 + 정렬 조합 (필터 + 정렬)
# -------------------------------------------------
st.subheader("④ 지역 + 정렬 조합 보기 (필터 + 정렬)")

if station_col is None:
    st.info("지역(station) 컬럼이 없어서 조합 기능을 사용할 수 없습니다.")
else:
    c1, c2 = st.columns(2)

    with c1:
        station_for_combo = st.selectbox("지역 선택(조합용)", stations, key="combo_station")

    with c2:
        order_for_combo = st.radio(
            "정렬 순서(조합용)",
            ("PM10 높은 순", "PM10 낮은 순"),
            key="combo_order",
            horizontal=True,
        )

    combo_filtered = explore_df[explore_df[station_col] == station_for_combo].copy()

    if order_for_combo == "PM10 높은 순":
        combo_result = combo_filtered.sort_values(pm10_col, ascending=False)
    else:
        combo_result = combo_filtered.sort_values(pm10_col, ascending=True)

    st.write(f"✅ **{station_for_combo}** / **{order_for_combo}** 결과")
    st.dataframe(combo_result, use_container_width=True)  # ✅ [수정]

st.caption("✅ 관제 팁: 필터(대상 선택) → 정렬(우선순위) 순서로 보면, 실무 관제처럼 판단이 쉬워집니다.")
```

실행
```bash
streamlit run app.py
```

![[Pasted image 20251220211105.png]]

---

## 관제 대시보드 화면 구성

🎯 목표 (한 문장)
> 외부 데이터(API 또는 CSV)를 받아 DataFrame으로 만들고,  
> 임계치 기준으로 위험만 선별해 사람이 판단할 수 있는 관제 대시보드 화면을 완성한다.

---
✅ 이 단계에서 완성의 기준
	여기서 말하는 완성은 기능을 많이 넣는 것이 아닙니다.  

다음 질문에 모두 “YES” 라고 답할 수 있으면 완성입니다.
- ❓ 데이터가 API든 CSV든 같은 화면에서 처리되는가
- ❓ 정상 데이터 / 이상 데이터가 구분되어 보이는가
- ❓ API가 실패해도 화면이 깨지지 않고 이유를 설명하는가
- ❓ 사람이 보고 판단할 수 있는 정보만 강조되어 있는가
    
이것이 실무에서 말하는 운영 가능한 관제 화면의 최소 조건입니다.

---
### 🧭 7단계까지 우리가 이미 만든 것

현재 코드는 이미 아래를 완성했습니다.
- API / CSV 로딩 → DataFrame
- 임계치 필터링(only_bad)
- 위험 목록 표 출력
- TOP10 차트
- 탐색(필터/정렬) 기능까지 구현
    
✅ 즉, “데이터 관제 + 탐색”의 핵심 구조까지 작성되었습니다.

---

### 8단계 – 관제 화면답게 만들기

7단계까지는 데이터를 보여주는 앱이라면,  
8단계부터는 관제 화면(운영 가능)이 되기 위해 아래 2가지를 보완합니다.

1. 상단 상태 요약(메트릭) 추가: 관제 담당자가 지금 상황을 바로 판단
2. 에러 안내/대체 흐름 강화: API 실패해도 화면이 설명 가능한 상태로 유지

---
### 관제 화면의 역할을 코드로 드러내기

❌ 관제 화면이 아닌 것
- 모든 데이터를 전부 나열하는 화면
- 예쁘지만 의미 없는 차트
- 클릭은 많지만 판단이 어려운 UI

✅ 관제 화면의 진짜 역할
- 정상 데이터는 넘기고,  
- 이상 데이터만 강조해서  
- 사람이 빠르게 판단하도록 돕는 화면

이 역할을 코드와 화면 구조로 명확히 드러내는 것이 8단계의 목표입니다.

---
### 🧩 관제 대시보드 기본 화면 구성

관제 담당자는 표를 보기 전에 “지금 위험한가?”를 먼저 확인합니다.
그래서 가장 위에 아래 3개를 숫자로 요약합니다.
- 전체 데이터 개수
- 위험(기준 초과) 개수
- 상태(정상/주의/경고)

1️⃣ 상단 – 현재 상태 요약
- 전체 데이터 개수
- 기준 초과(위험) 데이터 개수
- 현재 정상 / 주의 / 경고 같은 상태 메시지
    
👉 한눈에 지금 상황이 어떤지 알기 위함

✅ 추가 코드 :
```python
# =====================================================
# ✅ 5) 관제 핵심: 임계치 기반 필터링
# =====================================================
st.subheader("✅ 3) 관제 필터링 (임계치 이상만 보기)")

pm10_candidates = ["PM10", "pm10"]
#......코드생략
```
⚠️ 이 줄 “바로 아래”에 넣으면 화면 흐름이 자연스럽습니다.

✅ [추가] 1️⃣ 상단 상태 요약 코드
```python
# =====================================================
# ✅ 5-0) 관제 핵심: 임계치 기반 필터링
# =====================================================
st.subheader("📌 현재 관제 상태 요약")

total_count = len(df)
bad_count = len(filtered)

c1, c2, c3 = st.columns(3)

with c1:
    st.metric("전체 데이터 수", total_count)

with c2:
    st.metric(f"PM10 ≥ {threshold_pm10} 지역 수", bad_count)

with c3:
    # 위험 비율로 상태 표현(단순한 룰)
    if bad_count == 0:
        st.success("상태: 정상")
    elif bad_count < total_count * 0.3:
        st.warning("상태: 주의")
    else:
        st.error("상태: 경고")
```

현재 관제상태 요약 추가전
![[Pasted image 20251221172300.png]]

현재 관제상태 요약 추가후
![[Pasted image 20251221172309.png]]

---
2️⃣ 기준(임계치) 조절 영역

- PM10 기준값 슬라이더
- 나쁨 이상만 보기 체크박스
    
👉 사람이 상황에 따라 기준을 바꿔가며 판단 (이미 코드에 구현이 되어 있음)

PM10 기준값 슬라이더
```python
threshold_pm10 = st.sidebar.slider(
    "PM10 임계치 (나쁨 기준)",
    min_value=0,
    max_value=150,
    value=50,
    step=5,
)
```

나쁨 이상만 보기 체크박스
```python
only_bad = st.sidebar.checkbox("나쁨 이상만 보기(PM10 임계치 이상)", value=True)
```

그리고 실제 필터링은 여기서 적용돼요:
```python
if only_bad:
    filtered = df[df[pm10_col] >= threshold_pm10].copy()
else:
    filtered = df.copy()
```
➡️ 체크 ON: 임계치 이상만 남김  
➡️ 체크 OFF: 전체 데이터를 그대로 둠

---
3️⃣ 위험 목록 (핵심)
- 기준 이상 데이터만 필터링된 표
- 위치 + 수치 중심 (불필요한 컬럼 제거)
    
👉 관제 화면의 핵심
	지금 당장 봐야 할 대상이 누구인가?

위의 코드에서 이 부분이 이미 관제의 핵심 역할을 수행합니다.
```python
st.dataframe(
    filtered[show_cols].sort_values(by=pm10_col, ascending=False),
    use_container_width=True
)
```

관제 관점 설명
- 전체 데이터가 아니라
- 지금 당장 봐야 할 위험 대상 목록만 남긴다
- PM10 내림차순 = 우선순위 정렬

기준 이상 데이터만 필터링된 표
```python
if only_bad:
    filtered = df[df[pm10_col] >= threshold_pm10].copy()
```
임계치(`threshold_pm10`)를 넘은 데이터만 `filtered`에 남김

---
4️⃣ 차트 영역 (보조 판단: TOP10)

위의 코드에서 이 부분도 관제 목적에 딱 맞게 구성되어 있습니다.
```python
st.bar_chart(chart_df[[pm10_col]])
```

왜 차트가 필요한가?
- 숫자표는 정확한 값에 유리
- 차트는 비교에 유리  
    👉 위험도가 높은 지역이 눈으로 바로 보임

---
5️⃣ 에러/대체 안내 영역 (운영 필수)

이 부분도 이미 실무 관제 방식으로 구현하셨습니다.
```python
if err is not None:
    st.error("❌ 데이터를 가져오지 못했습니다.")
    st.code(err)
    st.stop()
```

실무에서는 API가 자주 깨집니다.
- 인증키 만료
- 서버 장애
- 응답 포맷 변경
- 타임아웃
    
그래서 관제 시스템의 완성 기준은:

> ❌ “에러가 안 나는 코드”  
> ⭕ “에러가 나도, 이유를 설명하고 화면이 유지되는 코드”

---

### 새로고침 버튼(refresh)을 “의도적으로” 동작하게 만들기
 
현재 코드에는 이미 아래 버튼이 있습니다.
```python
refresh = st.sidebar.button("🔄 새로고침(재요청)")
```
하지만 이런 의문이 생깁니다.
> 버튼을 눌렀는데… 정말 다시 API를 호출한 건지 잘 모르겠어요.
> 그 이유는 Streamlit의 기본 동작 방식 때문입니다.

🧠 Streamlit의 기본 동작 원리 (중요)
- Streamlit 앱은  
    👉 위에서 아래로 스크립트를 다시 실행하는 구조입니다.
- 버튼을 누르면 사실상 재실행은 이미 일어나고 있습니다.
    
❗ 하지만
- 코드 흐름상 버튼을 눌렀기 때문에 다시 실행한다는 의도가 코드에 명시적으로 보이지 않습니다.
- 그래서 학습 단계에서는 동작이 불분명하게 느껴집니다.

코드 추가하기
```python
if refresh:
    st.rerun()
```
이 코드는 이렇게 읽으면 됩니다 👇
> 사용자가 ‘새로고침’ 버튼을 눌렀다면  
> 이 앱을 지금 즉시 처음부터 다시 실행하라

```python
# =====================================================
# 1) 사이드바: 데이터 소스 & 관제 설정
# =====================================================
st.sidebar.header("⚙️ 설정")
# ..... 기존코드


# ✅ 새로고침 버튼
refresh = st.sidebar.button("🔄 새로고침(재요청)")

# ✅ 버튼을 눌렀을 때 즉시 rerun(강제 재실행)
if refresh:
    st.rerun()
```
👉 이 위치에 조건문을 추가하는 이유:
- 아직 데이터 로딩(API/CSV)이 시작되기 전
- 버튼 클릭 → 즉시 앱 전체 재실행
- “재요청”이라는 개념이 코드 흐름상 명확해짐

🔄 이 한 줄이 만들어주는 차이
❌ 기존 (버튼만 있을 때)
- 버튼을 눌러도  
    → “지금 이 버튼이 뭘 했는지” 초보자가 체감하기 어려움
- Streamlit 내부 동작을 암묵적으로 의존
    
⭕ 개선 후 (`st.rerun()` 추가)
- 버튼 클릭 = 앱을 다시 시작
- API 재호출 / CSV 재로딩이 명확
- 관제 시스템에서 말하는  
    “재요청 / 새로고침” 개념을 정확히 적용

코드를 추가하면 브라우저에서는 
- 화면 깜빡임 ❌
- 새 메시지 ❌
- 상태 변화 ❌
👉 아무 변화가 없어 보이는 게 맞습니다

그럼 왜 이 코드를 “굳이” 넣을까?
	이건 개념 학습 + 실무 대비용 코드입니다

지금은 버튼을 눌러도 자동으로 다시 실행되기 때문에  
`st.rerun()`이 눈에 띄지 않는다.

하지만 캐시, 세션 상태, 조건부 로딩이 들어가는 순간  
이 한 줄이 진짜 ‘강제 재요청 버튼’ 이 된다.

F5는 브라우저를 전체 새로고침하는 역할
`st.rerun()` 버튼 👉 Streamlit 앱만 다시 실행

`st.rerun()` 버튼은 무엇을 하나?
```python
if refresh:
    st.rerun()
```
이건:
- 브라우저는 그대로
- 서버 연결 유지
- 현재 세션 안에서
- 파이썬 스크립트만 다시 실행
    
📌 쉽게 말해:
	앱 내부에서 새로고침

비교 대상:
1. F5 (브라우저 새로고침)
2. 버튼만 있고 `if refresh:` 없는 경우
3. 버튼 + `if refresh: st.rerun()` 있는 경우

|구분|F5(브라우저 새로고침)|버튼만 있음|버튼 + `if refresh: st.rerun()`|
|---|---|---|---|
|클릭 대상|브라우저|Streamlit 위젯|Streamlit 위젯|
|코드 재실행|⭕ (서버 재연결 포함 가능)|⭕ (Streamlit이 자동 재실행)|⭕ (즉시 재실행 강제)|
|“강제 재실행” 의도|개발자 제어 밖|❌ (자동 동작에 의존)|⭕ (코드에 의도가 명시됨)|
|세션 유지|❌ (초기화될 수 있음)|⭕|⭕|
|`st.session_state` 유지|❌ (초기화될 수 있음)|⭕|⭕|
|캐시 제어(예: `st.cache_data`)|❌|❌|⭕ (캐시 비우고 재요청 같은 패턴 가능)|
|실무 관제에서 “재요청 버튼” 의미|애매함|애매함|명확함|
핵심은 이거예요:  
지금 단계에서는 버튼만 있어도 재실행이 되기 때문에 “겉으로 티가 안 나는 게 정상”이고,  
나중에 `cache` / `session_state` / “조건부 로딩”이 들어오면 `st.rerun()`이 진짜로 의미가 커집니다.


최종코드
```python
import streamlit as st
import requests
import pandas as pd

# =====================================================
# ✅ 0) Streamlit 기본 페이지 설정
# =====================================================
st.set_page_config(
    page_title="서울 미세먼지 관제 대시보드",
    page_icon="🌫️",
    layout="wide",
)

st.title("🌫️ 서울 미세먼지 관제 대시보드 (API / CSV → DataFrame → 관제 + 탐색)")
st.write(
    "데이터를 가져와서 DataFrame으로 만들고, 임계치 기준으로 위험 지역을 걸러본 뒤 "
    "필터/정렬로 관제 담당자처럼 탐색합니다."
)

# =====================================================
# ✅ 1) 사이드바: 데이터 소스 선택 + 관제 설정
# =====================================================
st.sidebar.header("⚙️ 설정")

data_source = st.sidebar.radio(
    "데이터 소스 선택",
    ["API(실시간)", "CSV 업로드(파일)", "CSV 로컬(폴더)"],
    index=1,
)

api_key = st.sidebar.text_input(
    "서울 OpenAPI KEY (API 모드에서만 사용)",
    value="",
    placeholder="여기에 본인 API KEY 입력",
)

limit = st.sidebar.selectbox("가져올 데이터 개수 (API 모드)", [5, 10, 20, 50], index=3)

threshold_pm10 = st.sidebar.slider(
    "PM10 임계치 (나쁨 기준)",
    min_value=0,
    max_value=150,
    value=50,
    step=5,
)

only_bad = st.sidebar.checkbox("나쁨 이상만 보기(PM10 임계치 이상)", value=True)

refresh = st.sidebar.button("🔄 새로고침(재요청)")

# ✅ 버튼을 눌렀을 때 즉시 rerun(강제 재실행)
if refresh:
    st.rerun()

# =====================================================
# ✅ 2) 데이터 로딩 함수들
# =====================================================

# ✅ [추가] API 호출은 캐시 적용(과도한 요청 방지)
@st.cache_data(ttl=60)
def load_from_api(api_key: str, limit: int):
    """
    API에서 JSON을 가져와 DataFrame으로 변환합니다.
    반환: (df, err)
      - 성공: (DataFrame, None)
      - 실패: (None, 에러메시지 str)
    """
    if not api_key.strip():
        return None, "API KEY가 비어 있습니다. 왼쪽 사이드바에 KEY를 입력하세요."

    url = f"http://openapi.seoul.go.kr:8088/{api_key}/json/airQualityAPI/1/{limit}/"

    try:
        response = requests.get(url, timeout=10)
    except Exception as e:
        return None, f"요청 실패(네트워크/timeout 가능): {e}"

    # ✅ [추가] HTTP 상태코드 방어(200이 아니면 에러 처리)
    if response.status_code != 200:
        return None, (
            f"HTTP 요청 실패 (status={response.status_code})\n\n"
            f"응답 미리보기:\n{response.text[:800]}"
        )

    content_type = response.headers.get("Content-Type", "")

    # ✅ [수정] Content-Type 판별을 이전 단계와 동일하게(더 안전)
    is_json = content_type.lower().startswith("application/json")
    if not is_json:
        preview = response.text[:800]
        return None, (
            f"JSON이 아닙니다. (Content-Type: {content_type})\n\n"
            f"응답 미리보기(앞부분):\n{preview}"
        )

    try:
        data = response.json()
    except Exception as e:
        return None, f"JSON 파싱 실패: {e}"

    if "airQualityAPI" not in data or "row" not in data["airQualityAPI"]:
        return None, f"응답 구조가 예상과 다릅니다.\n키 목록: {list(data.keys())}"

    rows = data["airQualityAPI"]["row"]
    df = pd.DataFrame(rows)
    return df, None


def load_from_csv_upload(uploaded_file):
    if uploaded_file is None:
        return None, "CSV 파일이 업로드되지 않았습니다. 왼쪽에서 파일을 선택하세요."

    try:
        df = pd.read_csv(uploaded_file)
        return df, None
    except Exception as e:
        return None, f"CSV 읽기 실패: {e}"


def load_from_csv_local(filename: str = "dust_sample.csv"):
    try:
        df = pd.read_csv(filename)
        return df, None
    except FileNotFoundError:
        return None, f"'{filename}' 파일을 찾을 수 없습니다. app.py와 같은 폴더에 두세요."
    except Exception as e:
        return None, f"로컬 CSV 읽기 실패: {e}"


# =====================================================
# ✅ 3) 데이터 로딩 (API / CSV)
# =====================================================
st.subheader("✅ 1) 데이터 로딩 결과")

# ✅ [추가] refresh 버튼 클릭 시 캐시 삭제 + 재실행
# (API 모드일 때 의미가 큼)
if refresh:
    load_from_api.clear()
    st.rerun()

uploaded_file = None
if data_source == "CSV 업로드(파일)":
    uploaded_file = st.sidebar.file_uploader("CSV 파일 업로드", type=["csv"])

if data_source == "API(실시간)":
    df, err = load_from_api(api_key, limit)
elif data_source == "CSV 업로드(파일)":
    df, err = load_from_csv_upload(uploaded_file)
else:
    df, err = load_from_csv_local("dust_sample.csv")

if err is not None:
    st.error("❌ 데이터를 가져오지 못했습니다.")
    st.write("아래 메시지를 확인하세요. (실무 관제에서도 이런 안내가 매우 중요합니다.)")
    st.code(err)
    st.stop()

# ✅ [추가] df None 방어 (혹시 모를 예외 케이스)
if df is None:
    st.error("❌ DataFrame 생성 실패(df=None).")
    st.stop()

st.success("✅ 데이터 로딩 성공 (DataFrame 생성 완료)")

# =====================================================
# ✅ 4) DataFrame 기본 확인
# =====================================================
st.subheader("✅ 2) DataFrame 미리보기 / 컬럼 확인")

col1, col2 = st.columns([2, 1])

with col1:
    st.write("📊 상위 5줄(df.head())")
    # ✅ [수정] width="stretch" 제거 → Streamlit 표준 use_container_width 사용
    st.dataframe(df.head(), use_container_width=True)

with col2:
    st.write("📌 컬럼 목록")
    st.write(list(df.columns))

# =====================================================
# ✅ 5) 관제 핵심: 임계치 기반 필터링
# =====================================================
st.subheader("✅ 3) 관제 필터링 (임계치 이상만 보기)")

pm10_candidates = ["PM10", "pm10"]
station_candidates = ["MSRSTE_NM", "station"]

pm10_col = next((c for c in pm10_candidates if c in df.columns), None)
station_col = next((c for c in station_candidates if c in df.columns), None)

if pm10_col is None:
    st.warning("⚠️ PM10 컬럼을 찾지 못했습니다. df.columns를 확인해서 pm10 컬럼명을 맞춰주세요.")
    st.stop()

df[pm10_col] = pd.to_numeric(df[pm10_col], errors="coerce")

if only_bad:
    filtered = df[df[pm10_col] >= threshold_pm10].copy()
else:
    filtered = df.copy()

st.write(f"- 사용 컬럼: **{pm10_col}**")
st.write(f"- 필터 기준: **{pm10_col} >= {threshold_pm10}**")
st.write(f"- 결과 행 개수: **{len(filtered)}개**")

show_cols = []
if station_col is not None:
    show_cols.append(station_col)
show_cols.append(pm10_col)

if len(filtered) == 0:
    st.info("✅ 기준 이상 지역이 없습니다. (모두 양호)")
else:
    st.dataframe(
        filtered[show_cols].sort_values(by=pm10_col, ascending=False),
        use_container_width=True,  # ✅ [수정]
    )

# =====================================================
# ✅ 5-0) 관제 핵심: 임계치 기반 필터링
# =====================================================
st.subheader("📌 현재 관제 상태 요약")

total_count = len(df)
bad_count = len(filtered)

c1, c2, c3 = st.columns(3)

with c1:
    st.metric("전체 데이터 수", total_count)

with c2:
    st.metric(f"PM10 ≥ {threshold_pm10} 지역 수", bad_count)

with c3:
    # 위험 비율로 상태 표현(단순한 룰)
    if bad_count == 0:
        st.success("상태: 정상")
    elif bad_count < total_count * 0.3:
        st.warning("상태: 주의")
    else:
        st.error("상태: 경고")


# =====================================================
# ✅ 6) 관제 마무리: 간단 차트 TOP N
# =====================================================
st.subheader("✅ 4) 차트로 보기 (관제 마무리: TOP 10)")

if len(filtered) == 0:
    st.info("표시할 데이터가 없습니다. 임계치를 낮추거나 '나쁨 이상만 보기'를 해제해보세요.")
else:
    chart_df = filtered.copy()

    if station_col is not None:
        chart_df = chart_df[[station_col, pm10_col]].dropna()
        chart_df = chart_df.sort_values(by=pm10_col, ascending=False).head(10)
        chart_df = chart_df.set_index(station_col)
        st.bar_chart(chart_df[[pm10_col]])
    else:
        chart_df = chart_df[[pm10_col]].dropna().sort_values(by=pm10_col, ascending=False).head(10)
        st.bar_chart(chart_df[[pm10_col]])

st.caption("✅ 정리: 데이터가 API든 CSV든, DataFrame만 만들면 필터링/표/차트는 동일하게 동작합니다.")

# =====================================================================
# ✅ 7) 관제 탐색: 필터링/정렬 실습 파트
# =====================================================================
st.divider()
st.header("🔍 5) 관제 데이터 탐색 (필터링 / 정렬)")

st.write(
    """
**왜 이 단계가 필요한가?**

- 앞에서는 시스템이 임계치로 **자동 감지(관제)** 를 했습니다.
- 이제는 관제 담당자처럼 **직접 질문하며 확인(탐색/검증)** 합니다.

✅ 필터링 = “조건에 맞는 행만 남긴다”  
✅ 정렬 = “남은 행을 위험도/우선순위 순서로 본다”
"""
)

explore_df = df.copy()

# -------------------------------------------------
# 7-1) 지역 선택 필터링
# -------------------------------------------------
if station_col is None:
    st.warning("⚠️ 지역(station) 컬럼이 없어 지역 기반 탐색(필터)이 제한됩니다. CSV 컬럼명을 확인하세요.")
else:
    st.subheader("① 선택한 지역만 보기 (필터링)")

    stations = sorted(explore_df[station_col].dropna().unique())
    selected_station = st.selectbox("지역 선택", stations)

    station_only = explore_df[explore_df[station_col] == selected_station].copy()
    st.write(f"✅ **{selected_station}** 데이터만 표시합니다.")
    st.dataframe(station_only, use_container_width=True)  # ✅ [수정]

# -------------------------------------------------
# 7-2) PM10 기준 이상 필터링(탐색용)
# -------------------------------------------------
st.subheader("② PM10 기준값을 바꿔가며 보기 (필터링)")

pm10_series = explore_df[pm10_col].dropna()
min_pm10 = int(pm10_series.min()) if len(pm10_series) else 0
max_pm10 = int(pm10_series.max()) if len(pm10_series) else 150

threshold_test = st.slider(
    "PM10 기준 값 선택(탐색용)",
    min_value=min_pm10,
    max_value=max_pm10,
    value=min(50, max_pm10),
)

high_pm10 = explore_df[explore_df[pm10_col] >= threshold_test].copy()
st.write(f"✅ PM10이 **{threshold_test} 이상**인 행만 표시합니다.")
st.dataframe(high_pm10, use_container_width=True)  # ✅ [수정]

# -------------------------------------------------
# 7-3) 정렬: 위험도 순으로 보기
# -------------------------------------------------
st.subheader("③ 위험도 순서로 정렬 (Sort)")

sort_order = st.radio(
    "정렬 순서 선택",
    ("PM10 높은 순", "PM10 낮은 순"),
    horizontal=True,
)

if sort_order == "PM10 높은 순":
    sorted_df = explore_df.sort_values(pm10_col, ascending=False)
else:
    sorted_df = explore_df.sort_values(pm10_col, ascending=True)

st.dataframe(sorted_df, use_container_width=True)  # ✅ [수정]

# -------------------------------------------------
# 7-4) 지역 + 정렬 조합 (필터 + 정렬)
# -------------------------------------------------
st.subheader("④ 지역 + 정렬 조합 보기 (필터 + 정렬)")

if station_col is None:
    st.info("지역(station) 컬럼이 없어서 조합 기능을 사용할 수 없습니다.")
else:
    c1, c2 = st.columns(2)

    with c1:
        station_for_combo = st.selectbox("지역 선택(조합용)", stations, key="combo_station")

    with c2:
        order_for_combo = st.radio(
            "정렬 순서(조합용)",
            ("PM10 높은 순", "PM10 낮은 순"),
            key="combo_order",
            horizontal=True,
        )

    combo_filtered = explore_df[explore_df[station_col] == station_for_combo].copy()

    if order_for_combo == "PM10 높은 순":
        combo_result = combo_filtered.sort_values(pm10_col, ascending=False)
    else:
        combo_result = combo_filtered.sort_values(pm10_col, ascending=True)

    st.write(f"✅ **{station_for_combo}** / **{order_for_combo}** 결과")
    st.dataframe(combo_result, use_container_width=True)  # ✅ [수정]

st.caption("✅ 관제 팁: 필터(대상 선택) → 정렬(우선순위) 순서로 보면, 실무 관제처럼 판단이 쉬워집니다.")
```

터미널에 이런 경고가 뜹니다.
```
For `use_container_width=True`, use `width='stretch'`. For `use_container_width=False`, use `width='content'`.
2025-12-21 19:11:27.795 Please replace `use_container_width` with `width`.

`use_container_width` will be removed after 2025-12-31.
```

`use_container_width` 경고의 원인

Streamlit이 최근 버전에서
- `st.dataframe(..., use_container_width=True)` 같은 옵션을
- 곧 제거(deprecate) 하기로 결정해서
    
2025-12-31 이후 제거 예정이라고 미리 경고를 띄우는 겁니다.

그래서 Streamlit이 “이제부터는 이렇게 쓰세요”라고 안내하는 거예요:
- `use_container_width=True` → `width="stretch"`
- `use_container_width=False` → `width="content"`
    
✅ 즉, 코드가 틀려서가 아니라, Streamlit API가 바뀌는 중이라서 나는 경고입니다.

모든 코드에서
```python
use_container_width=True
```
위의 코드를

```python
width="stretch"
```
로 한 번에 수정 + 검증

바로 수정하지 말고 먼저 어디에 있는지 확인
```bash
rg "use_container_width=True"
```

결과
```
(streamlit_day8) youjung@DESKTOP-PJCRMMU:~/day1_console/day9_api$ rg "use_container_width=True"
Command 'rg' not found, but can be installed with:
sudo snap install ripgrep  # version 12.1.0, or
sudo apt  install ripgrep  # version 14.0.3-1
See 'snap info ripgrep' for additional versions.
(streamlit_day8) youjung@DESKTOP-PJCRMMU:~/day1_console/day9_api$ 
```

해결방법:
1단계) ripgrep 설치
```bash
sudo apt update
sudo apt install ripgrep
```

설치 확인:
```bash
rg --version
```

2단계) 다시 검색 실행
```bash
rg "use_container_width=True"
```

결과
```
seoul_air_api.py
175:    st.dataframe(df.head(), use_container_width=True)
217:        use_container_width=True,  # ✅ [수정]
300:    st.dataframe(station_only, use_container_width=True)  # ✅ [수정]
320:st.dataframe(high_pm10, use_container_width=True)  # ✅ [수정]
338:st.dataframe(sorted_df, use_container_width=True)  # ✅ [수정]
369:    st.dataframe(combo_result, use_container_width=True)  # ✅ [수정]

venv/lib/python3.12/site-packages/streamlit/hello/animation_demo.py
60:        image.image(1.0 - (n_matrix / n_matrix.max()), use_container_width=True)

venv/lib/python3.12/site-packages/streamlit/hello/dataframe_demo.py
57:            st.altair_chart(chart, use_container_width=True)

venv/lib/python3.12....
```

3단계) 한꺼번에 수정 (백업 포함)
```bash
sed -i.bak 's/use_container_width=True/width="stretch"/g' seoul_air_api.py
```

4단계) 남아있는지 재확인
```bash
rg "use_container_width" seoul_air_api.py
```

최종확인
```bash
streamlit run seoul_air_api.py
```
이제 터미널이 정상적으로 출력됩니다.
