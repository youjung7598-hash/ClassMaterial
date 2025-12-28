프로젝트 코드명: `post-finder-cli`
프로젝트 유형: 외부 JSON API 기반 CLI 데이터 검색 & 저장 도구
    
🖥️ 프로젝트 개요:  
    JSONPlaceholder의 `/posts` API를 사용해서,
    - `userId`와 `키워드`를 입력받고
    - 해당 조건에 맞는 게시글을 API로 조회 → 필터 → 출력 → 파일로 저장하는 콘솔 프로그램입니다.  이 과정에서 자연스럽게 HTTP 요청, 쿼리 파라미터, JSON 파싱, 필터링, 예외 처리, 파일 저장, 디버깅 감각을 모두 연습하게 됩니다.

###### 이 프로젝트를 통해 자연스럽게 적용되는 내용
|구분|내용|
|---|---|
|웹 요청|`requests.get()`으로 외부 API 호출하기, `params`로 쿼리 파라미터 보내기|
|HTTP 상태 코드|`response.status_code`로 200/404/500 등 상태 확인하고 분기 처리|
|예외 처리|`try / except`로 네트워크 오류·입력 오류 방어하기 (`RequestException`, `ValueError`)|
|JSON 파싱|`response.json()`으로 JSON → dict/list 변환, 키 접근 (`["title"]`, `["body"]`)|
|데이터 필터링|제목/본문에 **키워드 포함 여부**로 필터링 (`if keyword.lower() in title.lower()`)|
|리스트/반복문|for 반복으로 게시글 목록 순회, `enumerate()`로 번호 붙이기|
|함수 분리|`fetch_posts`, `filter_posts`, `save_posts_to_json`, `print_posts` 등으로 역할 분리|
|파일 저장|필터된 결과를 `.json`, `.txt`로 저장 (`open`, `json.dump`)|
|디버깅 습관|상태 코드/에러 메시지 프린트, None 체크, 빈 리스트 처리 등 방어적 코드 작성|

###### 📄 기능 요구사항 (Functional Requirements)
|기능ID|기능설명|세부내용|
|---|---|---|
|F-01|사용자 입력 받기|콘솔에서 `userId`(정수)와 `검색 키워드`(문자열)를 입력받는다. 숫자 입력이 잘못되면 친절한 에러 메시지를 출력하고 종료한다.|
|F-02|게시글 목록 API 호출|`https://jsonplaceholder.typicode.com/posts`에 `userId`를 쿼리 파라미터로 붙여 GET 요청을 보낸다 (`params={"userId": user_id}` 사용).|
|F-03|HTTP 상태 코드 검사|응답 상태 코드가 200이 아닐 경우, 상태 코드와 함께 에러 메시지를 출력하고 프로그램을 종료한다.|
|F-04|네트워크 예외 처리|요청 과정에서 `requests.exceptions.RequestException`이 발생하면 잡아서 “네트워크 오류” 메시지를 출력하고 종료한다.|
|F-05|JSON 응답 파싱|`response.json()`으로 JSON을 파이썬 객체로 변환한다. 응답이 리스트인지 타입을 출력해본다.|
|F-06|키워드 필터링|받은 게시글 중에서 `title` 또는 `body`에 키워드가 (대소문자 무시) 포함된 게시글만 골라낸다.|
|F-07|콘솔 출력|필터된 게시글을 `[id=숫자] 제목` 형식으로 번호와 함께 출력하고, 본문 앞 50자 정도를 같이 보여준다. 필터 결과가 없으면 “검색 결과 없음” 메시지를 출력한다.|
|F-08|JSON 파일 저장|필터된 게시글들을 `filtered_posts_user{userId}.json` 이름의 파일로 저장한다 (`ensure_ascii=False, indent=2`).|
|F-09|텍스트 요약 파일 저장|동일한 데이터를 요약 형태(`id / title 한 줄`)로 `filtered_posts_user{userId}.txt`에 저장한다.|
|F-10|종료 메시지 출력|모든 처리가 끝난 뒤, 생성된 파일 이름과 함께 “프로그램 종료” 메시지를 출력한다.|

