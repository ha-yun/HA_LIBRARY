# 문자열 포맷팅

1. 문자열 %
    ```
        # '1 + 2 = 3'    
        '%d + %d = %d' % (1, 2, 3)   

        # 1 + 1.100000 = 2.100000
        '%d + %f = %f' % (1, 1.1, 1+1.1)    

        # 1 + 1.1 = 2.1
        '%s + %s = %s' % (1, 1.1, 1+ 1.1)
    ```

2. 문자열.format()
    ```
        '{} + {} = {}'.format( 1, 2, 1+2)

        # 2 + 1 = 3 => 인자값대로
        '{1} + {0} + {2}'.format( 1, 2, 1+2)    
    ```

3. fstring   
    ```
        f'{1} + {2} = {1+2}'

        # > :오른쪽 정렬, < : 왼쪽정렬, ^:중앙정렬\n# {a:자리수}
        # {a:>자리수} : 자리수 + 오른쪽 정렬
        # {b:*<자리수} : 자리수 + 왼쪽 정렬, 빈공간을 *로 채운다

        f'{a:>6}-{b:*<7}-{a+b:=^7}'

        # 이런것도 가능.. => '     1-2******-===3===' 
    ```


---

## 기타 함수
- ord()  :  아스키코드를 계산하는 함수
```
    #(97, 98)
    ord('a'), ord('b')
```

```
    # a카운트 정보를 가지고 있는 방번호, 인덱스 -> 0
    print( ord('a') - ord('a') )

    # b의 카운트 정보를 가지고 있는 방번호, 인덱스 -> 1
    print( ord('b') - ord('a') )

    # z의 카운트 정보를 가지고 있는 방번호, 인덱스 -> 25
    print( ord('z') - ord('a') )
```

- zip()  :  동일한 맴버를 가진 2개의 연속데이터를 기반으로 같은 위치에 존재하는 데이터간 쌍으로 구성
```
    a = [1, 2, 3]
    b = ['a','b','c']

    for temp in zip(a,b):
    print( temp )

    '''
        (1, 'a')
        (2, 'b')
        (3, 'c')
    '''
```

## 애너테이션
- 3.6부터 추가됨
- 변수, 인자, 반환값등등 타입 설명 o, 강제 x
- 파이썬 프로젝트나 코드를 깃헙에 반영하여 포플 처리 -> 고급지게 표현
- 애너테이션을 미적용 하면 Any 타입으로 설명됨

```
    score : int = 100 # 가급적 정수값을 대입, 사용해라(권장, 강제 x)
    text : str = "hello"

    # 함수 애너테이션
    def test( x:int=1, name : str = 'math') -> str:
    return f'{x} {name}'
```


-----
## Class 상속
- OOP의 특징중 하나
  - 부모 클레스를 상속받아서 부모의 유산을 자식이 그대로 사용, 업그레이드(재정의) 사용 가능
  - 파이썬의 모든 클레스의 수퍼 클레스는 Object
  - 자식 클레스 제로 베이스에서 생성되는것 아닌, 유산을 받아서 생성-> 빠른 개발 가능!!

- 상속
  - 부모 클레스 has a 자식클레스

- 추상화
  - 자식 클래스 is a 부모클래스

```
    # 상속 : 클래스명(부모클래스)
    class 제네시스1세대: pass
    class 제네시스2세대(제네시스1세대): pass


    # 상속관계 확인
    제네시스2세대.mro()

    # 다중 상속
    class HighEnd:pass
    class 제네시스3세대(제네시스2세대, HighEnd): pass
```

### class상 특별한 함수
- 데코레이터 사용 (@xxxx)
```
    class Test():
    # 클래스 변수
    m = 0

    # 클래스 함수
    @classmethod
    def class_func( arg ):
        '''
        arg는 클래스 자체가 전달
        '''
        print(arg.m)

    # 정적 함수
    @staticmethod
    def static_func():
        '''
        클래스 내에 어떤 변수도 사용 x
        단독 사용, 순수(퓨어) 함수 스타일 -> 유틸리티 용도 유용함
        '''
        a = 10
        print( '정적함수', a)
```

