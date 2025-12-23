```bash
# 기본 요청 라이브러리 (동기 방식)
pip install requests

# 비동기 방식 (성능이 중요할 경우)
pip install aiohttp aiodns
```

공공데이터를 사용하려면:
1. API 주소(URL) 를 확인하고
2. 인증키(서비스 키) 를 넣어서
3. 원하는 데이터를 요청하고
4. 받은 JSON 데이터를 파이썬으로 처리합니다

aiohttp는 비동기 HTTP 클라이언트/서버 프레임워크입니다. 주로 `asyncio` 기반의 비동기 웹 요청이나 비동기 웹 서버를 만들 때 사용합니다.

비동기 방식으로 HTTP 요청을 보내고 JSON 데이터를 받아오는 테스트코드:
Jupyter에서 테스트하기:
```python
import aiohttp
import json

async with aiohttp.ClientSession(connector=aiohttp.TCPConnector(ssl=False)) as session:
    async with session.get("https://dummyjson.com/products") as resp:
        j = await resp.json()
        print(json.dumps(j, indent=2))
```
- 실제 공공데이터 API나 REST API를 사용할 때도 이런 방식으로 데이터를 받아오게 됩니다.
- `aiohttp`는 빠른 처리를 원할 때 사용하는 비동기 방식이라 연습해보는 겁니다.
- `https://dummyjson.com`은 테스트용 API로, 실습할 때 서버에 피해 없이 자유롭게 테스트할 수 있어요.

---
공공체이터 포털에서 지난 날씨 데이터 가지고 오기
![[Pasted image 20250710185106.png]]

공공데이터포털
https://www.data.go.kr/data/15059093/openapi.do

검색어로 종관이라고 검색
![[Pasted image 20250710185416.png]]

![[Pasted image 20250710185526.png]]

활용신청:
![[Pasted image 20250710185707.png]]

활용신청:
![[Pasted image 20250710190040.png]]

전체코드:
```python
import requests

def get_weather_data():
    url = "http://apis.data.go.kr/1360000/AsosDalyInfoService/getWthrDataList"

    params = {
        "serviceKey": "여기에_본인_API_KEY",  # Decoding 키 권장
        "pageNo": 1,
        "numOfRows": 10,
        "dataType": "JSON",
        "dataCd": "ASOS",
        "dateCd": "DAY",
        "startDt": "20240101",
        "endDt": "20240110",
        "stnIds": "108",  # 서울(종관관측소)
    }

    response = requests.get(url, params=params, timeout=20)
    response.raise_for_status()  # ✅ 이제 200이므로 다시 넣어도 OK

    data = response.json()

    # API 자체 결과코드 확인 (00이면 정상)
    if data["response"]["header"]["resultCode"] != "00":
        print("❌ API 오류:", data["response"]["header"])
        return None

    item_list = data["response"]["body"]["items"]["item"]

    # 필요한 항목만 뽑아서 “리스트 안의 딕셔너리” 형태로 가공
    dict_list = []
    for item in item_list:
        item_dict = {
            "date": item.get("tm"),             # 날짜
            "min_temp": item.get("minTa"),      # 최저기온
            "max_temp": item.get("maxTa"),      # 최고기온
            "cloud_amount": item.get("avgTca"), # 평균 전운량
            "max_snow": item.get("ddMes"),      # 일 최심적설
            "rain": item.get("sumRn"),          # 일강수량
        }
        dict_list.append(item_dict)

    return dict_list


if __name__ == "__main__":
    weather_data = get_weather_data()

    if weather_data:
        print("✅ 가공된 데이터 개수:", len(weather_data))
        print("✅ 첫 번째 데이터 샘플:", weather_data[0])
        print("\n[앞에서 5개 출력]")
        for row in weather_data[:5]:
            print(row)
```


모듈 임포트 & 함수 정의
	`requests`는 파이썬에서 HTTP 요청을 보내기 위한 라이브러리입니다.  
	`get_weather_data()` 함수는 “종관(ASOS) 일자료”를 API에서 가져와 가공된 리스트로 반환합니다.