###### 📄 비기능 요구사항 (Non-Functional Requirements)
|항목|설명|
|---|---|
|실행 환경|Python 3.10+ 기준, 추가 외부 패키지는 `requests` 하나만 사용한다.|
|코드 구조|한 파일 안에서도 **역할별 함수 분리**를 지킨다. (입력, 요청, 필터링, 출력, 저장 등)|
|에러 메세지 가독성|모든 에러 상황에서 한국어로 “무슨 일이 일어났는지”를 설명하는 문장을 출력한다. (단순 Traceback만 보이지 않게 한다)|
|입력 검증|`userId`가 정수가 아니면 `ValueError`를 잡아서 안내하고, 프로그램은 조용히 종료한다.|
|방어적 프로그래밍|요청 실패, 빈 응답, 키 누락 등 상황에서 프로그램이 강제 종료되지 않도록 방어 코드를 작성한다.|
|학습 시간|초보 기준 **2시간 이내**에 구현·실행·간단한 수정까지 가능하도록 과도한 복잡도는 피한다.|
|코드 스타일|가독성 좋은 변수명/함수명, 적당한 주석, 상단에 상수/URL 모으기 등 “나중에 같이 보는 사람”을 배려한 코드 작성.|

✅ 출력 포맷 구성

`1)` 콘솔 출력 예시
```
📌 JSONPlaceholder 게시글 검색기 (post-finder-cli)
조회할 사용자 ID(userId)를 입력하세요 (예: 1): 1
검색할 키워드를 입력하세요 (예: qui): qui

👉 요청 URL: https://jsonplaceholder.typicode.com/posts
👉 요청 파라미터: {'userId': 1}
응답 상태 코드: 200
전체 게시글 개수: 10

🔎 'qui' 가 포함된 게시글 목록
========================================
1. [id=2] qui est esse
   본문 미리보기: est rerum tempore vitae sequi sint nihil reprehenderit dolor beatae ea dolores neque...

2. [id=3] ea molestias quasi exercitationem repellat qui ipsa sit aut
   본문 미리보기: et iusto sed quo iure voluptatem occaecati omnis eligendi aut ad voluptatem doloribus vel accusantium...

총 2개의 게시글이 검색되었습니다.

💾 JSON 파일로 저장됨: filtered_posts_user1.json
💾 텍스트 파일로 저장됨: filtered_posts_user1.txt

프로그램을 종료합니다. 👋
```

`2)` 생성되는 파일 예시
- `filtered_posts_user1.json`
    - 실제 JSON 데이터 전체 (id, userId, title, body 포함)
        
- `filtered_posts_user1.txt`
    - 한 줄에 한 게시글씩  
        `id=2 | qui est esse` 형태로 저장