## 접근 제어
- 종류 2레벨만 존재
  - public(공개), private(비공개)
  - 키워드 x
  - 이름으로 구분
    - __XXX : 비공개, 외부 노출 x, 외부 접근 x
    - _XXX  : 개발자가 굳이 알필요 없다라는 뉘앙스의 표식
```
    class Access_Test():
    def __init__(self, _a):
        self.__a = "hello" # 외부 노출 x, 접근 x, 내부에서만 사용
        self._a = _a # 굳이 알필요는 없다

    # 프리이빗한 함수
    def __pri_func(self):
        print('숨겨진 함수', self.__a, self._a)

    def _pri_func(self):
        print('내부에서만 사용되는 함수, 알필요 없다', self.__a, self._a)

    # 객체를 설명하는 기능 추가 -> 재정의
    def __str__(self):
        # print()를 통해서 객체 출력될대
        return "<객체에 대한 설명 기술1>"

    def __repr__(self):
        # 객체를 값으로 출력할때
        return "<객체에 대한 설명 기술2>"
```

```
    a = Access_Test('hi')
    a._Access_Test__pri_func()
    # 숨겨진 함수 hello hi
```

----
# 데이터 수집 BeautifulSoup (bs4)
1.타겟 사이트 접속 -> 요청 -> 응답 (html)
  - 요청(GET)
    - urllib.request.urlopen() 활용
  - 응답
    - 웹 사이트를 긁어낸다
    - 응답 데이터 -> 텍스트 -> html 형태(반정형 데이터)
      - 참고
        - 정형 데이터 : RDS 데이터
        - 비정형 : 이미지, 사운드 -> 바이너리 데이터
        - 반정형 : xml, json, html

2. 데이터 추출
  - html -> parsing -> DOM tree 구성 -> 탐색 -> 데이터 추출 -> 파이썬의 기본 자료구조를 이용하여 데이터 구성

  - 파서 : html5lib 사용 (bs4에 같이 혼용)
    - html -> DOM tree로 올릴때 사용하는 도구
  
  - 탐색 : 파이썬 문법 활용
    - css selector|xpath 사용
    - bs4 사용

3. 데이터 적제
  - pandas 라이브러리 사용 (필요한 만큼 사용)
  - 파일
    - csv/xlsx 등 형식 or database 적제(aws rds 활용)
    - 로컬에서 자동화 -> 로컬 db에 적제도 가능
  - 디비
    - 특정 테이블에 바로 밀어 넣기 수행
  - ...

```
    # 1. 타겟 사이트 접속 및 요청/응답
    import urllib.request as req
    TARGET_SITE = 'https://naver'
    res = req.urlopen(TARGET_SITE) # get 방식

    # 2. 데이터 추출 -> 파싱(파서 : html5lib)
    import bs4
    from bs4 import BeautifulSoup
    ### 파싱 = html을 읽어서 분석 -> 객체 생성 => 메모리 로드
    soup = BeautifulSoup( res, 'html5lib')

    len( soup.select('.class명') )

    # 3. 데이터 적제
    ## db에 적제하기 위한 타입 변환 => [{...}, ...] -> pandas의 DataFrame으로 변환
    df = pd.DataFrame.from_dict(results)
```

