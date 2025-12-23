# 🔹 if 조건문을 효율적으로 사용하기
	if, elif, else는 조건에 따라 코드를 실행하는 기본 구조지만,
	불필요하게 중복되거나 비효율적인 코드가 되지 않도록 조건문을 간결하
	고 명확하게 작성하는 습관이 중요합니다.

학점을 평가하는 조건문의 예시입니다. 0점에서  4.5점까지 재미요소를 넣어 표현된 것입니다. 

⛔ </> 예시코드:  비효율적인 조건문 (중복과 불필요한 조건, 순서 뒤죽박죽)
``` python
score = float(input("학점 입력> "))

if score >= 4.5:
    print("신")
elif score >= 4.2 and score < 4.5:
    print("교수님의 사랑")
elif score < 4.2 and score >= 3.5:
    print("현 체제의 수호자")
elif score >= 2.8 and score < 3.5:
    print("일반인")
elif score >= 2.3 and score < 2.8:
    print("일탈을 꿈꾸는 소시민")
elif score >= 1.75 and score < 2.3:
    print("오락문화의 선구자")
elif score < 1.75 and score >= 1.0:
    print("불가촉천민")
elif score >= 0.5 and score < 1.0:
    print("자벌레")
elif score < 0.5 and score > 0:
    print("플랑크톤")
elif score == 0:
    print("시대를 앞서가는 혁명의 씨앗")
```

이 조건문은 중복된 비교(`>=`와 `<`)를 반복적으로 사용하여 가독성이 떨어지고, 조건 간 우선순위가 명확하지 않으며, 불필요하게 긴 코드가 되어 비효율적입니다.
→ `elif score >= X:` 형태로 하한값만 비교하며 순차적으로 나열하면 훨씬 간결하고 효율적으로 바꿀수 있습니다.

⛔ 문제점:
	`and` 조건을 계속 사용하여 코드가 장황함
	`>=`와 `<` 조건이 반복되어 가독성이 떨어짐
	순서도 일부 중복되어 비효율적

⭕ </> 예시코드: 효율적으로 변경된 코드
```python
score = float(input("학점 입력> "))

if score == 4.5:
    print("신")
elif score >= 4.2:
    print("교수님의 사랑")
elif score >= 3.5:
    print("현 체제의 수호자")
elif score >= 2.8:
    print("일반인")
elif score >= 2.3:
    print("일탈을 꿈꾸는 소시민")
elif score >= 1.75:
    print("오락문화의 선구자")
elif score >= 1.0:
    print("불가촉천민")
elif score >= 0.5:
    print("자벌레")
elif score > 0:
    print("플랑크톤")
else:
    print("시대를 앞서가는 혁명의 씨앗")
```

---
💡 개선과정:

📚 1단계: 만점이 4.5 이므로 수정함
if score >= 4.5:
if score == 4.5:
 
📚 2단계: 위에서 score == 4.5는 이미 걸렀기 때문에 score < 4.5는 생략 가능. 
elif score >= 4.2 and score < 4.5
elif score >= 4.2

📚 3단계: 위에서 4.2 이상은 이미 처리됐으므로 score < 4.2는 필요 없음.
elif score < 4.2 and score >= 3.5
elif score >= 3.5:

📚 4단계: 위에서 3.5 이상은 이미 걸러졌으므로 score < 3.5는 생략.
elif score >= 2.8 and score < 3.5
elif score >= 2.8:

📚 5단계: 위에서 2.8 이상은 이미 처리되어 있어서 마찬가지로 상한 조건 제거.
elif score >= 2.3 and score < 2.8
elif score >= 2.3:

📚 6단계: 위에서 2.3 이상은 이미 처리됨 → 상한 생략.
elif score >= 1.75 and score < 2.3
elif score >= 1.75:

📚 7단계: 1.75 이상은 위에서 걸러졌으니, 하한 조건만 비교하면 충분.
elif score < 1.75 and score >= 1.0
elif score >= 1.0:

📚 8단계: 1.0 이상은 위에서 이미 걸러졌음 → score < 1.0 생략 가능.
elif score >= 0.5 and score < 1.0
elif score >= 0.5:

📚 9단계: 위에서 0.5 이상은 처리됐으므로 score < 0.5는 생략.
elif score < 0.5 and score > 0
elif score > 0:

📚 10단계: 마지막 조건은 0점밖에 없으므로 else로 간단히 처리 가능.
elif score == 0
else:

---
📝 문제1] 아래 코드의 문제점을 파악하고,  
`if`, `elif`, `else`를 활용하여 효율적으로 수정해보세요.

❌ 비효율적 코드:
```python
temp = 30 

if temp > 25:     
	print("더움") 
if temp > 15:     
	print("조금 따뜻함") 
else:     
	print("쌀쌀함")
```

✅ 정답 코드:
```python
temp = 30  

if temp > 25:     
	print("더움") 
elif temp > 15:     
	print("조금 따뜻함") 
else:     
	print("쌀쌀함")
```

🖨️ 출력 결과:
```python
더움
```

🔍 해설:
- 위에서 이미 `True`인 조건을 만족하면 아래 조건은 검사하지 않아야 함
- `if → elif → else` 구조로 효율성 향상
---
📝 문제2] 논리 연산자를 활용하여 중복 코드를 줄이세요.

❌ 중복 코드:
```python
age = 22  
if age >= 19 and age <= 29:     
	print("청년") 
elif age >= 30 and age <= 39:     
	print("장년")
```

✅ 정답 코드:
```python
age = 22  

if 19 <= age <= 29:     
	print("청년") 
elif 30 <= age <= 39:     
	print("장년")
```

🖨️ 출력 결과:
```python
청년
```

🔍 해설:
- 조건 범위 표현은 `and`보다 `<= age <=` 형식을 사용하면 더 간결하고 읽기 쉬움

---
📝 문제2] 아래 코드에서 조건 순서를 바꿔 효율적으로 작성하세요.

❌ 비효율적 코드:
```python
score = 100  

if score >= 60:     
	print("합격") 
elif score >= 90:     
	print("우수")
```

✅ 정답 코드:
```python
score = 100  

if score >= 90:     
	print("우수") 
elif score >= 60:     
	print("합격")
```

🖨️ 출력 결과:
```python
우수
```

🔍 해설:
- 더 구체적인 조건을 위에 배치해야, 그 조건이 먼저 걸려서 정확한 판별 가능
- 그렇지 않으면 일반 조건에 먼저 걸려서 우선순위가 무시됨

---
# 🔹 False로 변환되는 값
	파이썬에서 if 조건:처럼 조건문을 사용할 때,  
	조건이 False로 간주되는 값들은 자동으로 실행되지 않아요.  
	이 값들은 논리적으로 거짓(False)처럼 취급되기 때문에  
	따로 == False라고 쓰지 않아도 if문에서 무시됩니다.

파이썬에서는 아래 값들이 자동으로 `False`로 간주됩니다:
```python
None, 0, 0.0, "", [], {}, set()
```

| 값       | 설명                 |
| ------- | ------------------ |
| `None`  | 아무 값도 없음 (null 개념) |
| `0`     | 숫자 0               |
| `0.0`   | 실수형 0.0            |
| `""`    | 빈 문자열              |
| `[]`    | 빈 리스트              |
| `{}`    | 빈 딕셔너리             |
| `set()` | 빈 집합               |

</> 예시코드:  조건문에서 False로 취급되는 값들
```python
if "":
    print("빈 문자열입니다")  # 출력 안 됨

if 0:
    print("숫자 0입니다")      # 출력 안 됨

if []:
    print("빈 리스트입니다")   # 출력 안 됨

if None:
    print("None입니다")       # 출력 안 됨
```

🖨️ 출력결과:
```python
(아무것도 출력되지 않음)
```

🔍 해설:
- 위 예제에서는 모두 False로 간주되기 때문에  
    `if` 조건이 만족되지 않아 어떤 print 문도 실행되지 않음
---
◽ `None` – 아무것도 없음

```python
	value = None
	
	if value:     
		print("값이 있습니다.") 
	else:     
		print("값이 없습니다.")  # 출력
```

📌 **설명:** `None`은 값이 없다는 의미이며 조건문에서는 `False`로 처리돼요.

---
◽ `0` – 숫자 0
```python
	num = 0  
	
	if num:     
		print("양수입니다.") 
	else:     
		print("0 또는 음수입니다.")  # 출력
```

 🔍 해설:  숫자 0은 `False`, 1이나 -1 같은 숫자는 `True`예요.
 