✅ 정답 코드 (`post_finder_cli.py`)
아래 코드 그대로 `post_finder_cli.py`로 저장해서 실행하면 됩니다.
```python
"""
post_finder_cli.py

외부 JSON API(JSONPlaceholder)를 사용해
특정 userId의 게시글 중에서
제목(title) 또는 본문(body)에 특정 키워드가 포함된 것만
검색해서 출력하고, 파일로 저장하는 콘솔 프로그램.

학습 포인트:
- requests.get() + params로 API 호출하기
- HTTP 상태 코드 확인하기
- try/except로 네트워크/입력 에러 처리하기
- response.json()으로 JSON 파싱하기
- 리스트/딕셔너리 필터링하기
- 결과를 JSON / TXT 파일로 저장하기
"""

from __future__ import annotations

import json
from typing import Any, Dict, List, Optional

import requests
from requests.exceptions import RequestException

# -------------------------------
# 상수 정의
# -------------------------------

# JSONPlaceholder posts API 기본 URL
BASE_URL = "https://jsonplaceholder.typicode.com/posts"


# -------------------------------
# 1) API에서 게시글 가져오기
# -------------------------------

def fetch_posts_by_user(user_id: int) -> Optional[List[Dict[str, Any]]]:
    """
    주어진 userId에 해당하는 posts를 API에서 가져온다.

    성공 시: 게시글(dict) 리스트 반환
    실패 시: None 반환

    네트워크 오류, 상태 코드 200 아님, JSON 파싱 오류 등 상황을
    모두 방어적으로 처리한다.
    """
    params = {"userId": user_id}
    print()
    print("👉 API 요청 준비 중...")
    print(f"👉 요청 URL: {BASE_URL}")
    print(f"👉 요청 파라미터: {params}")

    try:
        # timeout을 설정해서 서버가 너무 오래 멈춰 있지 않도록 함
        response = requests.get(BASE_URL, params=params, timeout=5)
    except RequestException as e:
        print("❗ 네트워크 오류가 발생했습니다.")
        print("   (인터넷 연결 문제, 서버 응답 지연 등)")
        print("   상세 내용:", e)
        return None

    print("응답 상태 코드:", response.status_code)

    # 200 OK가 아닐 경우, 더 이상 진행하지 않음
    if response.status_code != 200:
        print("⚠️ 200 OK 응답이 아닙니다. 요청이 정상 처리되지 않았습니다.")
        print("   응답 본문:", response.text[:200])
        return None

    # JSON 파싱
    try:
        data = response.json()
    except ValueError as e:
        print("❗ JSON 파싱 중 오류가 발생했습니다.")
        print("   응답 본문:", response.text[:200])
        print("   상세 내용:", e)
        return None

    # 기대 형태는 'list' (여러 게시글)
    if not isinstance(data, list):
        print("⚠️ 예상과 다른 응답 형식입니다. (list가 아님)")
        print("   타입:", type(data))
        print("   내용:", data)
        return None

    print("전체 게시글 개수:", len(data))
    return data


# -------------------------------
# 2) 키워드로 게시글 필터링
# -------------------------------

def filter_posts_by_keyword(
    posts: List[Dict[str, Any]],
    keyword: str,
) -> List[Dict[str, Any]]:
    """
    posts 리스트 중에서
    title 또는 body에 keyword가 (대소문자 무시) 포함된 것만 필터링한다.

    keyword가 비어 있으면, 전체 posts를 그대로 반환한다.
    """
    keyword = keyword.strip()

    # 키워드가 비어 있으면, 필터링 없이 전체 반환
    if not keyword:
        return posts

    lower_kw = keyword.lower()
    filtered: List[Dict[str, Any]] = []

    for post in posts:
        # 방어 코드: title/body가 없을 수도 있으므로 get() 사용
        title = str(post.get("title", "")).lower()
        body = str(post.get("body", "")).lower()

        if lower_kw in title or lower_kw in body:
            filtered.append(post)

    return filtered


# -------------------------------
# 3) 결과 콘솔 출력
# -------------------------------

def print_posts(posts: List[Dict[str, Any]], keyword: str) -> None:
    """
    필터된 게시글 리스트를 콘솔에 보기 좋은 형식으로 출력한다.
    """
    print()
    print(f"🔎 '{keyword}' 가 포함된 게시글 목록")
    print("=" * 40)

    if not posts:
        print("검색 결과가 없습니다.")
        return

    for idx, post in enumerate(posts, start=1):
        post_id = post.get("id")
        title = str(post.get("title", "")).strip()
        body = str(post.get("body", "")).strip()

        # 본문 미리보기: 앞 80자만 잘라서 출력
        preview = body[:80].replace("\n", " ")

        print(f"{idx}. [id={post_id}] {title}")
        print(f"   본문 미리보기: {preview}...")
        print("-" * 40)

    print(f"총 {len(posts)}개의 게시글이 검색되었습니다.")


# -------------------------------
# 4) 결과를 파일로 저장
# -------------------------------

def save_posts_to_json(posts: List[Dict[str, Any]], user_id: int) -> str:
    """
    필터된 게시글 리스트를 JSON 파일로 저장한다.

    반환값: 생성된 파일 이름
    """
    filename = f"filtered_posts_user{user_id}.json"

    with open(filename, "w", encoding="utf-8") as f:
        json.dump(posts, f, ensure_ascii=False, indent=2)

    return filename


def save_posts_to_text(posts: List[Dict[str, Any]], user_id: int) -> str:
    """
    필터된 게시글의 id와 title만 한 줄씩 텍스트 파일로 저장한다.

    반환값: 생성된 파일 이름
    """
    filename = f"filtered_posts_user{user_id}.txt"

    with open(filename, "w", encoding="utf-8") as f:
        for post in posts:
            post_id = post.get("id")
            title = str(post.get("title", "")).strip()
            line = f"id={post_id} | {title}"
            f.write(line + "\n")

    return filename


# -------------------------------
# 5) 메인 흐름
# -------------------------------

def main() -> None:
    """
    콘솔 기반 메인 함수.

    1) userId, keyword 입력 받기
    2) API 호출 → 게시글 목록 가져오기
    3) 키워드로 필터링
    4) 콘솔에 출력
    5) JSON / TXT 파일로 저장
    """
    print("📌 JSONPlaceholder 게시글 검색기 (post-finder-cli)")

    # 1) userId 입력
    try:
        user_id_str = input("조회할 사용자 ID(userId)를 입력하세요 (예: 1): ").strip()
        user_id = int(user_id_str)
    except ValueError:
        print("❗ userId는 숫자로 입력해야 합니다. 프로그램을 종료합니다.")
        return

    # 2) 검색 키워드 입력
    keyword = input("검색할 키워드를 입력하세요 (예: qui, sunt 등 / 빈칸이면 전체): ").strip()
    if not keyword:
        print("⚠️ 키워드가 비어 있습니다. 전체 게시글을 대상으로 출력합니다.")

    # 3) API에서 게시글 가져오기
    posts = fetch_posts_by_user(user_id)
    if posts is None:
        print("❗ 게시글을 가져오지 못했습니다. 프로그램을 종료합니다.")
        return

    # 4) 키워드 필터링
    filtered_posts = filter_posts_by_keyword(posts, keyword)

    # 5) 콘솔 출력
    print_posts(filtered_posts, keyword)

    # 결과가 하나도 없으면 파일 저장 생략
    if not filtered_posts:
        print("파일로 저장할 데이터가 없으므로 저장을 생략합니다.")
        print("프로그램을 종료합니다. 👋")
        return

    # 6) 파일 저장
    json_filename = save_posts_to_json(filtered_posts, user_id)
    txt_filename = save_posts_to_text(filtered_posts, user_id)

    print()
    print(f"💾 JSON 파일로 저장됨: {json_filename}")
    print(f"💾 텍스트 파일로 저장됨: {txt_filename}")
    print()
    print("프로그램을 종료합니다. 👋")


# 이 파일을 직접 실행했을 때만 main()을 호출
if __name__ == "__main__":
    main()
```

