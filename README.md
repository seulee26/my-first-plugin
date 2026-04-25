# my-first-plugin

A Claude Code plugin: Korean daily retrospective & next-day planning agent team, with **optional** Google Calendar integration that uses **each user's own** Google account via Claude.ai's Google Calendar Connector. No API keys. No shared credentials. Nothing leaves your Claude session.

> 한국어 일일 회고 & 다음 날 플래닝을 위한 Claude Code 플러그인. 구글 캘린더 연동은 선택이고, **각 사용자가 자기 구글 계정을 직접 연결**하는 방식이다 (API 키 없음, 자격증명 공유 없음).

## What you get

- **4 subagents** — `daily-summary` → `blocker-resolver` → `task-breaker` → `schedule-writer` (optional)
- **3 skills** — `calendar-context`, `daily-log`, `next-day-priority`
- **1 slash command** — `/daily`
- **2 hooks** — `SessionStart` reminder, `UserPromptSubmit` hint

## Install (Claude Code)

```text
/plugin marketplace add seulee26/my-first-plugin
/plugin install my-first-plugin@my-first-plugin
```

That's it — restart Claude Code if needed, then run `/daily` to try it.

## Connect your own Google Calendar (optional)

This plugin **does not** ship with any Google credentials. It just *uses* the Google Calendar tools if you've connected your own Google account to Claude. To enable it:

1. Open **Claude.ai** in your browser → click your profile → **Connectors** (or **Settings → Connectors**).
2. Find **Google Calendar** and click **Connect**.
3. Sign in with **your own Google account** and grant access.
4. That's it. Claude Code now sees Google Calendar tools (`mcp__claude_ai_Google_Calendar__*`) inside your session, and this plugin will use them.

If you skip this step, the plugin still works — it just runs without calendar context (the `calendar-context` skill silently no-ops, `schedule-writer` won't be invoked).

If you ever see a permission error, the plugin will tell you to "re-authenticate Google Calendar in Claude.ai → Connectors". It will never ask for an API key.

## Usage

```text
/daily 오늘 한 일을 자유서술로 적어줘. 회의도 결정도 결과도.
```

The flow:

1. **`calendar-context` skill** — pulls today's events + tomorrow's events + tomorrow's free slots from *your* calendar.
2. **`daily-summary` agent** — outputs (성과 3줄 / 걸리는 것 2가지 / 내일 1순위 1가지).
3. **`blocker-resolver` agent** — proposes 2~3 resolution options per blocker.
4. **`task-breaker` agent** — breaks tomorrow's #1 task into 30-minute steps, slotted into your calendar's free time.
5. **`schedule-writer` agent** *(only when you say "캘린더에 등록해")* — dry-runs the schedule, asks for confirmation, then creates the events.

You can stop at any step ("요약만", "블로커만", "캘린더 빼") or skip Google Calendar entirely.

## Privacy & data flow

- **Plugin contains no user data.** This repository ships only agent/skill/hook definitions.
- **Calendar access is your account only.** Authentication happens once in Claude.ai (OAuth in your browser). The plugin cannot impersonate other users or share access between accounts.
- **No API key, no service account, no third-party server.** All Google Calendar calls are performed by Claude itself through the Connector you authorize.
- **Local-only state.** The optional `~/.claude-daily/tomorrow.md` file (used by the `SessionStart` hook to remind you of yesterday's planned #1 task) lives only on your machine.

## Directory layout

```
my-first-plugin/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── CLAUDE.md
├── README.md
├── LICENSE
├── agents/
│   ├── daily-summary.md
│   ├── blocker-resolver.md
│   ├── task-breaker.md
│   └── schedule-writer.md
├── skills/
│   ├── calendar-context/SKILL.md
│   ├── daily-log/SKILL.md
│   └── next-day-priority/SKILL.md
├── commands/
│   └── daily.md
└── hooks/
    └── hooks.json
```

## License

MIT — see [LICENSE](./LICENSE).
