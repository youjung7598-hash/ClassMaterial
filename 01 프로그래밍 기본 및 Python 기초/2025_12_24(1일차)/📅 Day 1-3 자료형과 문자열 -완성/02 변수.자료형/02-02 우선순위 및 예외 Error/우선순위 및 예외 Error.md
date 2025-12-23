### 🔹 연산자의 우선순위 
	하나의 계산식에 여러 연산자가 섞여 있을 때,  
	어떤 연산을 먼저 계산할지 정해진 규칙을 말합니다.

🔍 해석: 
	수학에서도 "괄호 먼저", "곱셈 먼저" 같은 계산 순서가 있듯이,  
	파이썬도 똑같이 연산자마다 우선 계산해야 할 순서가 정해져 있습니다.

 ‼️ 우선순위 순서 (중요한 순서부터)
 
| 순위  | 연산자                              | 의미       | 설명          |
| --- | -------------------------------- | -------- | ----------- |
| 1   | 괄호 `()`                          | 그룹 묶기    | 가장 먼저 계산    |
| 2   | 거듭제곱 `**`                        | 제곱       | 곱하기보다 먼저 계산 |
| 3   | 곱셈/나눗셈/몫/나머지 `*`, `/`, `//`, `%` | 곱하거나 나누기 | 덧셈/뺄셈보다 우선  |
| 4   | 덧셈/뺄셈 `+`, `-`                   | 더하고 빼기   | 가장 마지막 계산   |
✅ 외우기 쉽게!!  괄호( ) → 거듭제곱 `**` → 곱·나·몫·나 `*,/,//,%` → 덧·뺄 `+ -`

---
</>예시코드:  아래 코드를 연산하면? (곱셈이 먼저)
```python
result = 2 + 3 * 4
print(result) # 14
```

</>예시코드:  그러나 괄호를 사용하면? (괄호로 우선순위 변경)
```python
result = (2 + 3) * 4
print(result) # 20
```

</>예시코드: 괄호 없이 복합 연산 (다양한 연산 섞여 있을 때)
```python
result = 2 + 3 * 4 - 1 ** 2 
# 1**2 -> 1
# 3*4 -> 12
# 2+12-1 -> 13

print(result) # 13
```

🔍 해설:
1. `**` (거듭제곱) → 가장 먼저
2. `*`, `/`, `//`, `%` (곱셈, 나눗셈 등)
3. `+`, `-` (덧셈, 뺄셈)


</>예시코드: 괄호를 이용해 우선순위 변경
```python
result = (2 + 3) * (4 - 1) ** 2
# (2 + 3) = 5
# (4 - 1) = 3
# 5 * (3 ** 2) = 5 * 9 = 45

print(result) # 45
```

✅ 연산자 우선순위가 헷갈리면 **“무조건 괄호로 의도를 명확히”** 하는 습관을 들이는 것이 실무에서도 가장 안전합니다.

---
### 🔹 예외(에러)와 예외 처리의 기본 개념 

✅ 예외(Exception)란?
	프로그램을 실행할 때 예상치 못한 에러(오류)가 발생할 수 있는데,  그때 프로그램이 멈추지 않고 안전하게 처리할 수 있도록 만드는 기능입니다.

- 0으로 나누기
- 정의되지 않은 변수를 사용
- 리스트에 없는 인덱스 접근
- 문자열을 숫자로 바꾸려다 실패 등
    
이런 상황에서 파이썬은 **예외 객체**를 만들고, 기본적으로 프로그램을 멈추게 합니다.

✅ try–except 기본 구조
```python
try:
    # 에러가 발생할 가능성이 있는 코드
except 예외이름:
    # 에러가 발생했을 때 실행할 코드
```

---
❌ 대표적인 에러 상황  예시

</>예시코드:  0으로 나누기
```python
print(10 / 0)
```

❌ 🖨️ 출력결과: (Error발생)
```python
ZeroDivisionError: division by zero
```

🔍 해설:
	10을 0으로 나누려고 하면 수학적으로 정의되지 않아서 
	**ZeroDivisionError** 에러가 발생합니다.

---
⭕ 안전하게 처리하려면? 예외처리를 해줍니다.

</>예시코드:  0으로 나누기 안전하게 처리하기
```python
	try: # 이 안의 코드를 실행해본다(시도할때)
	    print(10 / 0) # ❌ 여기서 오류 발생 (ZeroDivisionError)
	except ZeroDivisionError: #예외일때
	    print("0으로 나눌 수 없습니다.") # 예외 메시지를 출력
```