---
### 역할별 분리 연습

디렉토리 구조
```
post_finder_cli/
├─ app_console.py          # 프로그램 시작점 (입력/흐름 제어)
└─ core/
   ├─ __init__.py          # 패키지 표시 (비어 있어도 됨)
   ├─ api_client.py        # API 호출 담당 (requests 사용)
   ├─ filters.py           # 키워드 필터링 로직
   └─ outputs.py           # 출력 + 파일 저장 담당
```
- `core/` 폴더 안에 비즈니스 로직 (API 호출, 필터링, 저장)을 넣고
- `app_console.py`는 입력 받고, 함수들을 조립만 하는 역할로 둡니다.

`core/api_client.py` — API 호출 전용
```python
"""
core/api_client.py

외부 JSON API(JSONPlaceholder)에 실제로 요청을 보내고
게시글 목록을 가져오는 기능만 담당하는 모듈.

- BASE_URL 상수 정의
- fetch_posts_by_user(user_id): userId로 posts 가져오기
"""

from __future__ import annotations

from typing import Any, Dict, List, Optional

import requests
from requests.exceptions import RequestException

# JSONPlaceholder posts API 기본 URL
BASE_URL = "https://jsonplaceholder.typicode.com/posts"


def fetch_posts_by_user(user_id: int) -> Optional[List[Dict[str, Any]]]:
    """
    주어진 userId에 해당하는 posts를 API에서 가져온다.

    성공 시: 게시글(dict) 리스트 반환
    실패 시: None 반환

    방어적으로 처리하는 상황:
    - 네트워크 오류 (인터넷, 서버 접속 문제)
    - HTTP 상태 코드가 200이 아닌 경우
    - JSON 파싱 실패
    - 응답 형식이 우리가 기대한 list가 아닌 경우
    """
    params = {"userId": user_id}
    print()
    print("👉 API 요청 준비 중...")
    print(f"👉 요청 URL: {BASE_URL}")
    print(f"👉 요청 파라미터: {params}")

    try:
        # timeout: 서버가 너무 오래 응답 안 할 때 무한 대기하지 않도록 제한
        response = requests.get(BASE_URL, params=params, timeout=5)
    except RequestException as e:
        print("❗ 네트워크 오류가 발생했습니다.")
        print("   (인터넷 연결 문제, 서버 응답 지연 등)")
        print("   상세 내용:", e)
        return None

    print("응답 상태 코드:", response.status_code)

    # 200 OK가 아니면 더 진행하지 않는다.
    if response.status_code != 200:
        print("⚠️ 200 OK 응답이 아닙니다. 요청이 정상 처리되지 않았습니다.")
        print("   응답 본문:", response.text[:200])
        return None

    # JSON 파싱 시도
    try:
        data = response.json()
    except ValueError as e:
        print("❗ JSON 파싱 중 오류가 발생했습니다.")
        print("   응답 본문:", response.text[:200])
        print("   상세 내용:", e)
        return None

    # 이 API에서는 list가 오는 것이 정상 (여러 게시글)
    if not isinstance(data, list):
        print("⚠️ 예상과 다른 응답 형식입니다. (list가 아님)")
        print("   타입:", type(data))
        print("   내용:", data)
        return None

    print("전체 게시글 개수:", len(data))
    return data
```

