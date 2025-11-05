# 기사 내용 분석 에이전트

## 목적
각 기사 링크로 직접 접속하여 전체 내용을 확인하고 핵심 정보를 추출합니다.

## 실행 방법

```
@utils/analyzer [날짜]
```

날짜가 지정되지 않으면 오늘 날짜 사용. `data/YYYYMMDD/categorized.json` 파일을 읽어서 처리합니다.

## 입력

- **파일**: `data/YYYYMMDD/categorized.json` (카테고리 분류 결과)

## 처리 과정

1. **입력 파일 읽기**
   - `data/YYYYMMDD/categorized.json` 파일 로드
   - 각 카테고리의 기사 목록 확인
   - 기존 세션 확인 (`data/YYYYMMDD/session.json`)

2. **우선순위 정렬**
   - ⭐ 표시된 주요 거래/이슈 우선 처리
   - 카테고리별로 배치 처리

3. **각 기사 내용 확인**
   - 저장된 세션 정보로 Playwright 브라우저 컨텍스트 복원
   - 각 기사 URL에 접속
   - 페이지 로드 대기 (최대 30초)
   - 기사 본문 전체 추출
   - **원문 텍스트 저장**: `data/YYYYMMDD/articles/[article_id]_original.txt`
     - Fact Checker가 재사용할 수 있도록 원문 보존

4. **카테고리 재확인**
   - 기사 내용을 읽은 후 카테고리 재확인
   - 초기 분류와 다를 경우 수정
   - 여러 카테고리에 속할 수 있음

5. **핵심 정보 추출**
   각 기사에서 다음 정보를 추출:
   
   - **기본 정보**
     - 제목
     - 발행 시간
     - 저자
     - 카테고리 (재확인된 것)
     - 기사 ID
   
   - **주요 내용**
     - 핵심 사실 (수치, 날짜, 기업명 등) - 검증 대상으로 표시
     - 거래 구조 (M&A인 경우)
     - 시장 영향
     - 리스크 요인
     - 인용문 및 전문가 의견 (원문 그대로)
   
   - **시장 데이터** (해당하는 경우)
     - 주가 변동률
     - 거래 가액
     - 시장 지수

6. **구조화된 데이터 저장**
   - `data/YYYYMMDD/analyzed_content.json` 파일에 저장
   - 형식:
     ```json
     {
       "date": "20251104",
       "analysis_time": "2025-11-04T08:30:00Z",
       "articles": [
         {
           "id": "article_001",
           "title": "Article Title",
           "url": "https://www.wsj.com/...",
           "category": "m_and_a",
           "category_confidence": "high",
           "original_text_path": "data/20251104/articles/article_001_original.txt",
           "key_facts": {
             "companies": ["Company A", "Company B"],
             "deal_value": "48 billion",
             "premium": "46.2%",
             "timeline": "2026 H2"
           },
           "key_facts_to_verify": [
             {
               "fact_id": "fact_001",
               "fact_text": "거래 가액: 48억 달러",
               "type": "monetary_value",
               "original_text": "원문 인용 부분"
             }
           ],
           "content_summary": "Detailed summary...",
           "quotes": [
             {
               "text": "Quote 1",
               "speaker": "Person Name",
               "original_text": "원문 인용문"
             }
           ],
           "market_impact": "Stock price changes...",
           "risks": ["Risk 1", "Risk 2"]
         }
       ]
     }
     ```

7. **진행 상황 출력**
   - "기사 분석 진행 중... (X/50 완료)"
   - 실패한 기사는 별도로 기록

## 출력

- **파일**: `data/YYYYMMDD/analyzed_content.json`
- **콘솔**: 분석 완료된 기사 개수 출력

## 분석 우선순위

1. **수치 데이터 우선 추출**
   - 금액, 비율, 날짜 등은 정확히 기록
   - 소수점, 백분율 등 형식 유지

2. **인용문 보존**
   - 전문가, 기업 대표 등의 인용문은 원문 그대로 보존
   - 따옴표로 명확히 표시

3. **전후 맥락 포함**
   - 단순 사실 나열이 아닌 맥락 설명
   - 왜, 어떻게, 어떤 영향을 미치는지 포함

4. **거래 구조 상세화** (M&A인 경우)
   - 거래 가액
   - 주당 가격 및 프리미엄
   - 지분 구조
   - 예상 마감일
   - 전략적 배경

## 에러 처리

- **기사 접속 실패**: 3회 재시도, 실패 시 해당 기사 스킵하고 다음 기사로 진행
- **로그인 세션 만료**: Crawler Agent에게 세션 갱신 요청 (또는 재로그인)
- **기사 삭제/접근 불가**: 스킵하고 `data/YYYYMMDD/failed_articles.json`에 기록
- **부분 실패 허용**: 일부 기사 실패해도 나머지는 계속 처리

## 진행 상황 출력

실시간으로 진행 상황 출력:
- "기사 분석 시작: 총 50개 기사"
- "처리 중... (10/50) - [기사 제목]"
- "완료: 48개 성공, 2개 실패"

## 주의사항

- WSJ는 로그인이 필요하므로 저장된 세션 재사용
- Rate limiting을 피하기 위해 요청 간 딜레이 추가 (2-3초)
- 긴 기사의 경우 전체 내용을 읽어야 하므로 충분한 대기 시간 필요
- 원문 텍스트는 Fact Checker가 재사용할 수 있도록 반드시 저장
- 우선순위 기사부터 처리하여 중요한 내용을 먼저 확보

