# 2026-KB-AI-Challenge
금융과 AI 의 만남

## 공모전 개요
<details>
  <summary><b>주제</b></summary>
  
   ### 소상공인 금융 지원 에이전트

     - 지역 상권, 경기지표, 소비 트렌드, 정책자금 정보를 종합적으로 분석하여 소상공인의 경영 의사결정을 지원
     - 창업 및 운영 단계에 필요한 금융 상품을 추천하고, 정부 지원 사업과 정책자금을 안내하며, 경영 환경 변화에 따른 맞춤형 금융 제안

</details>


## 기획
<details>
  <summary><b>문제 정의</b></summary>

  ### 문제 정의

  #### 기존 소상공인 금융 지원의 문제점

  창업 예정자와 기존 소상공인은 정책지원사업과 금융상품을 찾기 위해 여러 기관 사이트를 직접 방문해야 한다.
  
  대표적으로
  
  - 기업마당
  - K-Startup
  - 소상공인24
  - 금융감독원 금융상품한눈에
  - 은행 홈페이지
  
  등을 모두 찾아봐야 한다.
  
  그러나
  
  - 지원사업 수가 매우 많고
  - 정책 조건이 복잡하며
  - 자신에게 맞는 정책인지 판단하기 어렵고
  - 정책과 금융상품을 함께 추천해주는 서비스가 존재하지 않는다.
  
  또한 은행에서는 금융상품만 추천하고 정부에서는 정책만 제공하기 때문에
  
  사용자는 "지금 내 상황에서 무엇부터 신청해야 하는지" 판단하기 어렵다.

  ---

  #### 문제점 요약
  
  - 정책 정보가 여러 기관에 분산
  - 사용자 맞춤 추천 부재
  - 정책과 금융상품의 연계 부족
  - 검색 비용 증가
  - 창업 초기 정보 접근 어려움

</details>

<details>
  <summary><b>타겟</b></summary>

  ### 1차 타겟

  창업 예정자
  
  - 처음 사업 시작
  - 금융지식 부족
  - 어디에 창업해야 하는지 모름
  - 정책자금 존재도 모름

### 2차 타겟

  운영 중인 소상공인
  
  - 매출 감소
  - 추가 대출 필요
  - 정부지원사업 찾기 어려움
  - 폐업 고민

</details>

<details>
  <summary><b>목표</b></summary>
  
  ### 목표

  > "창업을 준비하거나 사업을 운영하는 소상공인이 현재 상황에 맞는 금융상품과 정부지원사업을 가장 빠르고 정확하게 추천받는 AI"
  
  **AI 금융 컨설턴트**
  
  - 창업 가능성 분석
  - 정부지원사업 추천
  - 정책자금 추천
  - KB 금융상품 추천
  - 다른 은행과 비교
  - 신청 준비까지 안내

  </details>


<details>
  <summary><b>시스템 구조</b></summary>

  ### 시스템 구조

  사용자 DB
  
↓

사용자 프로필 생성

↓

LLM 검색 키워드 생성

↓

RAG 정책 검색

↓

정책 후보

↓

금융상품 DB

↓

AI 통합 추천

↓

최종 컨설팅

 </details>

## 코드구현 정리
<details>
  <summary><b>STEP1</b></summary>

  ## 사용자 데이터 구축

### 목적

사용자의 사업 상황을 정형 데이터로 저장하여 AI가 개인 맞춤형 정책과 금융상품을 추천할 수 있도록 한다.

---

### 구현 내용

business_owner.xlsx 생성

시트

- 예비 창업자
- 기존 사업자

## 사용 기술

- Pandas
- Excel

---

예비창업자

```python
업종

지역

창업자금

희망대출

신용점수

예상매출

창업경험
```

기존사업자

```python
연매출

사업연수

기존대출

사업자산

직원수

신용점수
```

---

### 구현 이유

AI는 사용자 정보를 기반으로 추천을 수행하기 때문에

사용자의 사업 상황을 구조화된 형태로 저장하기 위해 구현하였다.