---
◽ `0.0` – 실수형 0
```python
	num = 0.0 
	 
	if num:     
		print("실수값이 있습니다.") 
	else:     
		print("0.0입니다.")  # 출력
```

 🔍 해설: 정수형 0과 마찬가지로 실수형 0도 `False`로 간주돼요.

---
◽ `""` – 빈 문자열
```python
	text = ""  
	
	if text:     
		print("문자열이 있습니다.") 
	else:     
		print("빈 문자열입니다.")  # 출력
```

 🔍 해설: 문자가 하나라도 있으면 `True`, 아무 글자도 없으면 `False`예요.

---
◽ `[]` – 빈 리스트
```python
	items = [] 
	 
	if items:     
		print("리스트에 값이 있습니다.") 
	else:     
		print("빈 리스트입니다.")  # 출력
```

 🔍 해설: 리스트에 요소가 하나라도 있으면 `True`, 아무것도 없으면 `False`예요.
 
---
◽ `{}` – 빈 딕셔너리
```python
	person = {}  
	
	if person:     
		print("딕셔너리에 정보가 있습니다.") 
	else:     
		print("빈 딕셔너리입니다.")  # 출력
```

 🔍 해설: 키-값 쌍이 하나라도 있으면 `True`, 없으면 `False`.

---
◽ `set()` – 빈 집합

```python
	my_set = set()
	  
	if my_set:     
		print("집합에 값이 있습니다.") 
	else:     
		print("빈 집합입니다.")  # 출력
```

 🔍 해설: 요소가 있는 집합은 `True`, 비어 있으면 `False`.
 
---
◽ `if not x:`는 어떤 의미일까?

🧠 핵심 개념
- `if x:` → `x`가 참(True)일 때 실행됩니다.
- `if not x:` → `x`가 거짓(False)처럼 간주될 때 실행됩니다.
즉, `x`가 비었거나 0이거나 None일 때 실행됨
```python
print(not True)    # False
print(not False)   # True

print(not 0)       # True → 0은 False → not False = True
print(not 123)     # False → 123은 True → not True = False

print(not "")      # True → 빈 문자열은 False → not False = True
print(not "hi")    # False → 문자열이 있으면 True → not True = False
```

</> 예시코드:
``` python
# x가 True로 간주되면 실행됨
x = 10
if x:
	print("실행됨") # True로 실행됨

# 그러나 not을 쓰면 False가 실행됨
x = 10
if not x:
	print("True")
else:
	print("False") # False가 실행됨
```

🧠 쉽게 말하면: 
	if x: 는 x가 비어있지 않으면 실행되고
	if not x: 는 x가 비어있으면 실행됩니다.

---
📝 문제1] 사용자가 입력한 문자열이 있는지 확인하여 입력값이 있으면 "입력한값" 을 출력하고, 없으면 "입력하지 않았습니다" 를 출력하세요.  

✨ 힌트:
	`input()`은 문자열을 반환함.
	사용자가 아무 것도 입력하지 않으면 `value`는 `""` (빈 문자열)
    `if value:`는 `False`로 평가되므로 `else`가 실행됨.

```python
value = input("값을 입력하세요: ")  # (예: 그냥 Enter만 누른 경우)

if value:
    print("입력한 값:", value)
else:
    print("입력하지 않았습니다")
```

✅ 정답:
```python
입력하지 않았습니다
```

---
📝 문제2] 다음 코드에서 `my_list`가 비어 있는 경우에만  "빈 리스트입니다"가 출력되도록 조건문을 완성하세요.

✅ 정답 코드:
```python
my_list = []

if len(my_list) == ___:
    print("빈 리스트입니다")
```

보기:
`1. -1`
`2. 0`
`3. 1`
`4. "0"`

✅ 정답: 2 

🔍 해설:
- `len(my_list)`는 리스트의 길이를 구함
- 빈 리스트의 길이는 `0`
- 따라서 `len(my_list) == 0`이면 빈 리스트임
---
📝 문제3] 리팩토링 
아래 코드는 리스트가 비어 있을 때 `"빈 리스트입니다"`를 출력합니다.  
이 코드를 더 간단한 조건문으로 바꾸려면 어떻게 해야 할까요?
```python
my_list = []

if len(my_list) == 0:
    print("빈 리스트입니다")
```

✏️ 조건문 한 줄만 바꿔보세요:
```python
my_list = []

if ____________________:     
print("빈 리스트입니다")
```