🖨️ 출력결과:
```python
0으로 나눌 수 없습니다.
```

🔍 해설:
	`try` 블록에서 에러가 발생하면,
	`except` 블록으로 넘어가서
    에러 메시지를 출력하고 프로그램은 멈추지 않습니다!

---
◽ `TypeError` 예외
	`TypeError`는 **자료형(타입)이 맞지 않을 때** 발생합니다.

</>예시코드: 숫자와 문자열 연산 에러
``` python
	# 숫자와 문자열은 연산할수 없어요.
	print("hi" + 3)  # TypeError
	
	# 예외처리로 에러 해결
	try:
		print("hi" + 3)
	except TypeError:
		print("숫자와 문자열은 연산할수 없어요.")
```

🖨️ 출력결과:
```python
TypeError: can only concatenate str (not "int") to str
```

💡 기억하기
- `"hi"`는 `str`, `3`은 `int`
- `str + int`는 정의되지 않았기 때문에 `TypeError`

---
###### ◽ 주요 내장 예외 종류 한 번에 보기 
| 코드                         | 에러 종류               | 설명                              |
| -------------------------- | ------------------- | ------------------------------- |
| `print(1 / 0)`             | `ZeroDivisionError` | 0으로 나누면 안 돼요! (수학에서도 정의되지 않음)   |
| `print("hi" + 3)`          | `TypeError`         | 문자열과 숫자는 서로 더할 수 없어요 (타입이 안 맞음) |
| `int("abc")`               | `ValueError`        | 문자 `"abc"`는 숫자로 바꿀 수 없어요        |
| `print(x)`                 | `NameError`         | `x`라는 변수를 만들지 않았는데 쓰려고 해서 오류예요  |
| `lst = [1]; print(lst[5])` | `IndexError`        | 리스트에 5번째 값이 없어요 (범위 밖)          |
| `d = {}; print(d["key"])`  | `KeyError`          | 딕셔너리에 `"key"`라는 항목이 없어요         |

◽ 디버깅용: Exception as e 패턴
	에러가 정확히 **어떤 종류**인지, **메시지가 뭔지**를 출력해서 보고 싶을 때 사용합니다.
```python
	try:
	    print("hi" + 3)
	except Exception as e:
	    print("에러 종류:", type(e).__name__)
	    print("에러 메시지:", e)
```

🖨️ 출력결과:
```python
	에러 종류: TypeError
	에러 메시지: can only concatenate str (not "int") to str
```

💡 이렇게 하면 어떤 에러든 에러 이름과 메시지를 자동으로 확인할 수 있어요!

---
</> 예시코드:
```python
try:
    my_dict = {"name": "Alice"}
    print(my_dict["age"])  # 존재하지 않는 키
except Exception as e:
    print("에러 종류:", type(e).__name__)
    print("에러 메시지:", e)
```

🖨️ 출력결과:
```python
에러 종류: KeyError
에러 메시지: 'age'
```

🔍 해설: 
	`my_dict`에 `"age"`라는 키가 없어서 `KeyError`가 발생합니다.

---
</> 예시코드:
```python
try:
	# 중첩 딕셔너리(nested dictionary) 구조 
    data = {
	    "user": {
	        "name": "Tom",
	        "score": 100
	    }
}
# data = {"user": {"name": "Tom", "score": 100}} 접을수도 있어요.
    
    average = data.get("user").get("total") / 0  # None을 나누기
except Exception as e:
    print("에러 종류:", type(e).__name__)
    print("에러 메시지:", e)  
```

💡 `.get("user").get("total")`은:
	`.get("user")`는 `"user"` 키에 있는 값을 가져옵니다.  
    → `{"name": "Tom", "score": 100}`
    그다음 `.get("total")`은 위 딕셔너리 안에서 `"total"`이라는 키를 찾습니다.  
    → 그런데 `"total"`은 존재하지 않으니까 `None`이 반환됩니다.

💡 None / 0 에러발생:
	`one`은 숫자가 아니에요.
	서로 타입이 같지 않으므로 TypeError가 발생됩니다.

🖨️ 출력 결과:
```bash
에러 종류: TypeError
에러 메시지: unsupported operand type(s) for /: 'NoneType' and 'int'
```