### 주요 코드

```python
excel = pd.read_excel(
    "business_owner.xlsx",
    sheet_name=None
)
Excel의 모든 시트를 한 번에 불러온다.
```

```python
startup_df = excel["예비 사업자"]
existing_df = excel["기존 사업자"]

사업 단계를 기준으로 데이터를 분리한다.
추천 과정에서 사업단계별로 다른 Prompt를 적용하기 위해 사용된다.
```
 </details>


<details>
  <summary><b>STEP2</b></summary>

  ## 금융상품 API 수집

## 목적

은행 금융상품 정보를 자동으로 수집하여 추천 데이터베이스를 구축한다.

---

## 사용 기술

- 금융감독원 FinLife Open API
- Requests

---

## 구현 이유

은행 홈페이지를 직접 크롤링하지 않고

공식 Open API를 이용하여

신뢰성 있는 금융상품 데이터를 수집하기 위해 구현하였다.

### 구현 내용

금융감독원

FinLife Open API

활용

수집 정보

- 은행명
- 상품명
- 평균금리
- 최대금리
- 최소금리
- 가입방법
- 대출한도

---

### 주요 코드

```python
response = requests.get(url, params=params)
FinLife API에 요청을 보내 금융상품 정보를 받아온다.
```

```python
data = response.json()
JSON 데이터를 Python Dictionary 형태로 변환한다.
```

```python
base_df = pd.DataFrame(data["result"]["baseList"])
상품 기본정보를 DataFrame으로 변환한다.
```

```python
option_df = pd.DataFrame(data["result"]["optionList"])
금리정보를 DataFrame으로 변환한다.
```

```python
financial_products = pd.merge(
    base_df,
    option_df,
    on="fin_prdt_cd"
)

상품정보와 금리정보를 하나의 금융상품 데이터로 통합한다.
```
 </details>


<details>
  <summary><b>STEP3</b></summary>

  ## 정책 데이터 수집

### 목적

정부 정책을 AI가 검색할 수 있도록 정책 데이터베이스를 구축한다.

### 사용 기술

- 기업마당 Open API
- Requests

---

### 구현 이유

정책은 여러 기관에 분산되어 있으므로

검색 가능한 하나의 데이터셋으로 구축하기 위해 구현하였다.

### 구현 내용

기업마당 Open API

활용

수집 정보

- 정책명
- 지원대상
- 지원내용
- 신청기간
- 신청방법
- 기관
- URL

---

### 주요 코드

```python
response = requests.get(url, params=params)
기업마당 API를 호출한다.
```

```python
items = data["jsonArray"]
정책 목록만 추출한다.
```

```python
all_data.extend(items)
페이지별 정책 데이터를 하나의 리스트로 누적 저장한다.
```

```python
policy_df = pd.DataFrame(all_data)
정책 데이터를 DataFrame으로 변환한다.
```
 </details>

<details>
  <summary><b>STEP4</b></summary>

  ## 정책 데이터 전처리

### 목적

정책 검색 성능을 높이기 위해 문서를 정제한다.

### 사용 기술

- BeautifulSoup
- Pandas

### 구현 이유

기업마당 API에는 HTML 태그가 포함되어 있어 Embedding 품질이 떨어질 수 있다. 이를 제거하여 검색 정확도를 향상시켰다.

### 주요 코드

```python
BeautifulSoup(
    text,
    "html.parser"
).get_text()
HTML 태그 제거
```

```python
fillna("")
결측치를 빈 문자열로 변경
Embedding 생성 시 오류를 방지한다.
```

```python
policy_df["document"] = ...
여러 컬럼을 하나의 긴 문서(Document)로 결합한다
```

### 구현 이유

Vector Search는

하나의 긴 문장을 입력받을 때 검색 성능이 가장 높기 때문이다.

 </details>

 <details>
  <summary><b>STEP5</b></summary>

  ## Embedding 생성

## 목적

텍스트를 숫자 벡터로 변환한다.

## 사용 기술

SentenceTransformer

