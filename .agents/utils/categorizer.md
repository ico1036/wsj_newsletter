# 카테고리 분류 에이전트

## 목적
크롤링된 기사들을 카테고리별로 분류합니다.

## 실행 방법

```
@utils/categorizer [날짜]
```

날짜가 지정되지 않으면 오늘 날짜 사용. `data/YYYYMMDD/raw_articles.json` 파일을 읽어서 처리합니다.

## 입력

- **파일**: `data/YYYYMMDD/raw_articles.json` (크롤링 결과)

## 카테고리 목록

다음 카테고리로 분류합니다:

1. **시장 개황 (Market Overview)**
   - 주요 지수 동향
   - 시장 동인
   - 일일 변동률

2. **M&A 및 기업 인수합병**
   - 인수합병 거래
   - 기업 분할
   - 거래 구조 및 가격

3. **금융 산업**
   - 은행 업계
   - 신용평가사
   - 금융 규제
   - FICO, 신용점수 관련

4. **은행 및 금융 서비스**
   - 개별 은행 뉴스
   - 금융 서비스 산업

5. **빅테크 & AI**
   - AI 투자
   - 클라우드 서비스
   - Magnificent Seven
   - 기술 기업 인수합병

6. **중국 관련**
   - 중국 시장 진출/철수
   - 대중 무역
   - 중국 규제

7. **정책 & 정부**
   - 정부 셧다운
   - 규제 정책
   - 정치적 이슈

8. **주식 시장**
   - 일간 최대 상승/하락주
   - 개별 주식 분석

9. **원자재 & 선물**
   - 금, 원유 가격
   - 비트코인
   - 선물 시장

10. **전문가 분석 (Heard on the Street)**
    - 칼럼
    - 전문가 의견

## 처리 과정

1. **입력 파일 읽기**
   - `data/YYYYMMDD/raw_articles.json` 파일 로드

2. **기사 제목 및 미리보기 분석**
   - 각 기사의 제목과 미리보기 텍스트를 분석
   - 키워드 매칭을 통해 카테고리 결정
   - 여러 카테고리에 속할 수 있는 경우 우선순위 결정

3. **카테고리별 그룹화**
   - 기사를 카테고리별로 분류
   - 주요 거래/이슈는 ⭐ 표시

4. **데이터 저장**
   - `data/YYYYMMDD/categorized.json` 파일에 저장
   - 형식:
     ```json
     {
       "date": "20251104",
       "categories": {
         "market_overview": [...],
         "m_and_a": [...],
         "finance_industry": [...],
         ...
       }
     }
     ```

## 출력

- **파일**: `data/YYYYMMDD/categorized.json`
- **콘솔**: 카테고리별 기사 개수 출력

## 분류 로직 예시

- "M&A", "acquisition", "merger" → M&A 및 기업 인수합병
- "FICO", "credit score", "VantageScore" → 금융 산업
- "AI", "cloud", "Amazon", "Microsoft", "OpenAI" → 빅테크 & AI
- "China", "Chinese" → 중국 관련
- "government shutdown", "regulatory" → 정책 & 정부
- "stock", "S&P 500", "NASDAQ" → 시장 개황 또는 주식 시장
- "gold", "oil", "bitcoin" → 원자재 & 선물

## 주의사항

- 한 기사가 여러 카테고리에 속할 수 있음
- 중요한 거래나 이슈는 우선순위를 높여 처리
- 불확실한 경우 "기타" 카테고리 사용

