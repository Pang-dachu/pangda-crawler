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