✅ 정답:
```python
if not my_list:     
```

🔍 해설:
- `my_list`가 비어 있으면 → `False`
- `not my_list` → `True`가 되어 조건식이 실행됨
- `len(my_list) == 0`과 같은 의미지만 더 간결하고 파이썬다운 표현

---
# 🔹 pass 키워드
	pass는 "아무 일도 하지 마라"는 뜻을 가진 파이썬의 특별한 키워드입니
	다. 주로 조건문, 반복문, 함수, 클래스 등을 작성할 때 아직 내용을 
	채우지 않았지만 문법 오류 없이 구조를 유지하고 싶을 때 사용해요.

🧾 보충설명:
	pass는 "나중에 코드를 채울 예정이야" 라고 알려주는 빈 자리 표시자입니다.

</> 예시코드:  조건만 만들고, 지금은 아무 행동도 하지 않을 때
``` python
age = 17

if age >= 19:
    print("성인입니다.")
else:
    pass  # 아직 미성년자일 때 할 행동은 정하지 않음
```

🖨️ 출력결과:
```python
(아무것도 출력되지 않음)
```

🔍 해설:
- `age = 17` → `if age >= 19`는 `False` → `else` 실행
- 하지만 `pass`는 실제로 아무 동작도 하지 않음
- 코드 구조는 완성되었지만, 로직은 미정일 때 임시로 채워놓는 용도

✅ 이것을 왜 사용하나?
- `if`, `for`, `while`, `def` 같은 블록문 뒤에는 반드시 코드가 들어가야 해요.
- 그런데 아직 코드를 안 짰거나, 나중에 짤 예정이면 에러가 납니다!
- 그래서 아무것도 하지 않겠다거나 나중에 작성할때 에러 방지를 위해 사용합니다.
---
❌ </> 에러 나는 코드 예시:
```python
if x > 0:
    # 아직 작성 안 했어요
```

⛔ 이런 식으로 코드 블록이 비어 있으면  `IndentationError` 또는 `SyntaxError`가 발생해요.

✔ 해결 방법:
```python
if x > 0:
    pass  # 지금은 아무 것도 안 하지만, 나중에 작성할게요!
```
---
◽ `pass`를 쓰는 3가지 대표 예시:

1️⃣ 조건문사용: 나중에 조건문을 작성할때
```python
number = int(input("정수 입력> "))

if number > 0:
    pass  # 양수일 때 나중에 할 일
else:
    pass  # 음수일 때 나중에 할 일
```

2️⃣ 함수에서 사용: 함수 이름부터 만들고 기능은 나중에 작성할때
```python
def calculate():
    pass
```

3️⃣ 반복문에서 사용: 반복구조는 짜놨지만 무엇을 반복할지 미정일때
```python
for i in range(5):
    pass
```

---
📝 문제1] 아래 코드에서 오류가 발생하지 않도록  수정하세요.

❌ 오류 발생 코드:
```python
if True:
```

✅ 정답 코드:
```python
if True:     
	pass
```

🖨️ 출력 결과:
```python
(아무 출력 없음, 오류도 없음)
```

🔍 해설:
- `if` 다음에는 반드시 실행할 코드가 있어야 함
- `pass`를 넣어 구조만 유지하면 문법 오류 없이 코드 작성 가능

---
📝 문제2] 함수를 선언하고 아직 구현될 값이 없을때 처리할수 있는 코드를 작성하세요. 단 출력 결과는 아무일도 일어나지 않습니다.

🖨️ 출력 결과:
```python
(아무 일도 일어나지 않음)
```

✅ 정답 코드:
```python
def greet():     
	pass
```

🔍 해설:
- `pass`를 사용하면 함수의 형식은 만들어 놓고 
    나중에 내용을 채워 넣을 수 있어요.

---
📝 문제3] 아래 코드를 실행하면 출력 결과는 어떻게 나올지 예상해 보세요.
```python
for i in range(3):     
	pass 
print("반복문 끝!")
```

✅ 정답:
```python
반복문 끝!
```

🔍 해설:
- 반복문을 돌기는 하지만 `pass` 때문에 실제로 아무 동작도 하지 않음
- 반복문은 3번 순회하되, 아무 일도 하지 않고 `"반복문 끝!"`만 출력됨

