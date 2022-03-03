from bs4 import BeautifulSoup
import bs4
from urllib.request import Request, urlopen
%matplotlib inline
from IPython.core.interactiveshell import InteractiveShell
InteractiveShell.ast_node_interactivity = "all"
import pandas as pd
pd.set_option('display.float_format', lambda x: '%.3f' % x)    # DataFrame 숫자데이터가 소수점 셋째자리까지 나오도록 설정
pd.set_option('max_columns', None)  # DataFrame에서 모든 컬럼을 다 볼 수 있게 설정
# pd.set_option('max_rows', None)  # DataFrame에서 모든 행을 다 볼 수 있게 설정
import numpy as np
import requests
from datetime import datetime as dt
import re

req = Request("https://www.marketcaphistory.com/spx-500/")
html_page = urlopen(req)

soup = BeautifulSoup(html_page, "lxml")

links = [] # 빈 리스트 만들기
for link in soup.findAll('a'): # soup에서 a태그 모두 찾고 그 안에서 루프
    links.append(link.get('href')) # href 링크걸린에를 links에 추가

# 데이터 정리하기
df = pd.DataFrame(links) # 리스트로 추출된 데이터를 DataFrame으로 변수에 저장, 다루기 쉽게
df = df.drop([0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,516,517,518,519,520,521,522,523])
df = df.reset_index() # 인덱스 재설정
df = df.drop("index", axis=1) # 불필요한 칼럼 삭제
# 중복되는 주소 지우기, 'regex=True'는 부분일치
df.replace('https://www.marketcaphistory.com/', '', regex=True, inplace=True) 

# for 루프 돌리기 위해서 데이터프레임을 다시 리스트로 변환 
np.array(df[0].tolist()) 
symbol_list = list(np.array(df[0].tolist()))

# for 루프로 심볼리스트의 모든 회사의 데이터를 total_df에 담는다.
url = "https://www.marketcaphistory.com/"
total_df = pd.DataFrame() # 데이터가 담길 df 생성
for symbol in symbol_list: # 주소값 끝부분 회사명만 있는 리스트
    result = urlopen(url + str(symbol)) # url 조합 자동완성 시키기
    html = result.read() 
    soup = BeautifulSoup(html, 'html.parser') 
    temp = soup.select("td.infotablebox") # 특정 클래스 선택
    table = temp[0] # 클래스로 특정할 수 없어서 첫번째 td가 내가 찾는 데이터임을 확인하고 지정해서 table에 변수 지정
    t_html = str(table) # 테이블 html 정보를 문자열로 변경하기
    t_list = pd.read_html(t_html) # 판다스의 read_html로 테이블 정보 읽기
    
    t_df = t_list[0] # 데이터프레임 선택하기
    df = t_df.drop(t_df.index[0]) # 첫번째 행을 삭제
    
    df["symbol"]=symbol # 컬럼에 해당 symbol을 추가
    total_df = pd.concat([total_df,df])  # df를 total_df와 이어 붙이기
total_df.columns = ['date', 'marketcap', 'company'] # 컬럼 이름 바꾸기
total_df["company"] = total_df["company"].str.replace("/", "") #불필요한 문자 제거

# 문자열을 datetime 형식으로 변환 (Y 대문자는 년도의 4자리를 의미)
total_df['date'] = pd.to_datetime(total_df['date'], format="%m/%d/%Y")

# marketcap 칼럼을 M,B,T이 포함된 문자형에서 숫자로 변환하는 코드
total_df.marketcap = (total_df.marketcap.replace(r'[MBT]+$', '', regex=True).astype(float) * \
                      total_df.marketcap.str.extract(r'[\d\.]+([MBT]+)', expand=False)
                              .fillna(1)
                              .replace(['M','B','T'], [10**6, 10**9, 10**12])
                              .marketcap.astype(int)) # 정수형으로 변환

df['date'] = df['date'].dt.strftime('%Y-%m') # 년도와 월만 남기고 day는 삭제
df['rank'] = "" # 빈 컬럼 추가하기

# date 기준으로 그룹바이해서 marketcap 값으로 랭킹을 부여한 값을 rank 컬럼에 입력 
# ('method=average'가 기본값이라서 소수점으로 나오기 때문에 min으로 지정하면 정수로 나타낸다.)
# (min으로 하면 공동1등이 있으면 2등은 없고 3등부터 표시)
df['rank'] = df.groupby('date')['marketcap'].rank(method='min', ascending=False)
df['rank'] = df['rank'].astype(int) # 순위를 정수형으로 변환