#### aws상 데이터베이스에 적제
- 파이썬 데이터베이스 프로그래밍
- sqlalchemy (파이썬에서 sql 지원관련 대표적 라이브러리)
- pymysql (파이썬 <-> mysql 연동용 라이브러리 중 하나)
- 절차
- 파이썬 <-> pymysql <-> sqlalchemy <-> mysql
```
    from sqlalchemy import create_engine

    # db 접속 정보 필요!
    IP = 'database-python.c1guqm8eion2.ap-southeast-1.rds.amazonaws.com'
    ID = 'admin'
    PW = '0123'
    PORT = 3306
    DBNAME = 'mydb'
    TABLE_NAME = 'tbl_exchange' # 테이블은 파이썬에서 데이터 밀어 넣을 때 없으면 자동으로 생성 시킴
    PROTOCAL = 'mysql+pymysql' # 디비 제품별로 상이하게 구성

    # 접속 URL 구성
    db_url = f'{PROTOCAL}://{ID}:{PW}@{IP}:{PORT}/{DBNAME}'
        
    # 1. DB 접속 관련 엔진 생성
    engine = create_engine(db_url)

    # 2. 접속시도 -> I/O
    conn = engine.connect()

    # 3. 데이터 밀어넣기
    df.to_sql(name=TABLE_NAME, # 대상 테이블
            con=conn,          # 접속정보
            if_exists='append', # 기존에 데이터가 있어도 누적하여 추가
            index=False         # 인덱스 정보 제거
            )

    # 4. 접속 종료
    conn.close()

```

# 웹크롤링(selenium)
- 브라우저 필요(Web Driver) 필요
  - 밴더별로 대부분 지원
  - Chrome 사용
    - 115 버전 이후부터는 제공 사이트 변경되었음
    - https://googlechromelabs.github.io/chrome-for-testing/
      - 반드시 본인 PC의 크롬의 버전과 일치해야함 -> 작동됨 -> 필요시 업데이트 필요
      - 본인 PC와 일치하는 버전 다운로드 -> 압축해제
        - 맥/리눅스 : chromedriver
        - 윈도우 : chromedriver.exe

```
    # 1. 모듈 가져오기
    from selenium import webdriver as wd

    # 2. 브라우저 가동
    driver = wd.Chrome() # 가장 기본형 (노 옵션,)

    # 3. 타겟사이트 접속, GET 방식
    TARGET_SITE = "https://www"
    # 접속
    driver.get(TARGET_SITE)

    # 4. 정보 획득
    from selenium.webdriver.common.by import By     # 대상을 특정하는 수단(표현)

    # 찾기
    sido_otps = driver.find_elements(By.CSS_SELECTOR, "#SIDO_NM0 > option")
    len(sido_otps)
```

# numpy
> The fundamental package for scientific computing with Python
- 수학(선형대수학), 과학용(푸리에) 라이브러리
- 특징
    - 빠른 연산(고속), 효율적 메모리, 벡터 연산에 탁월
    - 향후 배우는 모든 패키지의 베이스임!!
- 자료구조
    - 배열
    - ndarray(n-차원 배열)
    - **배열의 모든 구성원은 동일 타입!!**

### 배열 생성
- 생성
  - 입력데이터로부터 생성
    - 입력원 : [], (), ndarray, Series, DataFrame, ..
    - 출력 : ndarray
- 타입적 특징
  - 단일 타입만 ndarray 생성됨
  - **수치형(정수, 부동소수) <= 가장 많이 보임**

```
    arr = np.array(datas)

    # 기본 배열 카피(얕은 카피)
    arr3 = np.asarray(arr2)

    # 파이썬의 range()와 유사기능
    np.arange(5)

    # arange()로 연속수 생성 -> reshape()를 통해서 원하는 구조를 생성
    # 1차원 (1, ) => 2차원(4, 4)
    arr5 = np.arange(1,17).reshape(4,4)
    # -1 : 해당 차원 지정 x, 알아서 계산하여 적용
    np.arange(1, 17).reshape(4,-1)

    # 타입 변경 -> astype() , 정보 손실이 있더라고 타입변경
    arr.astype(np.int64)

    # 축변경
    # (8,4) => (4,8)
    arr.T, arr.T.shape
```
- 특수 배열 생성
```
    # zeros, ones, empty


    # zeros -> 데이터를 담을 그릇 용도로 많이 사용
    np.zeros( (2,3) )
    '''
    array([[0., 0., 0.],
       [0., 0., 0.]])
    '''

    np.ones( (2,3) )
    '''
    array([[1., 1., 1.],
       [1., 1., 1.]])
    '''

    # 임의값
    np.empty( (2,3) )
    '''
    array([[1., 1., 1.],
       [1., 1., 1.]])
    '''

    # 대각선 1, 좌우 대각선 대칭
    np.eye(3,3)
    '''
    array([[1., 0., 0.],
       [0., 1., 0.],
       [0., 0., 1.]])
    '''
```

