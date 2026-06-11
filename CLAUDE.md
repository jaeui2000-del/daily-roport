# CLAUDE.md — Claude Code 작업 지침

## 리포트 생성 절차

1. 오늘 날짜 확인 → 파일명: `YYYY-MM-DD.html`
2. 웹 검색으로 각 섹션 뉴스 수집 (Claude, ChatGPT, Gemini, 에이전트 트렌드)
3. `2026-06-11.html`을 템플릿으로 복사해 내용 교체
4. 날짜·요일·생성 시각을 실제 값으로 업데이트
5. `git add`, `git commit -m "Add daily report YYYY-MM-DD"`, `git push`

## HTML 컨벤션

- 모든 외부 링크: `target="_blank" rel="noopener noreferrer"` 필수
- 파일 수정 시 Edit 도구 사용 (PowerShell Set-Content는 한글 깨짐)
- 인코딩: UTF-8

## 뉴스 작성 기준

- 각 섹션 뉴스 2건
- 출처 링크 반드시 포함
- `💡 초보자 포인트:` 블록에 전문 용어 풀이 포함

## GitHub

- 레포: `jaeui2000-del/daily-roport`
- 브랜치: `main`
- 이슈 등록 시 `/review-and-file-issues` 스킬 사용