---
📝 문제4] 그렇다면 "반복문 끝!" 이라는 문장이 3회 출력되게 하려면 어떻게 해야 하는지 최대한 간단한 방식으로 수정해 보세요. 

🖨️ 출력 결과:
```python
반복문 끝!
반복문 끝!
반복문 끝!
```

✅ 정답:
```python
for i in range(3):      
    print("반복문 끝!")
```

---
# 🔹 raise NotImplementedError
	raise NotImplementedError는
	"아직 이 부분은 구현하지 않았어!" 라고 명시적으로 알려주는 에러 
	발생 명령입니다.  
	특히 함수나 클래스에서 내용을 나중에 작성할 예정일 때, 단순히 pass 
	대신 사용하면 개발 중이라는 것을 명확히 표시할 수 있어요.
	단, 삼항 연산자"에서는 가독성이 떨어지고 코드가 혼란스러워져서 
	권장하지 않습니다. 

🐣 쉽게 말하면:  
	`pass`는 "일단 넘어가자"**,  
	`raise NotImplementedError`는 "여기 구현 안 했으니 쓰면 안 돼!"

</> 예시코드:  조건문이 아직 완성되지 않았을 때
```python
x = input("기능을 선택하세요: ") 

if x == "1":
    print("기능 1 실행")
else:
    raise NotImplementedError("이 기능은 아직 구현되지 않았습니다.")
```

🖨️ 실행 결과:
```python
Traceback (most recent call last):   
... NotImplementedError: 이 기능은 아직 구현되지 않았습니다.
```

🔍 해설:
- `calculate_total()` 함수는 정의만 되어 있고, 아직 기능을 작성하지 않음
- 대신 `raise NotImplementedError`로 호출 시 의도적으로 오류 발생
- 이렇게 하면 개발 도중 실수로 이 함수가 실행되지 않도록 방지할 수 있어요

---
</> 예시코드: `pass` 사용 – "아무 것도 하지 않을 때"
```python
x = 10

if x > 0:
    pass  
    # 조건이 참일 때 아직 구현할 내용이 없거나, 일단 비워둘 때 사용
else:
    print("0 이하입니다.")
```

🔍 해설:
- `pass`는 구문적으로 필요한 자리만 채워주는 키워드입니다.
- 빈 블록 오류를 막기 위해 넣습니다.
- 예: 함수, 조건문, 클래스 등에서 임시로 "나중에 작성 예정"일 때
---
◽ `pass` 대신 `raise NotImplementedError`를 쓰는 상황은 언제일까요?

🔍 해설:
- `pass`: 그냥 넘어감 (오류 없음)
- `raise NotImplementedError`: 의도적으로 실행을 막고 오류 발생시킴

</> pass와 NotImplementedError 비교 예시:
```python
x = "beta"

# 1. pass 사용: 그냥 넘어감
if x == "alpha":
    print("알파 기능 실행")
elif x == "beta":
    pass  # 아직 구현 안 했지만 실행 중단은 원치 않음
else:
    print("기타 기능")

# 2. raise NotImplementedError 사용: 일부러 멈춤
if x == "alpha":
    print("알파 기능 실행")
elif x == "beta":
    raise NotImplementedError("베타 기능은 아직 구현되지 않았습니다.")
else:
    print("기타 기능")
```

👉 차이점:
- `pass`: 형식상 자리를 채우는 역할로 실행이 됩니다.
- `raise NotImplementedError`: 의도적으로 예외 발생, 실행 STOP 프로그램 중단

| 키워드                         | 역할       | 언제 사용?                       |
| --------------------------- | -------- | ---------------------------- |
| `pass`                      | 아무 동작 없음 | 틀만 잡아둘 때 (개발 예정)             |
| `raise NotImplementedError` | 강제 오류 발생 | 구현되지 않은 기능이 호출되지 않도록 막고 싶을 때 |

---
📝 문제1] 다음 중 `pass` 키워드에 대한 설명으로 올바른 것은?

1️⃣코드 실행을 강제로 중단시킨다.
2️⃣조건문을 반복 실행시킨다.
3️⃣아직 코드를 작성하지 않은 블록을 일단 넘어가게 해준다.
4️⃣오류가 발생했을 때 자동으로 예외를 처리한다.

✅ 정답: 3번

---
📝 문제2]  `raise NotImplementedError`의 주된 사용 목적은?

