# my-first-plugin — 일일 회고 & 다음날 플래닝 팀 (구글 캘린더 연동)

이 플러그인은 사용자의 하루 회고와 다음 날 플래닝을 한국어로 도와주는 **4-에이전트 팀** + **3개 스킬** + **1개 슬래시 커맨드** + **1개 훅** 으로 구성된다.

## 구글 캘린더 연동
구글 캘린더는 Claude 세션의 MCP 커넥터(`mcp__claude_ai_Google_Calendar__*`)로 연결된다. **인증은 플러그인이 아니라 Claude.ai → Connectors UI에서 한 번 한다** (플러그인이 OAuth를 들고 있을 수 없다). 권한 오류가 나면 사용자에게 "Claude.ai → Connectors → Google Calendar 재인증" 만 안내하고 캘린더 없이 흐름을 계속한다.

플러그인이 사용하는 캘린더 도구:
- `list_calendars` (필요시)
- `list_events` — 오늘/내일 일정, 충돌 검사
- `create_event` — `schedule-writer` 가 사용자 명시 동의 시에만

## 언제 이 팀을 작동시키나
사용자가 다음 중 하나를 말하면 이 팀의 흐름을 시작한다.
- "오늘 한 일 정리해줘", "데일리 회고", "하루 정리"
- "내일 뭐부터 할까", "내일 우선순위"
- `/daily` 슬래시 커맨드 호출

## 표준 흐름 (캘린더 → 4-에이전트 파이프라인)
0. **`calendar-context`** 스킬 — 오늘 일정 / 내일 일정 / 내일 빈 슬롯 수집 (캘린더 연결 시).
1. **`daily-summary`** 서브에이전트 — 오늘 한 일 입력 + 캘린더 맥락을 받아 (성과 3줄 / 걸리는 것 2가지 / 내일 1순위 1가지) 출력.
2. **`blocker-resolver`** 서브에이전트 — 위 결과의 "걸리는 것 2가지"를 입력받아 각각 해결 옵션 2~3개씩 제시.
3. **`task-breaker`** 서브에이전트 — "내일 1순위 1가지" + 빈 슬롯을 받아 30분 단위 실행 단계로 분해 (예정 슬롯 포함).
4. **`schedule-writer`** 서브에이전트 (선택) — 사용자가 "캘린더에 등록해" 라고 명시한 경우에만, 드라이런 → 확인 → `create_event` 로 실제 이벤트 생성.

순차 호출. 사용자가 "전체 다 돌려줘"면 0→1→2→3 자동 체이닝, 4는 명시 요청 시에만. 단계별로 끊고 싶다는 신호("요약만", "블로커만", "캘린더 빼")가 있으면 거기서 멈춘다.

## 스킬 사용 시점
- **`calendar-context`** — `/daily` 흐름 시작 시 자동. 오늘/내일 캘린더 데이터를 수집해 회고/플래닝 입력 맥락으로 합친다.
- **`daily-log`** — 사용자가 오늘 한 일을 기록하려고 할 때, 또는 입력 형식이 자유서술이라 정리가 필요할 때. 회고 직전에 사용한다.
- **`next-day-priority`** — `daily-summary` 결과의 "내일 1순위"가 모호하거나 후보가 여러 개일 때, 우선순위 결정 프레임을 제공한다.

## 훅 동작
- **SessionStart** — `~/.claude-daily/tomorrow.md` 파일이 존재하면, 어제 기록한 "내일 1순위" 작업을 세션 시작 시 보여준다. 파일이 없으면 조용히 통과한다.

## 출력 규칙 (팀 전체)
- 모든 출력은 **한국어**.
- 칭찬·격려·이모지 금지. 실무자 톤.
- 입력에 없는 사실을 만들어내지 않는다. 추측은 "추정:"으로 표시.
- 각 에이전트의 출력 포맷은 해당 `agents/*.md` 파일을 따른다.

## 디렉터리 구조
```
my-first-plugin/
├── .claude-plugin/plugin.json
├── CLAUDE.md
├── agents/
│   ├── daily-summary.md      # 1단계: 회고
│   ├── blocker-resolver.md   # 2단계: 블로커 해결안
│   ├── task-breaker.md       # 3단계: 1순위 작업 분해 (캘린더 슬롯 인지)
│   └── schedule-writer.md    # 4단계: 캘린더에 실제 등록 (선택)
├── skills/
│   ├── calendar-context/SKILL.md   # 0단계: 오늘/내일 일정 수집
│   ├── daily-log/SKILL.md
│   └── next-day-priority/SKILL.md
├── commands/
│   └── daily.md              # /daily
└── hooks/
    └── hooks.json            # SessionStart + UserPromptSubmit
```