```python
paraphrase-multilingual-MiniLM-L12-v2
```

### 구현 이유

LLM은 의미 기반 검색을 수행하므로 텍스트를 의미 벡터로 변환해야 한다.

### 주요 코드

```python
embedding = HuggingFaceEmbeddings(...)
문장을 384차원 벡터로 변환하는 모델을 생성한다.
```

 </details>

<details>
  <summary><b>STEP6</b></summary>

  ## Vector DB 구축

## 목적

정책 검색 속도를 높인다.

## 사용 기술

FAISS

## 구현 이유

수백 개 정책을 매번 LLM으로 비교하면 매우 느리다.

FAISS를 이용하면 가장 유사한 정책만 빠르게 검색할 수 있다.

## 주요 코드

```python
FAISS.from_documents(
    documents,
    embedding
)
정책 문서를 Vector DB로 저장한다.
```

```python
vector_db.save_local(...)
생성한 Vector DB를 저장하여 재사용한다.
```

 </details>

<details>
  <summary><b>STEP7</b></summary>

  ## 사용자 프로필 생성

## 목적

사용자의 정보를 자연어 형태로 변환한다.

## 사용 기술

Python 함수

## 구현 이유

LLM은 DataFrame보다

문장 형태를 더 잘 이해한다.

## 주요 코드

```python
make_profile(user)
사용자 정보를 하나의 문장으로 생성한다.
```

 </details>

<details>
  <summary><b>STEP8</b></summary> 

  ## 검색 키워드 생성

## 목적

AI가 검색 전략을 직접 생성한다.

## 사용 기술

Claude Sonnet 4.6

## 구현 이유

사용자가 직접 검색어를 입력하지 않아도

AI가 가장 적합한 검색어를 생성하도록 구현하였다.

## 주요 코드

```python
search_prompt
검색어 생성 Prompt
```

```python
client.chat.completions.create()
LLM이 검색 키워드를 생성한다.
```

 </details>

<details>
  <summary><b>STEP9</b></summary>

  ## 정책 검색(RAG)

## 목적

사용자에게 적합한 정책만 검색한다.

## 사용 기술

FAISS Retriever

## 구현 이유

전체 정책을 LLM에 전달하면 토큰이 낭비된다. 따라서 유사도가 높은 정책만 검색한다.

## 주요 코드

```python
retriever.invoke(query)
가장 유사한 정책 Top5 검색
```

```python
all_docs.extend(docs)
여러 검색어의 결과를 합친다.
```

```python
seen = set()
중복 정책 제거
```
 </details>

 <details>
  <summary><b>STEP10</b></summary>

# STEP 10

## 금융 상품 추천

## 목적

사용자에게 적합한 금융상품을 추천한다.

## 사용 기술

Pandas

## 구현 이유

은행별 동일 상품이 여러 개 존재하므로

대표 상품만 추천하도록 구현하였다.

또한 KB국민은행 공모전 특성을 반영하여

KB 상품을 우선 추천하도록 설계하였다.

## 주요 코드

```python
loan_recommend["kb_priority"]
국민은행 상품 여부를 판단한다.
```

```python
sort_values()
국민은행 우선 -> 금리순 정렬
```

```python
drop_duplicates()
은행별 대표상품 1개만 선택
```

```python
loan_context
금융상품 정보를 Prompt에 전달하기 위한 문자열 생성
```

---

## 최종 AI 추천

## 목적

정책과 금융상품을 함께 추천하는 AI 컨설팅을 수행한다.

## 사용 기술

Claude Sonnet 4.6 (LLM)

## 구현 이유

정책만 추천하는 것이 아니라

정책과 금융상품을 함께 활용하는 전략을 제안하기 위해 구현하였다.

## 주요 코드

```python
final_prompt
최종 AI 컨설팅 Prompt
```

```python
response = client.chat.completions.create(...)

LLM이
-사용자 분석
-정책 추천
-금융상품 추천
-신청 순서
-활용 전략
을 종합적으로 생성한다.
```
 </details>
  

  

  

  

     

  