```python
import requests

def get_weather_data():
```


API 요청 URL 작성
	아래 URL은 기상청 종관(ASOS) 일자료 조회 서비스의 엔드포인트입니다.
```python
url = "http://apis.data.go.kr/1360000/AsosDalyInfoService/getWthrDataList"
```
![[Pasted image 20250710191109.png]]

일반 인증키를 잘 보관해 두세요:
![[Pasted image 20250710193701.png]]

요청 파라미터(params) 구성
	핵심: 서비스키(serviceKey)는 Decoding(일반) 키를 권장
	공공데이터포털에서 제공하는 서비스키는 보통 2종류가 있습니다.

- Decoding(일반) 키: 퍼센트(%)가 없는 원문 형태
- Encoding 키: `%2F`, `%3D` 같이 퍼센트 인코딩이 포함된 형태
    
`requests.get(..., params=params)` 방식에서는  
Decoding(일반) 키를 그대로 넣는 방식이 가장 안전합니다.

> Encoding 키를 그대로 넣으면 환경에 따라 `%`가 다시 인코딩되어 `%252F` 형태가 되면서 인증 실패(401)가 날 수 있습니다.  
> (실제로 이 문제를 경험했고, unquote로 해결했죠.)

```python
params = {
    "serviceKey": "여기에_본인_API_KEY",  # Decoding 키 권장
    "pageNo": 1,
    "numOfRows": 10,
    "dataType": "JSON",   # JSON 응답 받기
    "dataCd": "ASOS",     # 종관(ASOS)
    "dateCd": "DAY",      # 일자료
    "startDt": "20240101",
    "endDt": "20240110",
    "stnIds": "108",      # 서울(종관관측소)
}
```
![[Pasted image 20250710190502.png]]
	참고: 만약 Encoding 키만 있다면 `unquote(ENCODED_KEY)`로 “디코딩 형태”로 만든 뒤 `serviceKey`에 넣어야 안정적입니다.

---
이 코드는 모든 데이터를 돌려본 샘플 데이터 입니다
```python
    """
    {
        "response":{
            "header":{
                "resultCode":"00",
                "resultMsg":"NORMAL_SERVICE"
            },
            "body":{
                "dataType":"JSON",
                "items":{
                    "item":[
                    {
                        "stnId":"108",
                        "stnNm":"서울",
                        "tm":"2022-01-01",
                        "avgTa":"-4.3",
                        "minTa":"-10.2",
                        "minTaHrmt":"0710",
                        "maxTa":"2.3",
                        "maxTaHrmt":"1544",
                        "mi10MaxRn":"",
                        "mi10MaxRnHrmt":"",
                        "hr1MaxRn":"",
                        "hr1MaxRnHrmt":"",
                        "sumRnDur":"",
                        "sumRn":"",
                        "maxInsWs":"4.5",
                        "maxInsWsWd":"70",
                        "maxInsWsHrmt":"0923",
                        "maxWs":"2.8",
                        "maxWsWd":"20",
                        "maxWsHrmt":"0819",
                        "avgWs":"1.5",
                        "hr24SumRws":"1335",
                        "maxWd":"50",
                        "avgTd":"-14.4",
                        "minRhm":"31",
                        "minRhmHrmt":"1329",
                        "avgRhm":"46.3",
                        "avgPv":"2.1",
                        "avgPa":"1019.8",
                        "maxPs":"1034.0",
                        "maxPsHrmt":"0247",
                        "minPs":"1027.3",
                        "minPsHrmt":"2351",
                        "avgPs":"1030.9",
                        "ssDur":"9.6",
                        "sumSsHr":"9.0",
                        "hr1MaxIcsrHrmt":"1200",
                        "hr1MaxIcsr":"1.82",
                        "sumGsr":"10.39",
                        "ddMefs":"",
                        "ddMefsHrmt":"",
                        "ddMes":"",
                        "ddMesHrmt":"",
                        "sumDpthFhsc":"",
                        "avgTca":"1.4",
                        "avgLmac":"1.4",
                        "avgTs":"-3.7",
                        "minTg":"-15.4",
                        "avgCm5Te":"-0.9",
                        "avgCm10Te":"-1.0",
                        "avgCm20Te":"-0.3",
                        "avgCm30Te":"0.9",
                        "avgM05Te":"2.7",
                        "avgM10Te":"6.6",
                        "avgM15Te":"10.1",
                        "avgM30Te":"15.1",
                        "avgM50Te":"17.2",
                        "sumLrgEv":"1.3",
                        "sumSmlEv":"1.8",
                        "n99Rn":"0.3",
                        "iscs":"",
                        "sumFogDur":""
                    }
                    ]
                },
                "pageNo":1,
                "numOfRows":2,
                "totalCount":1
            }
        }
    }
    """
```
---
API 서버에 GET 요청 보내기 + JSON 파싱
	`requests.get()`으로 서버에 요청을 보내면 `response` 객체가 돌아옵니다.  
	`response.json()`은 응답 본문(JSON 문자열)을 파이썬 객체로 변환합니다.

