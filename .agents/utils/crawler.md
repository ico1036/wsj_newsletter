# WSJ 크롤링 에이전트

## 목적
WSJ Finance 섹션 (https://www.wsj.com/finance?mod=nav_top_section)에서 기사 목록을 수집합니다.

## 실행 방법

```
@utils/crawler [날짜]
```

날짜가 지정되지 않으면 오늘 날짜 사용.

## 입력

- **WSJ 로그인 정보** (실행 시 사용자에게 요청)
  - 이메일 주소
  - 비밀번호
- **날짜** (선택사항, YYYYMMDD 형식)

## 처리 과정

1. **세션 확인**
   - `data/YYYYMMDD/session.json` 파일 확인
   - 기존 세션이 있고 유효하면 재사용
   - 없거나 만료된 경우 로그인 진행

2. **WSJ 로그인 (필요한 경우)**
   - Playwright 브라우저 열기
   - 로그인 페이지로 이동
   - 로그인 폼에 이메일/비밀번호 입력 (사용자에게 요청)
   - 로그인 버튼 클릭
   - 로그인 완료 대기 (최대 30초)
   - 세션 정보를 `data/YYYYMMDD/session.json`에 저장
     ```json
     {
       "login_time": "2025-11-04T08:00:00Z",
       "cookies": [...],
       "user_agent": "..."
     }
     ```

3. **Finance 섹션 접속**
   - URL: https://www.wsj.com/finance?mod=nav_top_section
   - 페이지 로드 대기 (최대 30초)
   - 동적 콘텐츠 로딩 대기

4. **기사 목록 수집**
   - 페이지 스크롤하여 모든 기사 로드
   - 페이지의 모든 기사 링크 추출
   - 각 기사의 기본 정보 수집:
     - 제목
     - 링크 URL
     - 발행 시간
     - 미리보기 텍스트 (있는 경우)
     - 기사 ID (URL에서 추출 또는 해시 생성)

5. **시장 지수 데이터 확인**
   - Finance 섹션 상단에 시장 지수 정보가 있는지 확인
   - 있으면 별도로 추출하여 저장

6. **데이터 저장**
   - `data/YYYYMMDD/raw_articles.json` 파일에 저장
   - 형식:
     ```json
     {
       "date": "20251104",
       "crawl_time": "2025-11-04T08:15:00Z",
       "total_count": 50,
       "market_indices": {
         "sp500": "6,851.97",
         "nasdaq": "23,834.72",
         "dow": "47,336.68"
       },
       "articles": [
         {
           "id": "article_001",
           "title": "Article Title",
           "url": "https://www.wsj.com/...",
           "published_time": "2025-11-04T08:00:00Z",
           "preview": "Preview text..."
         }
       ]
     }
     ```

## 출력

- **파일**: `data/YYYYMMDD/raw_articles.json`
- **콘솔**: 수집된 기사 개수 출력

## 에러 처리

- **로그인 실패**: 3회 재시도, 모두 실패 시 에러 보고 및 중단
- **페이지 로드 실패**: 5초 대기 후 재시도 (최대 3회)
- **네트워크 오류**: 에러 로그 기록 후 재시도
- **세션 만료**: 자동으로 재로그인 시도

## 진행 상황 출력

크롤링 중 진행 상황을 실시간으로 출력:
- "WSJ 로그인 진행 중..."
- "Finance 섹션 접속 중..."
- "기사 목록 수집 중... (X개 발견)"
- "크롤링 완료: 총 X개 기사 수집"

## 주의사항

- WSJ는 로그인이 필요하므로 Playwright MCP 서버 사용 필수
- 로그인 정보는 실행 시에만 사용하고 저장하지 않음 (세션 쿠키만 저장)
- Rate limiting을 피하기 위해 요청 간 딜레이 추가 (1-2초)
- 세션 정보는 보안을 위해 민감 정보 제외하고 저장