1️⃣ 특정 조건이 참일 때 반복을 멈추기 위해
2️⃣ 아직 구현되지 않은 기능을 실수로 호출하지 못하게 하기 위해
3️⃣ 프로그램을 빠르게 종료시키기 위해
4️⃣ 변수를 선언하지 않고도 사용할 수 있게 하기 위해

✅ 정답: 2번

---
📝 문제3]  아래 코드 실행 시 발생하는 결과는?
```python
option = "beta"

if option == "alpha":
    print("Alpha 기능 실행")
else:
    raise NotImplementedError
```

1️⃣ 아무 출력 없이 종료된다
2️⃣ `"Beta 기능 실행"`이 출력된다
3️⃣ `NotImplementedError` 예외가 발생한다
4️⃣ `pass`가 실행되어 넘어간다

✅ 정답: 3번

🧾 보충설명: 
- `option` 값이 `"alpha"`가 아니므로 `else` 블록이 실행됩니다.
- `else` 안에는 `raise NotImplementedError`가 있으므로 → 프로그램이 **강제 종료**되며 예외가 발생합니다.
- 이 예외는 "아직 구현되지 않은 코드"를 명확히 나타내는 용도로 사용됩니다.

---
# 💭직접 풀어보세요.

📝 문제1]  세 수 중 가장 큰 수 구하기
사용자로부터 정수 3개를 입력받아, 가장 큰 수를 출력하세요.  
단, 조건문만을 사용해야 하며, 내장 함수 `max()`는 사용하지 마세요.  
모두 같을 수도 있다는 조건은 무시해도 됩니다.

✨ 힌트:
- 두 수를 먼저 비교한 후, 결과와 남은 하나를 다시 비교하면 됩니다.
- `if`만 사용할 수도 있고, 삼항 연산자(`a if 조건 else b`)를 활용해도 됩니다.

🖨️ 출력 예시
```python
입력 예:  
10  
22  
17  
출력: 가장 큰 수는 22입니다.
```

✅ 정답 코드
```python
a = int(input())
b = int(input())
c = int(input())

# 두 수를 먼저 비교한 뒤, 나머지 수와 다시 비교
max_num = a if a > b else b
max_num = max_num if max_num > c else c

print(f"가장 큰 수는 {max_num}입니다.")
```

🔍 해설
- 첫 비교: `a > b` → 더 큰 수를 `max_num`에 저장
- 두 번째 비교: `max_num > c` → 최종적으로 가장 큰 수 추출
- `삼항 연산자` 두 번 사용해서 깔끔하게 비교 로직 구성
---
📝 문제2] 두 수를 입력받아 둘 다 짝수일 때만 "통과"를 출력하세요.

✨ 힌트:
	`a % 2 == 0`은 짝수인지 확인하는 조건입니다. 논리 연산자 `and`를 활용해 보세요.

🖨️ 출력 예시: (입력값: 4, 6)
```python
통과
```

✅ 정답 코드:
```python
a = int(input("첫 번째 숫자: "))
b = int(input("두 번째 숫자: "))

if a % 2 == 0 and b % 2 == 0:
    print("통과")
else:
    print("실패")
```

🔍 해설:
- 두 수 모두 짝수이면 `"통과"` 출력.
- 하나라도 홀수면 `"실패"` 출력.

---
📝 문제3] 숫자를 입력받아 0보다 크고 100 이하이면 "적절"을 출력하고, 그렇지 않으면 "범위 초과"를 출력하세요.

✨ 힌트:
`0 < num <= 100` 형태의 조건식을 사용하세요.

🖨️ 출력 예시: (입력값: 105)
```python
범위 초과
```

✅ 정답 코드:
```python
num = int(input("숫자를 입력하세요: "))

if 0 < num <= 100:
    print("적절")
else:
    print("범위 초과")
```

🔍 해설:
- `0보다 크고`, `100 이하`일 때만 `"적절"` 출력.
- 그 외에는 `"범위 초과"` 출력.

---
📝 문제4] 세 개의 수를 입력받아 모두 10 이상일 때만 "모두 통과"를 출력하세요.
 
✨ 힌트:
- `and` 연산자를 3개 연결해서 조건을 만들어 보세요.

🖨️ 출력 예시:
```python
실패
```