- 난수 배열
  - 기본, 표준편차1, 평균0인 데이터 분포상(정규분포)에서 난수 추출
```
    np.random.randn(2,3,4)
    '''
    array([[[-1.09707857,  1.32637405, -0.01907737, -0.7876788 ],
        [-0.86623523,  0.166322  ,  0.78015965, -0.94574245],
        [-0.963701  ,  0.47945798, -0.25438056, -0.43249188]],

       [[-0.02864067,  1.3297121 ,  0.52994237, -1.17167514],
        [-2.27594567,  1.1740634 ,  0.75434796, -1.02141863],
        [-1.92295893,  0.20717685,  1.11327035, -0.47132797]]])
    '''
```

### 불리언 인덱싱
- 조건식의 결과가 True인 데이터만 추출(인덱싱)하겠다
    - **필터링 처리를 데이터에 적용 -> 추출**
```
    arr[ arr > 0]   # 양수만 추출(인덱싱 -> 차원축소 -> 1차원)
    # 조건이 여러개라면?
    # [ (조건) | (조건)]
    # [ (조건) & (조건)]
    # 멤버들 중 절대값이 1보다 큰 데이터만 추출하시오 (예시, 절대값을 바로 적용해도 됨)
    arr[ (arr > 1) | (arr < -1) ]

    # 특정 조건에 해당되는 데이터만 일괄 초기화 => 결측치 처리 기본
    # 여기서는 음수를 결측치로 간주하겠다
    arr[ arr < 0 ] = 0
```

### 펜시 인덱싱
- 비연속적인 데이터를 선택적으로 추출
- pandas의 iloc으로 발전
```
    # 펜시 인덱싱 적용
    # 1차원에서 3, 0, 5, 2번 데이터만 가져온다
    # 표현자체가 [] 구성 => 차원유지가 됨
    arr[ [3, 0, 5, 2]]
```

----------


# pandas
> pandas is a fast, powerful, flexible and easy to use open source data analysis and manipulation tool,
built on top of the Python programming language.
- 데이터 분석용
- R <=> pandas(분석) + sklearn(머신러닝)
- 자료구조
  1. Series (1차원) : 데이터 + 인덱스
    - 차원구조상 벡터와 일치
  2. DataFrame (2차원) : 데이터 + 인덱스 + 컬럼
    - 차원구조상 매트릭스 일치

- 기본 플로우
  - 원본 raw 데이터(디비, 파일(엑셀,csv), 네트워크, json, ...) -> DataFrame 로드
  - 이후단계
    - 요구 사항별로 상이함
      - 데이터 전처리/추출/병합
      - 집계, 피벗등 다양한 작업 진행
  - 시각화, EDA
    - matplotlib, seaborn 등 다양한 라이브러리 활용 진행

```
    import pandas as pd
    # Series 구성
    vec = pd.Series( [-1,3,5,np.nan,8,7] )
    vec.name = '시리즈 더미데이터'
    # 컬럼 자리에 시리즈의 이름이 세팅

    # 시리즈에서 데이터만 추출 -> 배열로 데이터 추출
    vec.values
    # 인덱스 추출 => 제너레이터 형식으로 구성
    vec.index
```

```
    # pd.DataFrame(값, 인덱스, 열)
    df = pd.DataFrame( raw_data, indexs, cols)

    # 데이터 차원 정보
    df.shape

    # 특징 -> df는 컬럼별로 서로 상이한 타입을 가질수 있다 -> 배열하고 다른점 -> dtypes로 표기함
    # 머신러닝/딥러닝 -> 데이터를 수치화 해야 한다!! -> 학습이 된다 -> 모델을 구축
    df.dtypes

    df.columns
    df.values
    df.index

    # 결측검사
    df.info()

    # 기초 통계량 -> 타입이 수치인 컬럼만 출력
    df.describe()

    # 데이터 정렬(ex-나이기준)
    df.sort_values(by='age', ascending=False)
```