💡 왜 `ZeroDivisionError`가 아니고 `TypeError`인가요?
	0으로 나누면 `ZeroDivisionError`가 납니다.
	그런데 여기서는 먼저 `"total"` 키가 없어서 `None`이 나오고, 그걸 나누려고 했기 때문에,  "0으로 나누기 전에" 먼저 `None / 0`이라는 잘못된 연산이 발생합니다. 그래서 먼저 발생한 `TypeError`가 발생된 겁니다.

---
◽ 에러 확인후 예외 발생 없이 ⭕ 안전하게 처리하는 코드: (조건문으로)
```python
# 중첩 딕셔너리 구조
data = {
    "user": {
        "name": "Tom",
        "score": 100
    }
}

# total 키가 있는지 먼저 확인하고 처리
user_info = data.get("user", {})  # user 키가 없어도 에러 안 나게 빈 딕셔너리 기본값
total = user_info.get("total")    # total 키가 없으면 None

# None이 아닌 경우에만 나눗셈 수행
if total is not None and total != 0:
    average = total / 1  # 또는 다른 숫자
    print("평균:", average)
else:
    print(" 키의 값이 없습니다.")
```

◽ 에러 확인후 예외 발생 없이 안전하게 처리하는 코드: (try-except)
```python
data = {
    "user": {
        "name": "Tom",
        "score": 100
    }
}

try:
	# try 블록에는 에러가 발생할 가능성이 있는 코드를 적습니다.
    # total이 없으면 None이 되고, 나눗셈 시 에러 발생 가능
    total = data["user"]["total"]
    average = total / 1  # 또는 다른 숫자
    print("평균:", average)

except KeyError:
    print("total 키가 없습니다.")

except TypeError:
    print("total 값이 None이어서 계산할 수 없습니다.")

except ZeroDivisionError:
    print("0으로 나눌 수 없습니다.")
```

---
조건문(`if`)과 예외처리(`try-except`)는 목적이 다릅니다.  
파이썬에서는 "예상할 수 있는 상황은 `if`, 예상할 수 없는 에러는 `try-except`"로 구분해서 사용하는 것이 가장 바람직합니다.

1️⃣ 조건문으로 처리 (예방 중심)
```python
user = {"name": "Tom"}

if "age" in user:
    print("나이:", user["age"])
else:
    print("나이 정보 없음")
```

✅ **결과**: 안전하게 실행됨  
	`"age"` 키가 없더라도 미리 확인해서 에러 없음

---
2️⃣ 예외처리로 처리 (복구 중심)
```python
user = {"name": "Tom"}

try:
    print("나이:", user["age"])
except KeyError:
    print("나이 정보 없음")
```

✅ **결과**: 똑같이 잘 실행됨  
	하지만 `KeyError`가 **실제로 발생한 후에** 처리함

---
◽ try 블록에는 에러가 발생할 가능성이 있는 코드를 적습니다.

</> 하나더 예시
```python
try:
    x = int("10")     # 정수 변환 → 성공
    y = int("hello")  
    # 문자열을 숫자로 변환 → 💥 여기서 ValueError 발생 가능
    
    result = x / y  # 이 줄은 실행되지 않음 (앞에서 에러났으므로)
    
except ValueError:
    print("숫자로 변환할 수 없습니다.")
```

---
### 🔹 문자열 연산자와의 우선순위 비교

❓  문자열 연산 vs 숫자 연산, 왜 주의해야 할까?
	파이썬은 **왼쪽에서 오른쪽**으로 연산을 진행하며,  
	덧셈(+) 연산자는 숫자용 + 문자열용이 각각 다르게 정의되어 있습니다.
	따라서 문자열과 숫자를 섞은 계산식에서는 연산 순서나 괄호가 없으면 예상치 못한 에러가 발생할 수 있습니다.

🔍 해설: 
	문자열과 숫자는 절대 자동으로 변환되지 않습니다.
	문자열 + 숫자 + 문자열처럼 혼합된 수식에서는 왼쪽부터 순서대로 평가되므로,  `str()`, 괄호, f-string 같은 처리 없이는 **TypeError**가 자주 발생합니다.

---
❌ </>예시코드:  형변환 없이 문자열 + 수식 연결 시도
```python
name = "Eve"
score1 = 85
score2 = 90

print("학생: " + name + " / 총점: " + score1 + score2 + "점")
```

❌ 🖨️ 출력결과:
```python
TypeError: can only concatenate str (not "int") to str
```

🔍 해설:
	`"학생: " + name` → `"학생: Eve"`까지는 문제 없습니다.
	`"학생: Eve" + score1` → 여기서 `str + int` 시도 → TypeError
	파이썬은 `score1 + score2`를 먼저 계산하지 않고, 왼쪽부터 차례대로 연산합니다.