✅ 정답 코드:
```python
x = int(input("첫 번째 수: "))
y = int(input("두 번째 수: "))
z = int(input("세 번째 수: "))

if x >= 10 and y >= 10 and z >= 10:
    print("모두 통과")
else:
    print("실패")
```

🔍 해설:
- 세 수 모두 `10 이상`이어야 `"모두 통과"` 출력.
- 하나라도 조건에 안 맞으면 `"실패"` 출력.
---
📝 문제5] 숫자를 입력받아 짝수면 "EVEN", 홀수면 "ODD"를 출력하는 프로그램을 삼항 연산자를 사용해 작성하세요.

✨ 힌트:
- `값1 if 조건 else 값2` 형태를 써보세요.

🖨️ 출력 예시: (입력값: 7)
```python
ODD
```

✅ 정답 코드:
```python
num = int(input("숫자 입력: "))

result = "EVEN" if num % 2 == 0 else "ODD"
print(result)
```

🔍 해설:
- `% 2 == 0`이면 `"EVEN"` 출력, 아니면 `"ODD"` 출력.
- 삼항 조건식을 사용하면 한 줄로 표현할 수 있음.

---
📝 문제6] 사용자에게 등급(A, B, C, 기타)을 입력받아 각 등급에 따라 메시지를 출력하세요. A: "최우수", B: "우수", C: "보통", 나머지: "재평가".

✨ 힌트:
- `if`, `elif`, `else`를 모두 활용하세요.

🖨️ 출력 예시: (입력값: D)
```python
재평가
```

✅ 정답 코드:
```python
grade = input("등급 입력(A, B, C 등): ")

if grade == "A":
    print("최우수")
elif grade == "B":
    print("우수")
elif grade == "C":
    print("보통")
else:
    print("재평가")
```

🔍 해설:
- 등급에 따라 다른 문장을 출력.
- `if-elif-else`를 사용한 조건 분기 연습 문제.

---
📝 문제7] 로그인 시스템
사용자에게 아이디와 비밀번호를 입력받고,  
아이디가 `"admin"`이고 비밀번호가 `"1234"`일 때만 `"접속 성공"`을 출력하고,  
그 외에는 `"접속 실패"`를 출력하도록 프로그램을 완성하세요.

 빈칸을 채우세요.
```python
user_id = input("아이디를 입력하세요: ")
password = input("비밀번호를 입력하세요: ")

if (__________ and __________):
    print("접속 성공")
else:
    print("접속 실패")
```

✨ 힌트:
- `user_id == "admin"`
- `password == "1234"`

🖨️ 출력 예시: 입력값: `admin`, `1234`
```python
접속 성공
```

✅ 정답 코드:
```python
user_id = input("아이디를 입력하세요: ")
password = input("비밀번호를 입력하세요: ")

if user_id == "admin" and password == "1234":
    print("접속 성공")
else:
    print("접속 실패")
```

🔍 해설:
- 두 조건을 `and`로 묶어서 **둘 다 일치**할 때만 접속 허용.
- 보안 인증의 기초 로직에 활용됨.
---
📝 문제8] 이벤트 참여 조건
행사에 참여할 수 있는 조건은 다음과 같다:
- 나이는 만 20세 이상이어야 하고
- 지역은 "서울" 또는 "부산"이어야 한다.  
    조건을 만족하면 `"이벤트 참여 가능"`, 그렇지 않으면 `"참여 불가"`를 출력하는 코드를 작성하세요.

읽고 조건 해석하는 문제
```python
age = int(input("나이를 입력하세요: "))
city = input("거주 도시를 입력하세요: ")

if _________________________________:
    print("이벤트 참여 가능")
else:
    print("참여 불가")
```

✨ 힌트:
- `age >= 20`
- `city == "서울"` 또는 `city == "부산"`

🖨️ 출력 예시: 입력값: `23`, `서울`
```python
이벤트 참여 가능
```

✅ 정답 코드:
```python
age = int(input("나이를 입력하세요: "))
city = input("거주 도시를 입력하세요: ")

if age >= 20 and (city == "서울" or city == "부산"):
    print("이벤트 참여 가능")
else:
    print("참여 불가")
```

🔍 해설:
- `and`와 `or`의 결합이 중요한 문제.
- 괄호로 `or` 조건을 묶지 않으면 논리 오류가 발생할 수 있음.
- 실제 회원가입 조건이나 필터링 조건에서도 사용되는 방식.