### 인덱싱(loc, iloc)
1. loc
- 기본적인 인덱싱 방법
  - df.loc[ ]
- 추출에 기본 기준값 -> **loc**ation(위치 정보)
  - 위치 정보( 인덱스, 컬럼 )
  - 2차원 좌표계를 사용하는 것과 유사 -> 데이터를 특정하는 정보

```
    # : => 슬라이싱 => 차원 유지
    # 시작값 <= 데이터 <= 끝값(끝값이 포함된다. 특이점~!)
    df.loc['2010-01-01':'2010-01-03']

    # [인덱스, 컬럼] 조합 활용 => 특정 => 차원축소 2회 발생 => 스칼라 추출
    df.loc[ '2010-01-01', 'customer_name']

    # 차원 축소 1회
    temp = df.loc['2010-01-01', ['customer_name']]
    display(temp), type(temp)

    # 차원 유지
    temp = df.loc[['2010-01-01'], ['customer_name']]
    display(temp), type(temp)


    # : 조합
    # 차원유지, 데이터는 범위에 맞게 추출
    df.loc['2010-01-01':'2010-01-03', ['customer_name']]
```

2. iloc
- index location
  - 암묵적인 순서(인덱스)를 이용하여 데이터 추출
  - 0, 1, 2, ...
```
    # 인덱스 항목중 암묵적으로 순번이 0인 데이터를 추출 -> 특정 -> 차원축소 => Series
    df.iloc[ 0 ]

    # 넘파이의 펜시 인덱싱 적용
    # 원하는 데이터와 원하는 컬럼을 내맘대로 배치해서 획득 가능
    df.iloc[[1, 10,4],[3,1,6]]
```

### 불리언 인덱싱
```
    # 데이터중 젠더값이 F인 데이터만 (불리언 인덱싱)
    temp = cus[ cus.gender == 'F']
    temp.shape

    cus[(cus.membership_period >= 40) & (cus.gender == 'F')].shape
```

### 파생변수 생성 및 데이터 삭제
- 파생 변수
  - 기존의 데이터를 이용 새로운 데이터 생성
    - df[ 컬럼 ] = 데이터(Series나 배열레벨)
    - dict 상에 데이터 추가하는것과 유사함

- 삭제
    - del : 완전삭제
    - dropxxx : 해당 데이터를 제외하고 추출하는 기능
      - 옵션 중 ```inplace=True``` 적용하면 원본 제거

```
    # 원본 반영 => inplace=True
    cus.drop(['end_date'], axis=1, inplace=True)

    # del 사용
    del cus['class']

    # 파생 변수 추가
    df['is_mz'] = (df.age <= 40) & (df.age >= 21)

```

-----------
# 데이터의 형태(범주형 vs. 수치형)
### 범주형 데이터 (Categorical Data)
- 특징: 값을 셀 수 있고, 특정한 **카테고리(집합)**로 구분됨
1. 명목형(Nominal)	순서X (단순한 분류)	성별(남/여), 혈액형(A/B/O)
2. 순서형(Ordinal)	순서O (서열 존재)	영화 평점(⭐~⭐⭐⭐⭐⭐), 학점(A/B/C)
##### ➡ 머신러닝에서 어떻게 활용될까?
- ✅ 지도 학습(Supervised Learning) → 분류(Classification)
    - 범주형 데이터는 정확하게 딱 정답을 맞출 수 있는 문제에서 사용됨
        - 예: 이메일이 "스팸/비스팸"인지 구별하는 분류 문제
- ✅ 피벗, 집계의 대상이 됨
    - 범주형 데이터는 그룹별로 나눠서 집계(aggregation)하거나 피벗(pivoting) 가능
        - 예: 성별(남/여)별 평균 구매 금액을 구하는 경우