---
</>예시코드: 올바른 형변환 후 문자열 연결 (복합 구조)
``` python
name = "Eve"
score1 = 85
score2 = 90

print("학생: " + name + " / 총점: " + str(score1 + score2) + "점")
```

🖨️ 출력결과:
```python
학생: Eve / 총점: 175점
```

🔍 해설:
	`score1 + score2`를 먼저 계산한 뒤, `str()`로 형변환
	이후 문자열끼리 연결되므로 오류 없이 실행

---
❌ 잘못된 예시코드: 문자열 + 숫자 혼합 (괄호 없이)
```python
# 잘못된 코드: 문자열 + 정수 덧셈 → 오류 발생
price = 1200
tax = 100

# 이 줄에서 오류 발생: str + int + int + str → 순서대로 진행되다가 에러
print("총 결제 금액: " + price + tax + "원")
```

🖨️ 출력 결과:
`TypeError: can only concatenate str (not "int") to str`

---
</> 수정 예시 1: `str()`로 형변환
```python
# 수정 코드 1: 숫자 합산 후 문자열로 변환
price = 1200
tax = 100

total = price + tax
print("총 결제 금액: " + str(total) + "원")
```

🖨️ 출력 결과:
```python
총 결제 금액: 1300원
```

---
</> 수정 예시 2: `f-string` 사용
```python
# 숫자 합산 후 문자열로 변환
price = 1200
tax = 100

print(f"총 결제 금액: {price + tax}원")
```

🖨️ 출력 결과:
```python
총 결제 금액: 1300원
```

---
📝 문제1] 다음 코드를 실행하면 어떤 결과가 나올까요?
```python
x = 10 
y = 5  

print("결과: " + x - y)
```

✅ 정답:
```python
TypeError: can only concatenate str (not "int") to str
```

🔍 해설:
- `"결과: " + x` 에서 먼저 `str + int` 연산 시도 → **TypeError**
- 파이썬은 왼쪽부터 순서대로 계산하므로, `x - y`가 먼저 수행되지 않음
---
📝 문제1]  수정 예시 (2가지)
```python
x = 10 
y = 5  

# 방법 1: 수식 먼저 계산 후 str() 변환
print("결과: " + str(x - y))

# 방법 2: f-string 활용
print(f"결과: {x - y}")
```

---
📝 문제2] 다음 코드가 출력되도록 수정하세요.
```python
price = 1500
tax = 150

# 원하는 출력: 총액: 1650원
print("총액: " + price + tax + "원")
```

✅ 정답 코드:
```python
# 방법 1
print("총액: " + str(price + tax) + "원")

# 방법 2 (권장)
print(f"총액: {price + tax}원")
```


🖨️ 출력 결과:
```python
총액: 1650원
```

🔍 해설:
- `price + tax`는 먼저 계산해도 되지만, `str`로 감싸지 않으면 `"총액: " + 숫자`에서 TypeError
- f-string은 자동 형변환 덕분에 더 간결하고 안전함

---
📝 문제3] 아래 코드를 실행하면 오류가 발생합니다. 오류 원인을 파악하고, `총점이 150점 이상이면 "우수"`를 출력하도록 수정하세요.
```python
student = {
    "name": "Mina",
    "math": 80,
    "english": 75
}

if student["math"] + student["english"] >= 150:
    print("이름: " + student["name"] + ", 총점: " + student["math"] + student["english"])
```

✅ 정답 코드:
```python
total = student["math"] + student["english"]

if total >= 150:
    print(f"이름: {student['name']}, 총점: {total} → 우수")
```

🖨️ 출력 결과:
```python
이름: Mina, 총점: 155 → 우수
```

🔍 해설:
- `"총점: " + student["math"]`는 `str + int` → **TypeError**
- `총점`을 미리 계산하고 문자열 포맷팅으로 출력해야 오류가 없음
- f-string 사용 시 훨씬 간결하고 안전하게 출력 가능

---
### 🔹  중간 정검 문제를 함께 풀이해봅시다.

📝 문제1] 나머지 연산자 `%` 를 활용하여  짝수 와  홀수 판별하세요.

🌟힌트: 
	어떤 수를 2로 나눴을 때 나머지가 
	0이면 짝수 1이면 홀수 입니다.

✅ 정답:
```python
num = int(input("숫자를 입력하세요: "))

if num % 2 == 0:
    print("짝수입니다.")
else:
    print("홀수입니다.")
```

