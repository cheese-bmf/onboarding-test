# b.stage 커스텀 페이지 온보딩 (AI 에이전트용)

b.stage SaaS의 한 URL을 **커스텀 풀페이지(SDK_FULL)** 로 갈아 끼우는 프로젝트를, AI 코딩 에이전트가 **복붙 한 번**으로 시작하게 하는 온보딩이다. 로컬에서 돌아가는 상태까지는 자동으로 만들어 주고, b.stage 계정·조직·콘솔이 필요한 지점은 "여기서 이걸 받아오세요"라고 멈춰서 안내한다.

---

## 🚀 AI로 시작하기 (권장)

아래 한 줄을 복사해 Claude Code(또는 URL을 읽을 수 있는 AI 코딩 툴)에 붙여넣으세요.

```
https://raw.githubusercontent.com/cheese-bmf/onboarding-test/main/ONBOARDING.md 를 읽고 그대로 온보딩을 진행해줘. (문서를 요약하지 말고 원문 그대로 읽어 — 명령·코드블록이 그대로 있어야 해. 코드블록이 사라졌으면 `curl`로 raw를 다시 가져와.) 각 단계에서 감지→검증→실행→재검증하고, 실패하면 원문 오류를 그대로 알려줘. 그리고 **끝내지 말고 이어가줘** — 자동으로 되는 단계는 멈추지 말고 쭉 진행하고, 사람이 처리해야 하는 관문에서는 무엇을·어디서·왜 해야 하는지 알려준 뒤 내가 '완료했어'라고 하면 다시 검증하고 다음 단계로 계속 진행해.
```

에이전트가 위 URL의 지시서(runbook)를 읽고 환경 진단 → 스캐폴드 → 로컬 서버 기동까지 알아서 진행하고, b.stage 자격증명·조직 승인·콘솔 활성화가 필요한 지점에서 멈춰 안내합니다.

---

## 🔌 대안: 플러그인으로 설치 (Claude Code)

반복해서 쓰거나 팀에 배포하려면 플러그인으로 설치할 수 있어요. 설치하면 온보딩 관련 요청에 스킬이 자동으로 이 runbook을 따라갑니다.

```
/plugin marketplace add cheese-bmf/onboarding-test
/plugin install bstage-onboarding@bstage-plugins
```

설치 후 세션을 재시작하면 활성화됩니다. 내용이 바뀌면 `/plugin marketplace update` 로 갱신하세요.

> URL 방식과 플러그인 방식은 **같은 지시서([ONBOARDING.md](./ONBOARDING.md))를 가리킵니다.** 명령은 한 곳(ONBOARDING.md)에만 있고 플러그인 스킬은 그걸 읽어 실행하는 얇은 라우터예요 — 버전 어긋남을 막기 위해서입니다.

---

## 무엇을 하고, 어디서 멈추나

- **자동으로 되는 것**: 환경 진단(Node 20+/git), `bstage init` 스캐폴드, 템플릿 작성, `npm run dev` 로컬 렌더까지.
- **사람이 필요한 관문(WAITING)**: PARTNERS_TOKEN 발급 · API 키(appId/appSecret) · GitHub 조직 접근(flex 기안) · b.stage Console 활성화. 이 관문들은 지금 온보딩을 몇 시간씩 잡아먹는 지점이고, 개선 방향은 [TOBE.md](./TOBE.md)에 정리돼 있습니다.

## 파일

| 파일 | 용도 |
|---|---|
| [ONBOARDING.md](./ONBOARDING.md) | **정본 지시서** — 에이전트가 따라가는 runbook (SDK_FULL) |
| [TOBE.md](./TOBE.md) | 멈춤 지점(사람 관문) → TO-BE 개선 목록 |
| `plugins/bstage-onboarding/` | Claude Code 플러그인 (스킬 = ONBOARDING.md 라우터) |

---

_이 repo는 "복붙 한 번으로 온보딩" 방식을 검증하는 파일럿입니다. 대상 스테이지는 아직 없어, 배포(Phase 2)는 관문 안내까지가 현재 범위예요._