`core/filters.py` — 키워드 필터링 로직
```python
"""
core/filters.py

API로부터 받은 게시글 목록(posts)에서
검색 키워드(keyword)에 따라 필요한 데이터만 걸러내는 모듈.

- filter_posts_by_keyword(posts, keyword)
"""

from __future__ import annotations

from typing import Any, Dict, List


def filter_posts_by_keyword(
    posts: List[Dict[str, Any]],
    keyword: str,
) -> List[Dict[str, Any]]:
    """
    게시글 리스트(posts) 중에서
    title 또는 body에 keyword가 (대소문자 무시) 포함된 것만 필터링한다.

    - keyword 앞뒤 공백은 제거한다.
    - keyword가 비어 있으면, 필터링 없이 전체 posts를 그대로 반환한다.
    - title/body가 없을 수도 있으므로 dict.get()으로 안전하게 접근한다.
    """
    keyword = keyword.strip()

    # 키워드가 비어 있으면 필터링 하지 않고 전체 반환
    if not keyword:
        return posts

    lower_kw = keyword.lower()
    filtered: List[Dict[str, Any]] = []

    for post in posts:
        # 혹시라도 title/body가 None이거나 없는 경우를 대비해 str() + get() 사용
        title = str(post.get("title", "")).lower()
        body = str(post.get("body", "")).lower()

        # 키워드가 제목이나 본문에 하나라도 들어 있으면 결과에 추가
        if lower_kw in title or lower_kw in body:
            filtered.append(post)

    return filtered
```

`core/outputs.py` — 콘솔 출력 + 파일 저장
```python
"""
core/outputs.py

필터링된 게시글들을
- 콘솔에 보기 좋게 출력하고
- JSON / 텍스트 파일로 저장하는 기능을 담당하는 모듈.

- print_posts(posts, keyword)
- save_posts_to_json(posts, user_id)
- save_posts_to_text(posts, user_id)
"""

from __future__ import annotations

import json
from typing import Any, Dict, List


def print_posts(posts: List[Dict[str, Any]], keyword: str) -> None:
    """
    필터된 게시글 리스트를 콘솔에 보기 좋은 형식으로 출력한다.

    - 인덱스 번호(1부터 시작)
    - [id=글번호] 제목
    - 본문 일부(앞 80자) 미리보기
    """
    print()
    print(f"🔎 '{keyword}' 가 포함된 게시글 목록")
    print("=" * 40)

    if not posts:
        print("검색 결과가 없습니다.")
        return

    for idx, post in enumerate(posts, start=1):
        post_id = post.get("id")
        title = str(post.get("title", "")).strip()
        body = str(post.get("body", "")).strip()

        # 본문 미리보기: 앞 80자만 잘라서, 줄바꿈은 공백으로 치환
        preview = body[:80].replace("\n", " ")

        print(f"{idx}. [id={post_id}] {title}")
        print(f"   본문 미리보기: {preview}...")
        print("-" * 40)

    print(f"총 {len(posts)}개의 게시글이 검색되었습니다.")


def save_posts_to_json(posts: List[Dict[str, Any]], user_id: int) -> str:
    """
    필터된 게시글 리스트를 JSON 파일로 저장한다.

    - 파일 이름 예: filtered_posts_user1.json
    - ensure_ascii=False: 한글이 유니코드(\uXXXX)로 깨지지 않게 하기
    - indent=2: 예쁘게 들여쓰기

    반환값: 생성된 파일 이름 문자열
    """
    filename = f"filtered_posts_user{user_id}.json"

    with open(filename, "w", encoding="utf-8") as f:
        json.dump(posts, f, ensure_ascii=False, indent=2)

    return filename


def save_posts_to_text(posts: List[Dict[str, Any]], user_id: int) -> str:
    """
    필터된 게시글의 id와 title만 한 줄씩 텍스트 파일로 저장한다.

    - 파일 이름 예: filtered_posts_user1.txt
    - 한 줄에 하나의 게시글 정보를 넣는다.
      형식: id=숫자 | 제목

    반환값: 생성된 파일 이름 문자열
    """
    filename = f"filtered_posts_user{user_id}.txt"

    with open(filename, "w", encoding="utf-8") as f:
        for post in posts:
            post_id = post.get("id")
            title = str(post.get("title", "")).strip()
            line = f"id={post_id} | {title}"
            f.write(line + "\n")

    return filename
```

