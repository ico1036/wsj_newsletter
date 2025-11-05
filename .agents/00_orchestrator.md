# WSJ 뉴스 분석 워크플로우 오케스트레이터

## 목적
WSJ Finance 섹션의 뉴스를 수집, 분석, 검증하여 종합 분석 문서를 생성하는 전체 워크플로우를 조율합니다.

## 워크플로우 개요

```
1. 크롤링 (@utils/crawler)
   └─> data/YYYYMMDD/raw_articles.json

2. 카테고리 분류 (@utils/categorizer)
   └─> data/YYYYMMDD/categorized.json

3. 기사 내용 분석 (@utils/analyzer)
   └─> data/YYYYMMDD/analyzed_content.json

4. 팩트 체크 (@utils/fact_checker)
   └─> data/YYYYMMDD/verified.json

5. 문서화 (@utils/documenter)
   └─> WSJ_Finance_Daily_Analysis_YYYYMMDD.md
```

## 실행 방법

### 기본 실행
```
WSJ 뉴스 분석해줘
```

또는

```
@00_orchestrator WSJ Finance 섹션 분석 실행
```

### 특정 날짜 분석
```
@00_orchestrator 2025년 11월 3일 뉴스 분석
```

## 실행 순서

1. **날짜 확인 및 초기화**
   - 날짜가 지정되지 않으면 오늘 날짜 사용 (YYYYMMDD 형식)
   - 한국 시간 기준 오전 8시 이후 데이터 분석
   - `data/YYYYMMDD/` 폴더 생성 (없는 경우)

2. **크롤링 실행**
   - `@utils/crawler` 에이전트 호출
   - WSJ 로그인 정보 요청 (이메일, 비밀번호)
   - 로그인 세션 생성 및 저장 (`data/YYYYMMDD/session.json`)
   - WSJ Finance 섹션 URL에서 기사 목록 수집
   - 각 기사 링크 및 기본 정보 저장
   - 진행 상황: "크롤링 진행 중... (1/6)"

3. **카테고리 분류 (초기)**
   - `@utils/categorizer` 에이전트 호출
   - 크롤링된 기사를 제목/미리보기 기반으로 카테고리별 분류
   - 예: M&A, 금융 산업, 빅테크, 중국 관련 등
   - 진행 상황: "카테고리 분류 진행 중... (2/6)"

4. **기사 내용 분석**
   - `@utils/analyzer` 에이전트 호출
   - 저장된 로그인 세션 재사용
   - 각 기사 링크로 직접 접속하여 전체 내용 확인
   - 원문 텍스트를 `data/YYYYMMDD/articles/[id]_original.txt`에 저장
   - 핵심 정보 추출 및 정리
   - 카테고리 재확인/수정 (기사 내용 기반)
   - 진행 상황: "기사 분석 진행 중... (X/50 완료) (3/6)"

5. **팩트 체크**
   - `@utils/fact_checker` 에이전트 호출
   - 저장된 원문 텍스트 파일 활용 (재접속 최소화)
   - 수치, 날짜, 기업명 등 핵심 사실 검증
   - Hallucination 방지
   - 검증 수준 표시: ✅ 원문 재확인 완료 / ⚠️ 분석 결과만 확인 / ❌ 확인 불가
   - 진행 상황: "팩트 체크 진행 중... (4/6)"

6. **문서화**
   - `@utils/documenter` 에이전트 호출
   - `WSJ_Finance_Daily_Analysis_251104.md` 형식에 맞춰 문서 생성
   - 템플릿 기반 일관성 유지 (`@templates/`)
   - 검증 완료 사항 섹션 자동 생성
   - 최종 마크다운 파일 저장
   - 진행 상황: "문서 생성 완료! (6/6)"

## 에러 처리

- **부분 실패 허용**: 일부 기사 실패해도 나머지는 계속 처리
- **단계별 재실행**: 각 단계 실패 시 해당 단계만 재실행 가능
- **중간 결과 보존**: 모든 중간 결과는 `data/YYYYMMDD/` 폴더에 저장되어 부분 재실행 지원
- **재시도 로직**: 
  - 로그인 실패: 3회 재시도
  - 네트워크 오류: 5초 대기 후 재시도
  - 페이지 로드 실패: 최대 3회 재시도
- **에러 로깅**: 모든 에러는 `data/YYYYMMDD/errors.log`에 기록

## 부분 실행 지원

사용자가 특정 단계만 실행하고 싶을 경우:

```
@00_orchestrator 팩트 체크만 다시 해줘  # 4단계만 실행
@00_orchestrator 문서화만 해줘          # 5단계만 실행
```

각 단계는 필요한 입력 파일이 있는지 확인 후 실행합니다.

## 입력 요구사항

- WSJ 로그인 정보 (실행 시 프롬프트로 요청)
  - 이메일
  - 비밀번호

## 출력

- **최종 분석 문서**: `WSJ_Finance_Daily_Analysis_YYYYMMDD.md`
- **중간 데이터**: `data/YYYYMMDD/` 폴더 내:
  - `raw_articles.json`: 크롤링 결과
  - `categorized.json`: 카테고리 분류 결과
  - `analyzed_content.json`: 기사 분석 결과
  - `verified.json`: 팩트 체크 결과
  - `session.json`: 로그인 세션 정보 (선택사항)
  - `articles/`: 각 기사 원문 텍스트 파일
  - `errors.log`: 에러 로그

## 실행 통계

완료 후 다음 정보를 출력:
- 총 처리 시간
- 수집된 기사 개수
- 카테고리별 기사 개수
- 검증 완료된 사실 개수
- 실패한 기사 개수 (있는 경우)