### 수치형 데이터 (Numerical Data)
- 특징 : 숫자로 표현되며, 연산이 가능함
1. 이산형(Discrete)	정수값, 셀 수 있음(Countable)	제품 판매 개수(1, 2, 3…), 교통사고 발생 건수
2. 연속형(Continuous)	값이 끊이지 않고 연속적, 소수점 포함 가능	키(172.3cm), 몸무게(65.8kg), 집값(3.14억)
##### ➡ 머신러닝에서 어떻게 활용될까?
- ✅ 지도 학습(Supervised Learning) → 회귀(Regression)
    - 수치형 데이터를 예측하는 문제에 사용됨
        - 예: 집값 예측, 날씨 예측, 키/몸무게 예측 - 키가 172cm라고 해서 "남자/여자"로 딱 떨어지는 게 아니라, 연속적인 값을 예측하는 문제이므로 회귀 사용

### 집계(groupby)
- 대상
  - 집계 기준의 데이터(컬럼 기준) 명목형일때 가능, 일부 이산형도 가능함
```
    temp = df.groupby( ['is_mz','gender'] ).count()[['age']]
```

### 피벗
- 데이터 이면에 존재하는 숨겨진 의미를 파악 유용
  - 서열등 확인할 수 있음
  - 데이터 대상은 범주형
  - 집계와 유사성을 가짐
```
    # 피벗 진행 -> 데이터 1개만 세팅해서 인덱스 배치 -> 체크 -> 필요시 컬럼배치, 값 배치 진행
    '''
    - 왼쪽 인덱스 자리 : Name 컬럼값
    - 오른쪽 데이터(or value) 자리 : Quantity, Price 컬럼값
    - 데이터가 집계가 된다면 -> 기본 설정값(평균)으로 처리
    '''
    df_pv = pd.pivot_table( df, index=['Name'], values=['Quantity','Price'] )

    # 결측치 처리 -> 0으로 대체(판매 x)
    df_pv.fillna(0) # 채운다->na(결측치), 0:0으로 채운다


    # 처음부터 피벗할때 결측치는 0으로 세팅
    df_pv = pd.pivot_table( df,
                        index=['Manager', 'Rep', 'Name'], # 인덱스 (3레벨)
                        values=['Quantity','Price'], # 값 (수량, 단가)
                        aggfunc=['mean',len,'sum'], # 집계(평균, 개수, 총합)
                        columns=['Product'], # 컬럼 항목 제품단위로 분할
                        fill_value=0 # 결측치는 0으로 대체
                            )
```

### 병합
1. merge()
    - SQL의 join과 유사함
    - 2개의 df를 하나의 df로 합병
    - 공통된 컬럼 존재함
2. concat()
    - 단순 합치기
    - 축의 방향(합치는 방향) 필요 : axis

```
    # left join, 왼쪽 데이터는 모두 보전, 일치되는 것만 오른쪽 데이터 포함, 나머진 결측
    merged_df = pd.merge(left_df, right_df, on='key', how='left')

    # right join, 왼족 데이터는 모두 보전, 일치되는것만 오른쪽 데이터 포함, 나머진 결측
    tmp = pd.merge( left_df, right_df, on='key', how='right' )

    # outer join -> 쌍방에서 결측 발생 -> 정보 손실x, 결측 많이 나옴
    tmp = pd.merge( left_df, right_df, on='key', how='outer' )

    ---
    pd.concat([left_df, right_df])
    # 수평 합치기 -> 가로가 길어짐
    pd.concat( [left_df, right_df], axis=1 )
```

# 기타
### 중복제거
```
    tmp.drop_duplicates(inplace=True) # 원본 반영
```

### 결측처리
- 데이터 수집 혹은 생성과정에서 누락, 사고등 측정값이 없는 경우
- 조치사항
  - 특정값 초기화(보정)
    - 0으로 세팅
    - 이전값을 데이터값으로 사용
      - 시계열 데이터에서 주로 보임
      - 주가/환율(금융데이터), 센서(장비) 스마트팩토리, IOT
    - 이전/다음 데이터의 평균값으로 사용
      - 시계열 데이터에서 주로 보임
  - 삭제
  - 결측데이터가 패턴이 보이거나, 그 수가 많다면 그대로 데이터로 활용

```
    tmp.isna()  # 결측치 판단
```