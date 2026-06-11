---
name: hmpe-review-and-fix
description: >
  HMPE 코팅 뉴스 리포트 시스템(C:\HMPE 레포 + 스케줄 태스크 SKILL.md)을 검토하여
  버그는 즉시 수정·커밋·푸시하고, 개선점은 GitHub 이슈로 등록한 뒤 순서대로 처리하는
  end-to-end 리뷰·픽스 워크플로우.
  사용자가 "리뷰하고 이슈 등록해줘", "버그 찾아서 고쳐줘", "개선점 이슈로 등록하고
  처리해줘", "지금까지 작업 검토해줘", "HMPE 시스템 점검해줘" 등을 말할 때 반드시
  이 스킬을 사용하라.
---

## HMPE 리뷰 & 픽스 워크플로우

end-to-end 흐름: **검토 → 분류 → 버그 즉시 수정 → 이슈 등록 → 이슈 순차 처리 → 요약**

---

### Step 1 — 대상 파일 검토

아래 두 대상을 병렬로 읽어라:

1. **C:\HMPE** 레포 전체 파일 (`git log --oneline -10`, 각 파일 내용)
2. **C:\Users\Admin\.claude\scheduled-tasks\hmpe-news-daily\SKILL.md**

검토 시 다음 항목을 확인하라:

| 카테고리 | 체크 항목 |
|---|---|
| 하드코딩 | 날짜·연도·경로·사용자명이 고정값으로 박혀 있는가 |
| 보안 | 외부 링크에 `target="_blank" rel="noopener noreferrer"` 누락 여부 |
| 인코딩 | PowerShell `Set-Content`/`Out-File` 사용 여부 (한글 깨짐 원인) |
| 자동화 | git user 설정 누락으로 커밋 실패 가능성 |
| UX | README·index.html 등 탐색 편의 파일 부재 |
| 에러 처리 | 검색 0건·파일 없음 등 예외 상황 미처리 |
| 누락 기능 | 한국어 검색, 실제 실행 시각 반영 등 |

---

### Step 2 — 버그 vs 개선점 분류

| 유형 | 기준 | 처리 방식 |
|---|---|---|
| **Bug** | 실행 시 오류·데이터 손상·보안 결함·깨진 링크 | 즉시 수정 후 커밋·푸시 |
| **Enhancement** | 누락 기능·UX 개선·리팩터링 기회 | GitHub 이슈 등록 후 순차 처리 |

경계가 모호하면 이슈로 등록하는 쪽을 선택하라 (놓치는 것보다 낫다).

---

### Step 3 — 버그 즉시 수정

파일 수정 시 **반드시 Edit 또는 Write 도구**를 사용하라.
PowerShell `Set-Content` / `Out-File`은 한글이 깨지므로 절대 사용 금지.

수정 완료 후:

```powershell
Set-Location "C:\HMPE"
git config user.email "jaeui2000-del@users.noreply.github.com"
git config user.name "jaeui2000-del"
git add .
git commit -m "Fix: <수정 내용 한 줄 요약>"
git push origin main
```

버그가 없으면 이 단계를 건너뛰어라.

---

### Step 4 — GitHub 이슈 등록

#### 4-1. 토큰 획득 (Bash 도구 사용)

```bash
printf "protocol=https\nhost=github.com\n" | git credential fill
```

출력의 `password=` 값이 토큰이다.

#### 4-2. 이슈 JSON 작성 → POST

각 이슈를 `$env:TEMP\issue_N.json`에 저장 후 curl로 POST하라.
JSON을 curl 인라인에 직접 넣으면 한글·특수문자가 깨지므로 반드시 파일 경유.

```powershell
# 예시: 이슈 1개 등록
$issue = @{
    title  = "[Enhancement] 제목"
    body   = "## 문제`n설명`n`n## 제안`n내용"
    labels = @("enhancement")
} | ConvertTo-Json -Depth 3
[System.IO.File]::WriteAllText("$env:TEMP\issue_1.json", $issue, [System.Text.Encoding]::UTF8)
```

```bash
curl -s -X POST "https://api.github.com/repos/jaeui2000-del/HMPE/issues" \
  -H "Authorization: token TOKEN" \
  -H "Content-Type: application/json" \
  --data-binary "@/c/Users/Admin/AppData/Local/Temp/issue_1.json"
```

등록 후 응답의 `number`와 `html_url`을 기록하라. 완료 후 임시 파일 삭제.

---

### Step 5 — 이슈 순차 처리

등록된 이슈를 번호 순서대로 처리하라. 각 이슈마다:

1. **코드/파일 수정** — Edit 또는 Write 도구
2. **커밋 & 푸시**
   ```powershell
   Set-Location "C:\HMPE"
   git add .
   git commit -m "Fix #N: <이슈 제목 요약>"
   git push origin main
   ```
3. **이슈 close**
   ```bash
   curl -s -X PATCH "https://api.github.com/repos/jaeui2000-del/HMPE/issues/N" \
     -H "Authorization: token TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"state":"closed"}'
   ```

SKILL.md(스케줄 태스크) 수정이 필요한 이슈가 여럿일 경우, 한 번에 모아서 수정 후 단일 커밋으로 처리해도 된다.

---

### Step 6 — 요약 리포트 출력

모든 처리가 끝나면 아래 형식으로 출력하라:

```
## 리뷰 완료

### 버그 수정 (커밋·푸시 완료)
- [파일명] 수정 내용

### 이슈 등록 및 처리 완료
- #N [제목] — https://github.com/jaeui2000-del/HMPE/issues/N

### 발견 없음
(해당 없으면 "발견된 이슈가 없습니다."로 표시)
```

---

## 환경 참고

| 항목 | 값 |
|---|---|
| 레포 경로 | `C:\HMPE` |
| 스케줄 태스크 | `C:\Users\Admin\.claude\scheduled-tasks\hmpe-news-daily\SKILL.md` |
| GitHub 레포 | `jaeui2000-del/HMPE` |
| git user.email | `jaeui2000-del@users.noreply.github.com` |
| git user.name | `jaeui2000-del` |
| 토큰 획득 | Bash: `printf "protocol=https\nhost=github.com\n" \| git credential fill` |
| 파일 수정 도구 | Edit / Write (PowerShell Set-Content 절대 금지) |