`core/__init__.py` — (선택) 묶어주기
```python
"""
core 패키지 초기화 파일.

여기에서 필요한 함수들을 다시 한 번 import 해서
core.fetch_posts_by_user 같은 형태로도 쓸 수 있게 만들어 줄 수 있다.
(선택 사항: 안 써도 프로그램 동작에는 문제 없음)
"""

from .api_client import fetch_posts_by_user  # noqa: F401
from .filters import filter_posts_by_keyword  # noqa: F401
from .outputs import (  # noqa: F401
    print_posts,
    save_posts_to_json,
    save_posts_to_text,
)
```
이 파일은 “편의용”이라서 없어도 되고, 있어도 됩니다.

`app_console.py` — 프로그램 진짜 시작점
```python
"""
app_console.py

콘솔에서 실행되는 시작점(엔트리 포인트) 파일.

역할:
- 사용자 입력 받기(userId, keyword)
- core.api_client / core.filters / core.outputs에 있는
  기능들을 조합해서 전체 흐름을 완성한다.

실제 "비즈니스 로직"은 core 쪽에 두고,
여기는 "입력/출력 흐름"만 담당하는 구조.
"""

from __future__ import annotations

from core import (
    fetch_posts_by_user,
    filter_posts_by_keyword,
    print_posts,
    save_posts_to_json,
    save_posts_to_text,
)


def main() -> None:
    """
    콘솔 기반 메인 함수.

    1) userId, keyword 입력 받기
    2) API 호출 → 게시글 목록 가져오기
    3) 키워드로 필터링
    4) 콘솔에 출력
    5) JSON / TXT 파일로 저장
    """
    print("📌 JSONPlaceholder 게시글 검색기 (post-finder-cli)")

    # 1) userId 입력
    try:
        user_id_str = input("조회할 사용자 ID(userId)를 입력하세요 (예: 1): ").strip()
        user_id = int(user_id_str)
    except ValueError:
        print("❗ userId는 숫자로 입력해야 합니다. 프로그램을 종료합니다.")
        return

    # 2) 검색 키워드 입력
    keyword = input(
        "검색할 키워드를 입력하세요 (예: qui, sunt 등 / 빈칸이면 전체): "
    ).strip()
    if not keyword:
        print("⚠️ 키워드가 비어 있습니다. 전체 게시글을 대상으로 출력합니다.")

    # 3) API에서 게시글 가져오기
    posts = fetch_posts_by_user(user_id)
    if posts is None:
        print("❗ 게시글을 가져오지 못했습니다. 프로그램을 종료합니다.")
        return

    # 4) 키워드 필터링
    filtered_posts = filter_posts_by_keyword(posts, keyword)

    # 5) 콘솔 출력
    print_posts(filtered_posts, keyword)

    # 결과가 하나도 없으면 파일 저장 생략
    if not filtered_posts:
        print("파일로 저장할 데이터가 없으므로 저장을 생략합니다.")
        print("프로그램을 종료합니다. 👋")
        return

    # 6) 파일 저장
    json_filename = save_posts_to_json(filtered_posts, user_id)
    txt_filename = save_posts_to_text(filtered_posts, user_id)

    print()
    print(f"💾 JSON 파일로 저장됨: {json_filename}")
    print(f"💾 텍스트 파일로 저장됨: {txt_filename}")
    print()
    print("프로그램을 종료합니다. 👋")


if __name__ == "__main__":
    main()
```