---
📝 문제2] 요일 계산하기를 만들어봅니다.
	요일은 7일마다 반복되기 때문에, 날짜 계산할 때 % 7을 활용합니다.
	아무리 많은 날짜를 더해도 % 7을 하면 요일로 환산할 수 있어요!

📘 사전지식:
어떤수라도 7로 나눈 나머지는 0~6사이가 나옵니다.
리스트에 순번은 0부터 시작합니다. 그래서 수요일은 3번이예요.
📋 요일 번호표로 확인해봅니다.

| 요일  | 숫자  |
| --- | --- |
| 일요일 | 0   |
| 월요일 | 1   |
| 화요일 | 2   |
| 수요일 | 3   |
| 목요일 | 4   |
| 금요일 | 5   |
| 토요일 | 6   |
✅ `index()` 함수의 정확한 역할
	`index()` 함수는 어떤 값이 "어느 위치(인덱스)에 있는지"를 찾아주는 함수입니다.  즉, 값 → 위치 번호를 알려주는 함수입니다.

📖 Syntax 인덱스 함수의 기본문법
```python
시퀀스.index(찾고자 하는 값)
```

- 시퀀스: 문자열 리스트 튜플등 여러 값을 순서대로 나열한 자료형을 통틀어 부르는 공통된 분류 이름
- 반환값: 해당 값이 처음 나오는 인덱스 번호 (0부터 시작)

✅ 정답:
```python
# 요일을 숫자로 매칭하는 리스트
weekdays = ["일", "월", "화", "수", "목", "금", "토"]

# 사용자 입력
today_name = input("오늘은 무슨 요일인가요? (예: 월, 화, 수): ")
after_days = int(input("며칠 후를 계산할까요? (예: 5): "))

# 요일 계산
today = weekdays.index(today_name) # 일~토 문자열
day_index = (today + after_days) % 7 # 0~6

# 결과 출력
print(f"오늘은 {weekdays[today]}요일입니다.")
print(f"{after_days}일 후는 {weekdays[day_index]}요일입니다.")
```

🖨️ 출력결과:
```python
오늘은 무슨 요일인가요? (예: 월, 화, 수): 목
며칠 후를 계산할까요? (예: 5): 5
오늘은 목요일입니다.
5일 후는 화요일입니다.
```

공학용 계산기 mod(modulo)를 사용한다.

---
📝 문제3 응용문제]  이번에는 직접 숫자를 입력받고 요일을 계산해 봅시다.
아래와 같이 직접 숫자를  입력하여  결과가 출력되게 수정하세요.

- 오늘 요일을 숫자로 입력하세요 (일:0, 월:1, 화:2, 수:3, 목:4, 금:5, 토:6):
- 며칠 뒤를 알고 싶나요? 숫자로 입력하세요:

🖨️ 출력결과:
```python
오늘은 수요일 입니다.
5일 뒤는 월요일 입니다.
```

✅ 정답:
```python
# 요일을 숫자로 매칭하는 리스트
weekdays = ["일", "월", "화", "수", "목", "금", "토"]

# 사용자로부터 오늘 요일 입력받기
today = int(input("오늘 요일을 숫자로 입력하세요 (일:0, 월:1, 화:2, 수:3, 목:4, 금:5, 토:6): "))
after_days = int(input("며칠 뒤를 알고 싶나요? 숫자로 입력하세요: "))

# (today + after_days) % 7 계산
day_index = (today + after_days) % 7

# 결과 출력
print(f"오늘은 {weekdays[today]}요일 입니다.")
print(f"{after_days}일 뒤는 {weekdays[day_index]}요일 입니다.")
```

🖨️ 사용자 입력:
```python
오늘 요일을 숫자로 입력하세요 (일:0, 월:1, 화:2, 수:3, 목:4, 금:5, 토:6): 3
며칠 뒤를 알고 싶나요? 숫자로 입력하세요: 13
```

🖨️ 출력결과:
```python
오늘은 수요일 입니다.
13일 뒤는 화요일 입니다.
```

❓ 왜 `% 7`을 쓰는가?
	요일은 7개만 반복되죠? 그래서 숫자가 아무리 커도 `% 7`을 하면 0~6 사이의 숫자로 다시 돌아올 수 있어요.
	이것이 바로 "나머지 연산자 `%`를 이용한 순환 처리"입니다.

📌 즉, `8 % 7 = 1` → 5일 뒤는 "월요일"이라는 의미예요!
