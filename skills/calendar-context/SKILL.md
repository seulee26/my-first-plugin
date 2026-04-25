---
name: calendar-context
description: 회고/플래닝 시 사용자의 구글 캘린더에서 오늘 일정과 내일 일정을 가져와 daily-summary 입력의 맥락으로 합친다. /daily 흐름의 0단계로 보통 자동 호출. "오늘 일정 가져와", "내일 캘린더 봐줘", "회고에 일정 같이" 같은 요청에 트리거.
---

# 캘린더 맥락 수집 스킬

## 전제
구글 캘린더는 Claude 세션의 MCP 커넥터 (`mcp__claude_ai_Google_Calendar__*`) 를 통해 이미 연결되어 있다. 인증 흐름은 Claude.ai의 Connectors UI에서 이미 완료되었다고 가정한다. 도구가 권한 오류를 내면 사용자에게 "Claude.ai → Connectors → Google Calendar 재인증 필요" 라고만 안내하고 흐름을 멈춘다.

## 사용하는 MCP 도구
- `mcp__claude_ai_Google_Calendar__list_calendars` — 캘린더 목록 (필요시)
- `mcp__claude_ai_Google_Calendar__list_events` — 오늘 / 내일 이벤트 조회

## 처리 절차
1. 시스템 컨텍스트의 `currentDate` 와 사용자 타임존 (기본 `Asia/Seoul`) 기준으로:
   - 오늘 범위: `YYYY-MM-DDT00:00:00+09:00` ~ 다음날 `T00:00:00+09:00`
   - 내일 범위: 그 다음 24시간
2. `list_events` 호출 시 `orderBy=startTime`, `timeZone=Asia/Seoul` 고정.
3. 두 호출은 **병렬**로 실행한다.
4. `eventType` 이 `outOfOffice`, `workingLocation`, `birthday` 인 항목은 표기는 하되 회고/플래닝 우선순위에서 제외 표시.
5. 보안상 다른 참석자 이메일은 도메인만 남기고 마스킹하지 않는다(사용자 본인 데이터). 단, 사용자가 출력 공유를 의도한 경우 별도 요청 시 마스킹.

## 출력 포맷 (한국어)

```
## 캘린더 맥락 (YYYY-MM-DD KST 기준)

### 오늘 일정 (N건)
- HH:MM–HH:MM  {일정 제목}
- HH:MM–HH:MM  {일정 제목}
(없으면: "오늘 등록된 일정 없음")

### 내일 일정 (M건)
- 10:00–11:00  ...
(없으면: "내일 등록된 일정 없음")

### 비어있는 30분 이상 슬롯 (내일, 09:00~22:00 기준)
- 11:00–13:00 (2시간)
- 16:00–22:00 (6시간)
```

## 사용 흐름
- `daily-summary` 호출 직전, 이 스킬 결과를 사용자 자유서술 입력과 함께 묶어 전달한다.
- `task-breaker` 가 시간 슬롯을 제안할 때, "내일 일정"과 "비어있는 슬롯" 항목을 참조한다.
- `schedule-writer` 가 캘린더에 이벤트를 만들 때, 같은 슬롯 정보를 충돌 검사용으로 사용한다.

## 규칙
- 사용자에게 데이터를 보여주는 출력은 위 포맷만. 원본 JSON, htmlLink, eventId 노출 금지.
- 호출 실패 시 흐름을 막지 말고 "캘린더 조회 실패 — 캘린더 없이 진행" 한 줄만 남기고 다음 단계로 넘긴다.
