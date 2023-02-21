# Lotto 6/45 1회부터 최신회차 까지 크롤링 

## Lotto Crawling.

* 사용 기능 추가

```python
import pandas as pd 
import requests
from bs4 import BeautifulSoup

from datetime import datetime
import re

import warnings
warnings.filterwarnings('ignore')
```


* url 설정하기
    - url 설정 부분에서 Teddy Note의 게시물을 참고하여 특정 회차 url 확인 

```python 
# count에 원하는 회차 입력 
count = 1055
url = f"https://dhlottery.co.kr/gameResult.do?method=byWin&drwNo={count}"
```


* response 및 text 부분 저장
    - url 설정 부분에서 Teddy Note의 게시물을 참고하여 특정 회차 url 확인 

```python 
# 해당 부분에서 Response [200] 확인할 것
response = requests.get(url)
response.text
```

* BeautifulSoup 의 html.pareser 사용

```python 
dom = BeautifulSoup( response.text, "html.parser")
```

* dom의 내용을 대략적으로 보면 아래와 같다.
```
<!DOCTYPE html>

<html lang="ko">
<head>
<meta charset="utf-8"/>
<meta content="동행복권" id="utitle" name="title"/>
<meta content="동행복권 1055회 당첨번호 4,7,12,14,22,33+31. 1등 총 11명, 1인당 당첨금액 2,362,815,205원." id="desc" name="description"/>
<title>로또6/45 - 회차별 당첨번호</title>
<title>동행복권</title>
<meta content="IE=edge" http-equiv="X-UA-Compatible"/>
<link href="/images/common/favicon.ico" rel="shortcut icon" type="image/x-icon"/>
<link href="/images/common/favicon.ico" rel="icon" type="image/x-icon"/>
<script src="/js/jquery-1.9.1.min.js" type="text/javascript"></script>
<script src="/js/jquery-ui.js" type="text/javascript"></script>
<script charset="utf-8" src="/js/common.js" type="text/javascript"></script>
<script type="text/javascript">
```

* 여기서 날짜 정보를 우선적으로 가지고 올건데 크롬의 개발자 도구를 사용하여 날짜 위치의 html 정보를 확인한뒤 css selector 를 사용하면 아래의 코드로 날짜의 값을 불러올 수 있다.
```python
dom.select(".desc")[0].text

# 실행결과 >>> '(2023년 02월 18일 추첨)'
```

* 실행된 결과에서 날짜 값만 추출한다. 
```python
date = datetime.strptime(dom.select(".desc")[0].text, "(%Y년 %m월 %d일 추첨)")

# 실행결과 >>> datetime.datetime(2023, 2, 18, 0, 0)
```


* 이번에는 해당 날짜의 당첨 번호 6자리를 추출한다.
    * 해당 값도 개발자도구를 통해 css selector 를 이용하여 불러온다. 
```python
number = dom.select("#article > div:nth-child(2) > div > div.win_result > div > div.num.win")[0].text

# 실행결과 >>> '\n당첨번호\n\n4\n7\n12\n14\n22\n33\n\n'
```

* 위의 코드가  실행되면 불필요한 정보들이 섞여있는데 숫자 값만 추출한다.
```python 
win_number = list(map(int, (number.split("\n"))[3:-2]))

# 실행결과 >>> [4, 7, 12, 14, 22, 33]
```

* 보너스 번호를 추출한다.
    * 방법은 위의 6자리 숫자를 추출하는 방법과 유사하나 css selector의 값이 조금 다르다.
```python 
bonus = dom.select("#article > div:nth-child(2) > div > div.win_result > div > div.num.bonus > p > span")[0].text

bonus = int(bonus)
```

* 이후 회차별 총 당첨 금액 및 1인당 수령 금액을 얻어왔으나 수정간에 코드가 꼬여 해당 부분은 추후에 추가해야 할 것 같다.

* 가장 위의 count 에 할당한 한개의 회차의 값을 데이터 프레임으로 생성한다.
```python 
lotto_df = pd.DataFrame({
    "날짜" : [date],
    "회차" : count,
    "num1" : win_number[0],
    "num2" : win_number[1],
    "num3" : win_number[2],
    "num4" : win_number[3],
    "num5" : win_number[4],
    "num6" : win_number[5],
    "bonus" : bonus,
    # "total_1st" : ''.join( re.findall("[0-9]+", total_price[0]) ),
    # "personal_1st" : ''.join( re.findall("[0-9]+", person_price[0]) )

})
```


