# [AWS] EC2 인스턴스 성능 리포트

<h3> ✔️ 주 제 : 네이버 금융 코스피 지수 크롤링 속도 비교</h3>
<h3> ✔️ 아키텍처 : 64bit(ARM) </h3>
<h3> ✔️ OS: Ubuntu  24.04 LTS </h3>

<br>

## 1. TEAM name : War Of Money 💸

<li>박준승</li>
<li>신정인</li>
<li>양일교</li>
<br>

## 2. 비교 지표

<img width="399" alt="Untitled" src="https://github.com/koorukuroo/pda_4th/assets/93638922/90c797df-83e0-4f65-b073-fcedda9139e9">
<h4> - 동일 Instance Family 내 Instance size 별 성능 분석 </h4>

<h3>✔️인스턴스 유형 List :</h3>
<li>t4g.medium </li>
<li>t4g.xlarge</li>
<li>c6g.medium</li>
<li>c6g.xlarge</li>
<li>m6g.medium</li>
<li>m6g.xlarge</li>

<br>

# 3. Test Code 💻

```python
import aiohttp
import asyncio
import bs4
import datetime as dt
import time
historical_prices = []
def date_format(d):
    d = str(d).replace('-', '.')
    yyyy = int(d.split('.')[0])
    mm = int(d.split('.')[1])
    dd = int(d.split('.')[2])
    this_date = dt.date(yyyy, mm, dd)
    return this_date
async def fetch_page(session, url):
    async with session.get(url) as response:
        return await response.text()
async def parse_page(index_cd, page_n):
    url = f'https://finance.naver.com/sise/sise_index_day.nhn?code={index_cd}&page={page_n}'
    async with aiohttp.ClientSession() as session:
        html = await fetch_page(session, url)
        source = bs4.BeautifulSoup(html, 'lxml')
        date = source.find_all('td', 'date')
        price = source.find_all('td', 'number_1')
        page_prices = []
        for n in range(len(date)):
            if date[n].text.split('.')[0].isdigit():
                this_date = date[n].text
                this_date = date_format(this_date)
                this_close = price[n * 4].text
                this_close = float(this_close.replace(',', ''))
                line = [this_date, this_close]
                page_prices.append(line)
        return page_prices
async def historical_index_naver_async(index_cd, start_page, end_page):
    tasks = []
    for page_n in range(start_page, end_page + 1):
        tasks.append(parse_page(index_cd, page_n))
    results = await asyncio.gather(*tasks)
    for result in results:
        historical_prices.extend(result)
# 실행 예시
async def main():
    index_cd = 'KPI200'  # 예: 코스피 200
    start_page = 1
    end_page = 100  # 성능 테스트를 위해 페이지 수를 적절히 조정 가능
    await historical_index_naver_async(index_cd, start_page, end_page)
if __name__ == "__main__":
    # 크롤링을 5번 반복
    sum = 0
    for i in range(5):
        historical_prices.clear()
        start_time = time.time()
        # Python 스크립트에서 asyncio.run() 사용
        asyncio.run(main())
        end_time = time.time()
        sum += end_time - start_time
        print(round(end_time - start_time, 2))
    print(round(sum/5, 2))
```

<br>

## 4. 결과 분석 (5회 평균)

| 인스턴스 유형  | 하드웨어 스펙  | 크롤링 테스트 | 비용 (시간당)|
| ------------- | --------------- | ---------- | -------- |
| t4g.medium    | 2v CPU, 4G RAM  | 1.31 sec   |  0.0385 USD  |
| t4g.xlarge    | 4v CPU, 16G RAM | 1.26 sec   |   0.154 USD  |
| c6g.medium    | 1v CPU, 2G RAM  | 1.21 sec   |  0.047 USD   |
| c6g.xlarge    | 4v CPU, 8G RAM  | 1.16 sec   |  0.188 USD   |
| m6g.medium    | 1v CPU, 4G RAM  | 1.19 sec   |  0.0416 USD   |
| m6g.xlarge    | 4v CPU, 16G RAM | 1.17 sec   |  0.1664 USD   |

<br>

## 6. 

## 5. 특이 사항

 <br>

## 6. 정리

<br>



## 7. 앞으로 더 공부해볼 내용

</br>
