# Flue 완전 정복 가이드 (한국어 정리본) 🚀

> 이 문서는 `bmshin94/flue` 저장소를 전수조사하며 나눈 대화를 정리한 학습/전략 노트입니다.
> 작성일: 2026-09-28

## 🔗 관련 링크

| 구분 | 주소 |
| --- | --- |
| 내 저장소 (fork) | https://github.com/bmshin94/flue |
| 원본 저장소 (upstream) | https://github.com/withastro/flue |
| 공식 문서 / 홈페이지 | https://flueframework.com |
| 모델 계층(Pi) | https://pi.dev |
| 스킬 표준 | https://agentskills.io |
| 스타 히스토리 | https://www.star-history.com/withastro/flue/ |
| 외부 소개 글 | https://betterstack.com/community/guides/ai/flue-framework/ |

---

## 목차

1. [Flue란 무엇인가](#1-flue란-무엇인가)
2. [저장소 전수조사 결과](#2-저장소-전수조사-결과)
3. [핵심 개념 쉽게 이해하기](#3-핵심-개념-쉽게-이해하기)
4. [설치 및 사용법](#4-설치-및-사용법)
5. [자주 묻는 질문 7가지](#5-자주-묻는-질문-7가지)
6. [수익화 아이디어 12선](#6-수익화-아이디어-12선)
7. [추천 로드맵](#7-추천-로드맵)

---

## 1. Flue란 무엇인가

**한 줄 요약**

> Flue = "Claude Code 같은 자율 에이전트를, 내 서비스 안에 내가 직접 만들어 넣게 해주는 TypeScript 프레임워크"

- 제작: **Astro 팀 (withastro)**
- 버전: **v2.0.8** (28개 패키지 동일 버전)
- 라이선스: **Apache-2.0** (상업적 이용 가능)
- README 캐치프레이즈: *"Not another SDK."*

### 기존 방식과의 차이

| 항목 | 기존 SDK 직접 호출 | Flue |
| --- | --- | --- |
| 구조 | `messages.create()` 호출 → 응답 | 에이전트를 **함수 하나**로 정의 |
| 자율성 | 내가 짠 순서대로만 동작 | 목표만 주면 **알아서** 도구 선택 |
| 상태 | 직접 DB 저장 구현 | **자동 영속화 + 크래시 복구** |
| 코드 실행 | 직접 구현 | **샌드박스 내장** |

### 대표 코드

```ts
'use agent';                                   // "나 에이전트임" 표시
import { useModel, useSubagent } from '@flue/runtime';

export function WithSubagent() {               // 대문자 export 함수 = 에이전트 1개
  useModel('anthropic/claude-sonnet-4-6');     // 훅으로 능력 장착
  useSubagent({ name: 'greeter', agent: Greeter });

  return '인사할 일이 생기면 greeter 서브에이전트한테 맡기고 결과를 그대로 보고해.';
  //     ↑ return 값이 곧 시스템 프롬프트
}
```

**React와의 대응 관계**

| React | Flue |
| --- | --- |
| `useState` | `usePersistentState` |
| `useEffect` | `useAgentStart` / `useResponseFinish` |
| 컴포넌트 함수 | 에이전트 함수 |
| JSX 반환 | **프롬프트 문자열** 반환 |

---

## 2. 저장소 전수조사 결과

```
flue/
├── packages/     ← 28개 npm 패키지 (핵심)
├── examples/     ← 28개 실전 예제
├── blueprints/   ← 44개 마크다운 설치 가이드
├── apps/docs/    ← 공식 문서 사이트 (Astro)
├── apps/www/     ← 랜딩 페이지
├── demo/         ← Vite + React 채팅 SPA
├── .flue/        ← 이 저장소가 자기 자신을 운영하는 에이전트
└── .github/      ← CI 워크플로우
```

### 2.1 코어 패키지 6종

| 패키지 | 역할 |
| --- | --- |
| `@flue/runtime` | 심장부. 훅, 세션, 도구, 샌드박스, 영속화 |
| `@flue/vite` | `'use agent'` 스캔 + 빌드 Vite 플러그인 |
| `@flue/cli` | `flue` 바이너리 (run / init / add / docs) |
| `@flue/sdk` | 배포된 에이전트 호출 클라이언트 |
| `@flue/react` | `useAgent()` 리액트 훅 |
| `@flue/opentelemetry` | 관측 / 트레이싱 |

### 2.2 훅 19종 (`packages/runtime/src/hooks/`)

```
use-model              모델 지정
use-tool               도구 장착
use-sandbox            샌드박스 연결
use-skill              스킬(SKILL.md) 로드
use-subagent           서브에이전트 위임
use-mcp-connection     MCP 서버 연결
use-persistent-state   영속 상태
use-instruction        지시문 추가
use-initial-data       초기 데이터
use-agent-start        / use-agent-finish      라이프사이클
use-response-start     / use-response-finish   응답 훅
use-delivery           메시지 전달
use-data-writer        스트리밍 데이터
use-dispatch-message   이벤트 디스패치
```

### 2.3 DB 어댑터 5종
`postgres`, `mysql`, `mongodb`, `redis`, `libsql`

### 2.4 채널(메신저 연동) 17종
```
slack   discord   teams    telegram   whatsapp   messenger
github  linear    notion   intercom   zendesk    stripe
shopify twilio    resend   google-chat  salesforce-marketing-cloud
```
→ 프로바이더 웹훅을 검증해 에이전트 대화로 자동 변환.

### 2.5 예제 (`examples/hello-world/` 15종)

| 파일 | 검증 내용 |
| --- | --- |
| `with-thinking.ts` | 추론 강도(`thinkingLevel`) 조절 |
| `with-sandbox.ts` | Daytona 원격 샌드박스 쉘 실행 |
| `compaction-test.ts` | 컨텍스트 초과 시 자동 요약 |
| `with-abort.ts` | 타임아웃 / 취소 처리 |
| `fs-test.ts` | 가상 파일시스템 읽기·쓰기 |
| `with-registered-provider.ts` | **Ollama 로컬 모델** 연결 |
| `with-skill.ts` | 스킬 로드 및 구조화 결과 |
| `local-env-smoke.ts` | `local()` 환경변수 화이트리스트 검증 |

### 2.6 블루프린트 44개

`flue add <kind> <name>` 을 실행하면 "설치 방법이 적힌 마크다운"을 반환합니다.
사람이 읽어도 되고 **코딩 에이전트에게 그대로 먹여도** 되는 가이드입니다.

- `sandbox--*` (11): e2b, daytona, modal, vercel, cloudflare, boxd, islo, mirage, exedev, cloudflare-computer
- `database--*` (9): postgres, supabase, turso, valkey, mysql, mongodb, redis, libsql
- `channel--*` (17)
- `tooling--*` (3): braintrust, sentry, vitest-evals

### 2.7 `.flue/agents/pr-redirect.ts` — 보안 설계의 교과서

외부 PR을 자동 분석해 버그면 Issue, 기능이면 Discussion으로 변환하고 PR을 닫는
**실제 프로덕션 에이전트**입니다. 핵심은 **권한 분리(Privilege Separation)** 입니다.

```
       LLM (외부 입력을 읽음 = 속을 수 있음)
              │  읽기 전용 GITHUB_TOKEN 만 보유
              ▼
      [스키마로 검증된 판단 결과]
              │
              ▼
   순수 TypeScript 코드 (절대 안 속음)
              │  쓰기 권한 FREDKBOT_GITHUB_TOKEN 보유
              ▼
       실제 이슈 생성 / PR 닫기
```

```ts
useSandbox(local({ env: { GH_TOKEN: ghToken } }));
// FREDKBOT_GITHUB_TOKEN(쓰기 권한)은 샌드박스 env 허용목록 밖에 있음
```

소스 주석 원문: *"프롬프트 인젝션이 완전히 성공해도 봇이 아무것도 쓸 수 없다."*

---

## 3. 핵심 개념 쉽게 이해하기

### 3.1 비유: "AI 직원 관리 시스템"

| Flue 코드 | 비유 | 설명 |
| --- | --- | --- |
| `export function MyAgent()` | 직원 1명 | 함수 하나 = AI 직원 하나 |
| `return "~~해줘"` | 업무 지시서 | 시스템 프롬프트 |
| `useModel(...)` | 두뇌 선택 | 똑똑한 모델 / 저렴한 모델 |
| `useTool({...})` | 업무 권한 | DB 조회, 메일 발송 등 |
| `useSkill(...)` | 업무 매뉴얼 | "환불 처리는 이렇게" |
| `useSandbox(...)` | 개인 작업실 | 코드 실행 환경 |
| `useSubagent(...)` | 부하 직원 | 전문 분야 위임 |
| `usePersistentState()` | 업무 수첩 | 잊지 않게 기록 |
| Durability | 출근 기억 | 서버 재시작해도 이어서 작업 |

### 3.2 useModel 규칙

- **반드시 1회** 호출 (0회 = 에러, 2회 = 에러)
- 형식: `'provider-id/model-id'`
- 선언일 뿐이며, **API 키를 코드에 넣지 않습니다.** 런타임이 연결·인증·재시도를 담당.

```ts
useModel('anthropic/claude-opus-4-6', {
  thinkingLevel: 'high',                  // off | minimal | low | medium(기본) | high | xhigh | max
  compaction: { keepRecentTokens: 16000 },
});
```

### 3.3 영속성 (Durability)

```
손님: "라떼 3잔이요"
   ↓ [대화 로그에 기록]
결제 처리 중... 서버 재시작!
   ↓
재기동 후 로그를 읽고 "여기까지 했었네" → 이어서 진행
```

관련 파일:
- `conversation-writer.ts` — 모든 이벤트를 로그에 기록
- `conversation-reducer.ts` — 로그를 재생해 현재 상태 복원
- `conversation-fold-checkpoint.ts` — 중간 체크포인트

게임의 세이브/로드와 동일한 개념(이벤트 소싱).

### 3.4 Compaction (컨텍스트 압축)

```
[1~80번 대화] ──요약──> "손님이 라떼 좋아하고 알러지 있음"
[81~100번 대화] ────────> 원문 유지
```

`packages/runtime/src/compaction.ts` 가 담당하며,
`examples/hello-world/src/agents/compaction-test.ts` 가 동작을 검증합니다.

### 3.5 샌드박스 안전도

| 종류 | 설명 | 안전도 |
| --- | --- | --- |
| `bash(new Bash({ fs: new InMemoryFs() }))` | 메모리 안의 가상 리눅스 (디스크 없음) | 최상 |
| `local()` | 실제 호스트, 단 환경변수는 화이트리스트만 | 보통 |
| `daytona()` / `e2b()` / `modal()` | 클라우드 격리 컨테이너 | 높음 |

> 참고: 인터넷 허용 옵션 이름이 `dangerouslyAllowFullInternetAccess` 입니다.
> 위험한 기능은 이름부터 위험하게 지어둔 좋은 설계입니다.

---

## 4. 설치 및 사용법

### 사전 준비물

- **Node.js >= 22.19.0** (필수)
- pnpm >= 11 (모노레포 개발 시)
- 모델 API 키 1개 (또는 Ollama 같은 로컬 모델)

### 방법 A — AI 에이전트에게 맡기기 (공식 권장)

코딩 에이전트에 아래 프롬프트를 붙여넣습니다.

```
Read https://flueframework.com/start.md then help create my first agent...
```

### 방법 B — 자동 스캐폴딩

```bash
npx flue init my-agent
cd my-agent
```

### 방법 C — 수동 설치

```bash
npm install @flue/runtime @flue/cli
```

```ts
// flue.config.ts
import { defineConfig } from '@flue/runtime/config';
export default defineConfig({ target: 'node' }); // 또는 'cloudflare'
```

```
# .env
ANTHROPIC_API_KEY="sk-ant-..."
```

```ts
// src/agents/assistant.ts
'use agent';
import { useModel } from '@flue/runtime';

export function Assistant() {
  useModel('anthropic/claude-haiku-4-5');
  return '너는 친절한 도우미야. 짧게 대답해.';
}
```

```bash
# 서버 없이 바로 실행
npx flue run src/agents/assistant.ts --message "다섯 단어로 인사해줘"

# 대화 이어가기 (앞 대화를 기억함)
npx flue run src/agents/assistant.ts --id hello-1 -m "게 이름 뭐가 좋을까?"
npx flue run src/agents/assistant.ts --id hello-1 -m "3개만 더"
```

### 서버로 띄우기

```bash
npm install @flue/vite hono vite
```

```ts
// vite.config.ts
import { flue } from '@flue/vite';
import { defineConfig } from 'vite';
export default defineConfig({ plugins: [flue()] });
```

```ts
// src/app.ts  ← 이 파일명은 고정
import { createAgentRouter } from '@flue/runtime/routing';
import { Hono } from 'hono';
import { Assistant } from './agents/assistant.ts';

const app = new Hono();
app.route('/agents/assistant', createAgentRouter(Assistant));
export default app;
```

```bash
npx vite dev     # http://localhost:5173
npx vite build   # dist/server.mjs (Node) 또는 Cloudflare Worker
```

```bash
curl -X POST http://localhost:5173/agents/assistant/hello-1 \
  -H 'content-type: application/json' \
  -d '{"kind":"user","body":"농담 하나 해줘"}'

curl "http://localhost:5173/agents/assistant/hello-1?view=history"
```

### CLI 명령어

| 명령어 | 용도 |
| --- | --- |
| `flue init [dir]` | 프로젝트 생성 |
| `flue run <path> -m "..."` | 서버 없이 에이전트 1회 실행 |
| `flue add [kind] [name]` | 블루프린트(설치 가이드) 받기 |
| `flue update <kind> <name>` | 기존 연동 업데이트 가이드 |
| `flue docs [read\|search]` | 오프라인 문서 조회 / 검색 |

`flue run` 주요 플래그

| 플래그 | 설명 |
| --- | --- |
| `--id <id>` | 대화 이어가기 (기본값: 새 ULID) |
| `--name <agent>` | 모듈에 에이전트가 여럿일 때 선택 |
| `--json` | JSON 결과 엔벨로프 출력 (CI 친화) |
| `--new` | 이미 존재하면 실패 (중복 실행 방지) |
| `--data '<json>'` | 생성 시 초기 데이터 (`useInitialData`) |
| `--env <path>` | 대체 `.env` 파일 |

종료 코드: `0` 완료 / `1` 실패·설정 오류 / `130` 중단

> 저장소: DB 설정이 없으면 `node_modules/.cache/flue/run.db` 에 자동 저장됩니다.

---

## 5. 자주 묻는 질문 7가지

### Q1. 플러그인? 스킬? MCP?

**셋 다 아니고 "프레임워크"입니다.**

| | 정체 | 비유 |
| --- | --- | --- |
| 플러그인 | 기존 앱에 끼우는 확장 | 콘센트에 꽂는 것 |
| 스킬 | AI에게 주는 매뉴얼 문서 | 업무 매뉴얼 |
| MCP | AI ↔ 도구 연결 **규격** | USB 규격 |
| **Flue** | **앱 자체를 만드는 뼈대** | **건물 골조** |

다만 Flue는 저 셋을 **품고 있습니다**.

- `@flue/vite` 의 `flue()` → Vite **플러그인**은 맞음 (빌드용 부품)
- `useSkill()` → agentskills.io 표준 SKILL.md **소비자**
- `useMcpConnection()` → MCP **클라이언트**

### Q2. API 토큰이 필요한가?

| 경우 | 키 필요 여부 |
| --- | --- |
| 클라우드 모델 (anthropic, openai, google 등) | 필요 |
| **Cloudflare Workers AI** (`cloudflare/...`) | **불필요** |
| **로컬 모델 (Ollama)** | **불필요 (비용 0)** |
| 외부 서비스 연동 (GitHub, Slack, Daytona) | 각 서비스 토큰 필요 |

지원 프로바이더(Pi 기반): `anthropic`, `openai`, `google`, `amazon-bedrock`,
`google-vertex`, `groq`, `mistral`, `xai`, `deepseek`, `cerebras`, `together`,
`fireworks`, `openrouter` 등

빌드 크기 최적화:
```ts
flue({ providers: ['anthropic', 'openai'] });  // 명시한 것만 번들에 포함
```

Ollama 연결 예시 (`examples/hello-world/src/agents/with-registered-provider.ts`):
```ts
setProvider(createProvider({
  id: 'ollama',
  auth: { apiKey: { name: 'Ollama (keyless)', resolve: async () => ({ auth: {} }) } },
  models: [{
    id: 'llama3.1:8b',
    baseUrl: 'http://localhost:11434/v1',
    cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
    // ...
  }],
}));
```

### Q3. 왜 GitHub에서 유명한가?

현재 **약 8.2k 스타, 전체 랭킹 #6908** 수준입니다.

1. **Astro 팀 브랜드** — 이미 쌓인 신뢰 자산
2. **"Not another SDK"** — AI 프레임워크 피로감을 정확히 공략한 카피
3. **React 훅 API** — 기존 프론트엔드 지식을 그대로 재활용, 진입장벽 파괴
4. **완성도** — 채널 17 / 샌드박스 11 / DB 9 / 예제 28 / 블루프린트 44, v2.0.8로 성숙한 상태 공개
5. **타이밍** — Claude Code·OpenClaw 같은 자율 에이전트 붐에 정확히 진입
6. **Dogfooding** — `.flue/agents/pr-redirect.ts` 로 저장소가 스스로를 운영
7. **Apache-2.0 + 개방성** — MCP, Durable Streams 등 열린 표준만 채택
8. **역발상 기여 정책** — *"Pull requests are not accepted"* 가 그 자체로 화제

### Q4. 로컬 에이전트 구축에 도움이 되는가?

**매우 적합합니다.**

- `flue run` — transport-free 로컬 실행 (HTTP 서버·빌드 산출물 불필요)
- `node_modules/.cache/flue/run.db` — 설정 0으로 영속성 확보
- Ollama + InMemoryFs + 로컬 SQLite → **인터넷 0, 비용 0, 유출 0**
- `flue docs search` — 오프라인 문서 검색

만들 수 있는 것 예시

| 아이디어 | 재료 |
| --- | --- |
| 내 파일 정리봇 | `local()` 샌드박스 + 파일 도구 |
| 개발 일지 자동 작성 | git log 도구 + 스케줄 |
| 사내 문서 검색 에이전트 | `useSkill` + 로컬 벡터DB 도구 |
| 테스트 자동 수정봇 | 샌드박스에서 테스트 실행·수정 |

주의사항
- Node 22.19+ 필수
- 8B급 로컬 모델은 **도구 호출 정확도가 낮음** → 복잡한 에이전트는 클라우드 모델 권장
- `local()` 은 실제 호스트이므로 환경변수 화이트리스트 확인 필수

### Q5. React로 만들 수 있는가?

**프론트엔드는 YES, 에이전트 로직은 NO.**

```
┌──────────────────┐   HTTP  ┌─────────────────┐
│   React 앱        │ ──────> │  Flue 서버       │
│  @flue/react      │   SSE   │  @flue/runtime  │
│  useAgent()       │ <────── │                 │
└──────────────────┘         └─────────────────┘
```

- `packages/react/src/use-agent.ts` 에 `useAgent()` 훅이 이미 존재
- `demo/` 폴더에 Vite + React + shadcn/ui 채팅 SPA 완성품 포함
  (`chat-view.tsx`, `composer.tsx`, `message-list.tsx`, `message-item.tsx`, `markdown.tsx`)
- 브라우저에 API 키를 둘 수 없으므로 에이전트는 반드시 서버에 위치

### Q6. PHP로 만들 수 있는가?

**직접 구현은 불가, 연동은 가능합니다.**

불가한 이유: `'use agent'` 지시어를 Vite 플러그인이 파싱하고, 훅 시스템이 JS 실행
컨텍스트에 의존하며, 런타임이 Node 22+ / Cloudflare Workers 전용이기 때문입니다.

하이브리드 구성 (권장)

```php
<?php
$ch = curl_init('http://localhost:5173/agents/assistant/user-' . $userId);
curl_setopt_array($ch, [
  CURLOPT_POST => true,
  CURLOPT_HTTPHEADER => ['Content-Type: application/json'],
  CURLOPT_POSTFIELDS => json_encode(['kind' => 'user', 'body' => $message]),
  CURLOPT_RETURNTRANSFER => true,
]);
$response = curl_exec($ch);  // 202 Accepted

$history = file_get_contents(
  "http://localhost:5173/agents/assistant/user-{$userId}?view=history"
);
```

대안
- PHP에서 `shell_exec("npx flue run ... --json")` 으로 CLI만 사용
- PHP 네이티브 도구(LLPhant, Neuron AI) 사용 — 단 Flue급 기능은 없음

### Q7. 주의할 점

- **PR을 받지 않습니다.** 기여는 Issue / Discussion 으로만.
- `AGENTS.md` 에 *"No tests exist in the repo"* 라고 적혀 있으나 실제로는
  `packages/runtime/src/*.test.ts` 등 테스트 파일이 존재합니다. 문서가 다소 뒤처진 상태.
- v2.0.8이지만 아직 젊은 프로젝트로, API 변경 가능성이 있습니다.
- Node >= 22, pnpm >= 11 <12 요구.

---

## 6. 수익화 아이디어 12선

> 전제: **Apache-2.0** 라이선스이므로 상업적 이용이 자유롭습니다.

### TIER S — 진입 쉽고 수익성 좋음

#### 1. 업종 특화 슬랙 AI 비서 SaaS
채널 패키지가 이미 구현되어 있어 로직만 작성하면 됩니다.

| 항목 | 내용 |
| --- | --- |
| 타깃 | 스타트업 개발팀 / 마케팅팀 / 고객지원팀 |
| 가격 | 팀당 월 49,000 ~ 199,000원 |
| 난이도 | ★★☆☆☆ |
| MVP | 2~4주 |

상품 예시: 온보딩 도우미, 회의록 → 할 일 티켓 자동 생성, 배포 감시봇
재료: `packages/slack` + `useSkill` + `usePersistentState`

#### 2. 쇼핑몰 CS 자동화 (한국 시장)
Shopify, Stripe, Zendesk, Intercom 채널이 모두 준비되어 있습니다.

```ts
'use agent';
export function CsAgent() {
  useModel('anthropic/claude-haiku-4-5');  // 저렴한 모델로 마진 확보
  useSkill(환불정책);
  useTool(주문조회);
  useTool(배송추적);
  useTool(환불처리);
  usePersistentState('고객이력', {});
  return '너는 우리 쇼핑몰 CS 담당이야. 친절하고 정확하게.';
}
```

| 항목 | 내용 |
| --- | --- |
| 가격 | 문의 건당 100~300원 또는 월 99,000원~ |
| 차별점 | 카페24 / 네이버 스마트스토어 특화 (경쟁자 적음) |
| 난이도 | ★★★☆☆ |

참고 예제: `examples/support-desk`, `examples/zendesk-channel`

#### 3. 코드 리뷰 / PR 트리아지 봇
`.flue/agents/pr-redirect.ts` 가 사실상 상용 제품 수준의 베이스 코드입니다.

| 항목 | 내용 |
| --- | --- |
| 기능 | PR 자동 리뷰, 이슈 중복 탐지, 컨벤션 검사, 릴리스 노트 생성 |
| 가격 | 레포당 월 $10~50 (GitHub Marketplace) |
| 난이도 | ★★☆☆☆ |
| 시장 | 글로벌 (한국어 불필요) |

### TIER A — 기술력이 있다면 큰 시장

#### 4. Flue 호스팅 플랫폼 ("Vercel for Agents")
Flue는 프레임워크만 제공하고 호스팅은 하지 않습니다.

```
git push → 자동 빌드 → 배포 → 대시보드
  (대화 로그 / 토큰 비용 / 에러 추적 / 키 관리)
```

| 항목 | 내용 |
| --- | --- |
| 가격 | Free / Pro $20 / Team $99 |
| 난이도 | ★★★★★ |
| 규모 | 잠재적으로 가장 큼 |
| 리스크 | Astro 팀이 직접 출시할 가능성 |

#### 5. 샌드박스 인프라 제공
`blueprints/sandbox--*.md` 에 이미 11개 업체가 연동되어 있다는 것은 시장이 존재한다는 증거입니다.
한국 리전 특화 / GPU 샌드박스 / 초당 과금. 난이도 ★★★★★

#### 6. 에이전트 관측·평가 SaaS
`examples/braintrust`, `examples/vitest-evals`, `examples/sentry` 의 존재가 수요를 증명합니다.
`@flue/opentelemetry` 위에 SaaS를 얹는 구조. 월 $50~500, 난이도 ★★★★☆

### TIER B — 빠른 현금화

#### 7. 유료 템플릿 / 스타터킷
- 쇼핑몰 CS 스타터킷 $49 / 슬랙봇 프로덕션 템플릿 $99 / 풀스택 보일러플레이트 $149
- Gumroad, Lemon Squeezy, CodeCanyon
- 난이도 ★★☆☆☆, 재고 없음

#### 8. 교육 콘텐츠 (한국어 선점)
현재 한국어 Flue 자료가 거의 없습니다.
유튜브 / 인프런 강의(99,000원) / 전자책(29,000원) / 블로그 → 컨설팅 리드
난이도 ★★☆☆☆

#### 9. 스킬 · 블루프린트 마켓플레이스
Flue 스킬은 agentskills.io 표준을 따르므로 호환 스킬 팩 판매가 가능합니다.
세무/회계 5만원, 의료 상담 가이드라인 15만원, 법무 계약 검토 20만원
→ 코드가 아니라 **전문 지식**을 판매. 난이도 ★★☆☆☆

#### 10. 컨설팅 / SI
중소기업 에이전트 구축 건당 500만~3,000만원, 레거시(PHP/Java) 연동 1,000만원~,
유지보수 월 100만원~. 난이도 ★★★☆☆

### TIER K — 한국 시장 특화 (블루오션)

#### 11. 카카오톡 채널 어댑터 ★ 가장 추천
채널 17종 중 **카카오톡이 없습니다.** 한국 메신저 점유율을 고려하면 명백한 공백입니다.

```ts
// @mypackage/flue-kakao
import { kakaoChannel } from '@mypackage/flue-kakao';
app.route('/channels/kakao', kakaoChannel.route());
```

수익 모델
1. 오픈소스 공개 → 명성 확보 + 컨설팅 유입
2. 유료 상업 라이선스 ($199)
3. 이를 기반으로 "카톡 AI 상담봇 SaaS" 런칭

추가 확장: 네이버웍스, 라인, 토스페이먼츠, 알리고(문자)
난이도 ★★★☆☆, 경쟁자 사실상 없음

#### 12. 한국형 인프라 어댑터
네이버 클라우드 / 카카오 i 클라우드 배포 어댑터,
개인정보보호법 대응 감사 로그 패키지. 기업 라이선스 500만원~

### 전략 매트릭스

```
수익규모
   ↑
크 │  #4 호스팅      #5 샌드박스
   │  #6 관측SaaS
   │           #11 카톡어댑터 (추천)
중 │   #1 슬랙봇  #2 쇼핑몰CS
   │   #3 코드리뷰봇
   │       #10 컨설팅
소 │  #7 템플릿  #8 교육  #9 스킬팩
   └────────────────────────────→
      쉬움            어려움      난이도
```

---

## 7. 추천 로드맵

```
1개월차   교육 콘텐츠 시작 (#8)
          → 학습 + 명성 + 초기 수익 동시 확보
              ↓
2~3개월차 카카오톡 채널 어댑터 오픈소스 공개 (#11)
          → "한국 Flue 전문가" 포지션 선점
              ↓
4~6개월차 슬랙봇(#1) 또는 쇼핑몰 CS(#2) SaaS MVP 런칭
          → 앞서 쌓은 신뢰로 초기 고객 확보
              ↓
1년차     호스팅 플랫폼(#4) 또는 컨설팅(#10)으로 확장
```

**핵심 전략: "한국어 × Flue" 교집합을 지금 선점하기.**

---

## 부록: 빠른 참조 치트시트

```bash
# 프로젝트 생성
npx flue init my-agent

# 로컬 1회 실행
npx flue run src/agents/assistant.ts -m "안녕"

# 대화 이어가기
npx flue run src/agents/assistant.ts --id chat-1 -m "계속해줘"

# CI용 JSON 출력
npx flue run src/agents/triage.ts -m "분석해줘" --json | jq -r .message

# 블루프린트 목록
npx flue add

# 오프라인 문서 검색
npx flue docs search "sandbox"

# 개발 서버 / 빌드
npx vite dev
npx vite build
```

```ts
// 훅 빠른 참조
useModel('anthropic/claude-sonnet-4-6', { thinkingLevel: 'medium' }); // 필수, 1회만
useTool({ name, description, input, async run({ data }) { /* ... */ } });
useSkill(skillMarkdown);
useSandbox(local());                       // 또는 bash(...), daytona(...)
useSubagent({ name, description, agent });
useMcpConnection(connection);
const [state, setState] = usePersistentState('key', initial);
useInstruction('추가 지시문');
```

---

*이 문서는 Claude Code 세션에서 `bmshin94/flue` 저장소를 분석하며 작성되었습니다.*
