---
name: bstage-onboarding
description: Use when the user wants to start a b.stage custom page/app (SDK_FULL) — scaffolding a b.stage 3rd-party template, setting up @partners-bmf SDK, "b.stage 온보딩", "커스텀 페이지 만들기", "bstage init". Guides the agent through the canonical onboarding runbook: environment doctor → local scaffold/build → b.stage connect gates.
---

# b.stage 온보딩 스킬 (라우터)

이 스킬은 명령을 자체적으로 담지 않는다. **정본 지시서 하나(ONBOARDING.md)를 읽어 그대로 수행**한다. 명령을 두 곳에 중복하면 버전이 어긋나기 때문이다.

## 수행 절차

1. 아래 정본 runbook을 **원문 그대로**(요약 없이) 가져온다. 요약 프록시는 코드블록·명령을 접으므로, 가능하면 `curl`로 raw를 받는다:
   ```bash
   curl -s https://raw.githubusercontent.com/cheese-bmf/onboarding-test/main/ONBOARDING.md
   ```
   (WebFetch만 쓸 수 있고 명령이 요약돼 사라졌으면, 사용자에게 원문을 붙여 달라고 요청한다.)
2. 그 문서의 지시를 **그대로** 따른다:
   - 각 단계를 **감지 → 검증 → 실행 → 재검증** 순으로 처리
   - 실패하면 **추측하지 말고 원문 오류를 보고한 뒤 멈춘다**
   - `WAITING`(사람 관문: PARTNERS_TOKEN·API 키·GitHub 조직·Console 활성화)을 만나면 자동으로 넘기지 말고, **무엇을·어디서·왜 받아야 하는지 + 재개 방법**을 사용자에게 안내하고 멈춘다
3. 문서에 없는 명령·플래그가 필요하면 지어내지 말고 `--help`로 실제 CLI를 확인하거나 사용자에게 묻는다.

## 원칙 (ONBOARDING.md와 동일)

- **생성 먼저, 연결은 나중에** — 로컬 스캐폴드·빌드 성공을 자격증명보다 앞에 둔다.
- **성공은 "명령이 끝났다"가 아니라 "검증이 통과했다"** 로 정의한다.

URL을 가져올 수 없는 환경이면 사용자에게 `ONBOARDING.md` 내용을 붙여 달라고 요청한 뒤 동일하게 수행한다.
