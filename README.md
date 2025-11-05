# WSJ 뉴스 분석 워크플로우

매일 한국 시간 오전 8시에 WSJ Finance 섹션을 크롤링하여 종합 분석 문서를 생성하는 멀티 에이전트 시스템입니다.

## 빠른 시작

Cursor 채팅에서 다음 명령어로 실행:

```
WSJ 뉴스 분석해줘
```

## 시스템 구조

### 에이전트 워크플로우

```
@00_orchestrator (메인)
  ├─> @utils/crawler (크롤링)
  ├─> @utils/categorizer (카테고리 분류)
  ├─> @utils/analyzer (기사 분석)
  ├─> @utils/fact_checker (팩트 체크)
  └─> @utils/documenter (문서화)
```

### 디렉토리 구조

```
.agents/
  ├── 00_orchestrator.md          # 메인 워크플로우
  ├── DESIGN_DISCUSSION.md        # 설계 토론 기록
  ├── utils/
  │   ├── crawler.md              # 크롤링 에이전트
  │   ├── categorizer.md          # 카테고리 분류 에이전트
  │   ├── analyzer.md              # 기사 분석 에이전트
  │   ├── fact_checker.md          # 팩트 체크 에이전트
  │   └── documenter.md           # 문서화 에이전트
  └── templates/
      └── m_and_a_template.md     # M&A 템플릿 (예시)

data/
  └── YYYYMMDD/
      ├── raw_articles.json
      ├── categorized.json
      ├── analyzed_content.json
      ├── verified.json
      ├── articles/
      │   └── [article_id]_original.txt
      └── session.json
```

## 주요 기능

### 1. 자동 크롤링
- WSJ Finance 섹션에서 기사 목록 수집
- 로그인 세션 관리 및 재사용
- 시장 지수 데이터 자동 추출

### 2. 스마트 분류
- 기사 제목/미리보기 기반 초기 분류
- 기사 내용 기반 재분류
- 10개 주요 카테고리 지원

### 3. 심층 분석
- 각 기사 전체 내용 분석
- 원문 텍스트 저장 (재사용 가능)
- 핵심 사실 구조화 추출

### 4. 팩트 체크
- 원문 기반 검증
- Hallucination 방지
- 검증 수준 표시 (✅ ⚠️ ❌)

### 5. 문서화
- 템플릿 기반 일관성 유지
- 검증 완료 사항 자동 포함
- 예시 파일 형식 준수

## 사용 예시

### 전체 워크플로우 실행

```
@00_orchestrator WSJ Finance 섹션 분석 실행
```

### 특정 날짜 분석

```
@00_orchestrator 2025년 11월 3일 뉴스 분석
```

### 부분 실행

```
@00_orchestrator 팩트 체크만 다시 해줘
@utils/documenter 문서 생성
```

## 에이전트별 역할

### Orchestrator
- 전체 워크플로우 조율
- 진행 상황 관리
- 에러 처리 및 재시도

### Crawler
- WSJ 로그인 및 세션 관리
- 기사 목록 수집
- 시장 지수 데이터 추출

### Categorizer
- 기사 카테고리 분류
- 키워드 매칭
- 우선순위 결정

### Analyzer
- 기사 내용 분석
- 원문 텍스트 저장
- 핵심 사실 추출

### Fact Checker
- 원문 기반 검증
- Hallucination 방지
- 검증 결과 기록

### Documenter
- 마크다운 문서 생성
- 템플릿 적용
- 최종 문서 저장

## 설계 특징

1. **모듈화**: 각 에이전트가 독립적으로 작동
2. **재사용성**: 중간 결과 저장으로 부분 재실행 가능
3. **효율성**: 로그인 세션 및 원문 텍스트 재사용
4. **신뢰성**: 팩트 체크 및 검증 시스템
5. **유연성**: 부분 실행 및 커스터마이징 지원

## 제약사항

- WSJ 로그인 필요 (Playwright 사용)
- Cursor 구독비용만 사용 (외부 API 비용 없음)
- 처리 시간: 약 10-15분 (기사 50개 기준)

## 참고 문서

- `.agents/DESIGN_DISCUSSION.md`: 설계 토론 기록
- `.agents/00_orchestrator.md`: 워크플로우 상세
- `WSJ_Finance_Daily_Analysis_251104.md`: 출력 예시

## 문제 해결

### 로그인 실패
- 로그인 정보 확인
- 세션 파일 삭제 후 재시도 (`data/YYYYMMDD/session.json`)

### 기사 접속 실패
- 일부 기사 실패는 정상 (부분 실패 허용)
- 실패한 기사는 `data/YYYYMMDD/failed_articles.json`에 기록

### 검증 오류
- 원문 파일 확인 (`data/YYYYMMDD/articles/`)
- 필요시 Analyzer 단계 재실행

## 라이선스

이 프로젝트는 개인 사용 목적으로 작성되었습니다.