* 위에서 작성한 기능을 함수로 작성하여 원하는 회차들의 값을 얻어오도록한다.
```python 
# 특정 회차 지정하여 크롤링
def crawling_lotto(count) :
    url = f"https://dhlottery.co.kr/gameResult.do?method=byWin&drwNo={count}"

    response = requests.get(url)
    dom = BeautifulSoup( response.text, "html.parser")

    # date 생성
    date = datetime.strptime(dom.select(".desc")[0].text, "(%Y년 %m월 %d일 추첨)")

    # 당첨번호 생성
    number = dom.select("#article > div:nth-child(2) > div > div.win_result > div > div.num.win")[0].text
    win_number = list(map(int, (number.split("\n"))[3:-2]))

    # 보너스 번호 생성
    bonus = dom.select("#article > div:nth-child(2) > div > div.win_result > div > div.num.bonus > p > span")[0].text
    bonus = int(bonus)

    # # 1등 총 당첨 금액 
    # total_price = [table_data.select(f"#article > div:nth-child(2) > div > table > tbody > tr:nth-child({i}) > td:nth-child(2) > strong")[0].text for i in range(1,6) ]

    # # 개인 1등 당첨 금액 
    # person_price = [table_data.select(f"#article > div:nth-child(2) > div > table > tbody > tr:nth-child({i}) > td:nth-child(4)")[0].text for i in range(1,6) ]

    return  {
                "날짜" : date,
                "회차" : count,
                "num1" : win_number[0],
                "num2" : win_number[1],
                "num3" : win_number[2],
                "num4" : win_number[3],
                "num5" : win_number[4],
                "num6" : win_number[5],
                "bonus" : bonus,
                # "total_1st" : ''.join( re.findall("[0-9]+", total_price[0]) ),
                # "personal_1st" : ''.join( re.findall("[0-9]+", person_price[0]) )
                }


def df_lotto(start, end) :  
    lotto_df = pd.DataFrame()     

    for i in range(start, end+1) :
        result = crawling_lotto(i)

        lotto_df = lotto_df.append({
                    "날짜" : result[ "날짜" ],
                    "회차" : result[ "회차" ],

                    "num1" : result[ "num1" ],
                    "num2" : result[ "num2" ],
                    "num3" : result[ "num3" ],
                    "num4" : result[ "num4" ],
                    "num5" : result[ "num5" ],
                    "num6" : result[ "num6" ],

                    "bonus" : result[ "bonus" ],
                    # "total_1st" : result[ "total_1st" ],
                    # "personal_1st" : result[ "personal_1st" ]
        }, ignore_index = True)

    return lotto_df
```


## 전회차 데이터 수집

* 1회부터 1055회까지 한번에 크롤링을 시도하였으나 중간에 연결이 끊기는 문제가 생겨 200회씩 나누어서 수집하였다.
```python 
lotto_1= df_lotto(1,200)
lotto_200 = df_lotto(201,400)
lotto_400 = df_lotto(401,600)
lotto_600 = df_lotto(601,800)
lotto_800 = df_lotto(801,1000)
lotto_1000 = df_lotto(1001,1055)
```

* 이후 수집한 데이터를 concat을 사용하여 아래로 이어붙인다.
```python 
lotto_1055 = pd.concat([lotto_1, lotto_200], axis=0)
lotto_1055 = pd.concat([lotto_1055, lotto_400], axis=0)
lotto_1055 = pd.concat([lotto_1055, lotto_600], axis=0)
lotto_1055 = pd.concat([lotto_1055, lotto_800], axis=0)
lotto_1055 = pd.concat([lotto_1055, lotto_1000], axis=0)

lotto_1055.reset_index(inplace=True,drop=True)

# 데이터 생성에 적지않은 시간이 걸리므로 csv 파일로 생성
lotto_1055.to_csv("lotto_1055.csv", encoding="utf-8-sig", index=False)
```

* 번호가 num1 ~ num6 으로 나누어져 있으므로 각 열에 대해서 value_counts를 사용
```python
num1 = pd.DataFrame( lotto_1055["num1"].value_counts() )
num2 = pd.DataFrame( lotto_1055["num2"].value_counts() )
num3 = pd.DataFrame( lotto_1055["num3"].value_counts() )
num4 = pd.DataFrame( lotto_1055["num4"].value_counts() )
num5 = pd.DataFrame( lotto_1055["num5"].value_counts() )
num6 = pd.DataFrame( lotto_1055["num6"].value_counts() )
```

* 이후 인덱스가 Lotto의 각 번호이므로 index를 기준으로 merge를 진행한다
```python 
total_num = pd.merge(num1, num2, left_index =True, right_index = True, how="outer")
total_num = pd.merge(total_num, num3, left_index =True, right_index = True, how="outer")
total_num = pd.merge(total_num, num4, left_index =True, right_index = True, how="outer")
total_num = pd.merge(total_num, num5, left_index =True, right_index = True, how="outer")
total_num = pd.merge(total_num, num6, left_index =True, right_index = True, how="outer")

# 없는 인덱스에 경우 NaN으로 값이 설정되므로 해당 부분을 0으로 채워준다 .
total_num.fillna(0, inplace=True)
```


* 이후 각 숫자별 합산 횟수를 구하고 새로운 컬럼으로 생성하여 정리한다.
```python
total_num["sum"] = total_num.sum(axis=1)
total_num = total_num[["sum"]]
total_num = total_num.reset_index()
total_num.columns=["number", "count"]
```


## 번호별 1등 당첨 횟수 시각화
```python
import matplotlib.pyplot as plt 
import seaborn as sns

plt.rc('font', family='Malgun Gothic') # For Windows
print(plt.rcParams['font.family'])

%matplotlib inline
```

```python
plt.figure(figsize=(15,4))
plt.title("번호별 1등 당첨 횟수")
sns.barplot(data=total_num, x="number", y="count")

plt.ylim(110,165)
plt.show()
```

* 실행결과
<img src="~@source/output.png" />