```python
response = requests.get(url, params=params, timeout=20)
response.raise_for_status()  # HTTP 상태코드가 200이 아니면 예외 발생

data = response.json()
```
- `raise_for_status()`는 401/404/500 같은 HTTP 오류를 조기에 잡는 용도입니다.
- HTTP 200이어도 API 내부에서 오류가 날 수 있으므로 `resultCode`도 확인합니다.
---
샘플 응답 데이터 구조 이해
	응답은 크게 다음 구조로 내려옵니다.
- `response.header` : 결과코드/메시지
- `response.body.items.item` : 실제 데이터 배열(list)
    
필요한 데이터만 수집:
```python
    if data["response"]["header"]["resultCode"] == "00":
        item_list = data["response"]["body"]["items"]["item"]
        dict_list = []
        for item in item_list:
            item_dict = {}
            item_dict["date"] = item["tm"] # 시간
            item_dict["min_temp"] = item["minTa"] # 최저 기온
            item_dict["max_temp"] = item["maxTa"] # 최고 기온
            item_dict["cloud_amount"] = item["avgTca"] # 평균 전운량
            item_dict["max_snow"] = item["ddMes"] # 일 최심적설
            item_dict["rain"] = item["sumRn"] # 일강수량
            dict_list.append(item_dict)
        return dict_list
    else:
        return None
```
---
API 내부 결과코드(resultCode) 확인
	공공데이터 API는 HTTP 200이라도 내부적으로 실패할 수 있으므로  
	반드시 `resultCode == "00"` 인지 확인합니다.
```python
if data["response"]["header"]["resultCode"] != "00":
    print("❌ API 오류:", data["response"]["header"])
    return None
```

필요한 데이터만 추출해서 리스트로 가공
	`item_list`에는 날짜별 관측값이 여러 건 들어 있고, 각 item은 dict입니다.  
	그중 우리가 필요한 필드만 뽑아서 `dict_list`로 재구성합니다.
```python
item_list = data["response"]["body"]["items"]["item"]

dict_list = []
for item in item_list:
    item_dict = {
        "date": item.get("tm"),             # 날짜
        "min_temp": item.get("minTa"),      # 최저 기온
        "max_temp": item.get("maxTa"),      # 최고 기온
        "cloud_amount": item.get("avgTca"), # 평균 전운량
        "max_snow": item.get("ddMes"),      # 일 최심적설
        "rain": item.get("sumRn"),          # 일강수량
    }
    dict_list.append(item_dict)

return dict_list
```
	item["필드"] 대신 item.get("필드")를 쓰면, 값이 비어 있거나 키가 없을 때도 에러 
	없이(None) 처리할 수 있어 안정적입니다.

---
실행 진입점(main)에서 함수 호출
	`.py` 파일에서 실행하려면 함수를 정의만 하지 말고 호출해야 합니다.  
	`if __name__ == "__main__":` 블록은 이 파일을 직접 실행할 때만 동작합니다.

---
실행:
```python
python weather_asos.py
```