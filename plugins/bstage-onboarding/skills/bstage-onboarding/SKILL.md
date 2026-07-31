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
   - 실패하면 **추측하지 말고 원문 오류를 보고한 뒤 CHECKPOINT로 일시정지한다**. 세션을 끝내지 말고 사용자가 처리할 일과 재개 신호를 안내한다.
   - **끝내지 말고 이어간다(체크포인트-재개 루프)**: 자동 단계는 멈추지 말고 쭉 진행하고, 사람 관문(PARTNERS_TOKEN·API 키·GitHub 조직·Console 활성화)이나 실행 실패에서는 무엇을·어디서·왜를 안내한 뒤 사용자의 "완료했어" 신호를 기다렸다가, 신호를 받으면 해당 검증 명령을 다시 돌려 통과 시 다음 단계로 이어간다. "온보딩 중단"으로 세션을 끝내지 않는다.
   - 이미 생성된 프로젝트 디렉터리가 있으면 init을 다시 실행해 덮어쓰지 않는다. init 출력 또는 예상 경로의 파일을 확인하고, 생성된 프로젝트의 검증 단계부터 재개한다. 상태가 불명확하면 디렉터리를 삭제하라고 지시하지 말고 파일 목록과 원문 오류를 CHECKPOINT로 보고한다.
3. 문서에 없는 명령·플래그가 필요하면 지어내지 말고 `--help`로 실제 CLI를 확인하거나 사용자에게 묻는다.

## 원칙 (ONBOARDING.md와 동일)

- **생성 먼저, 연결은 나중에** — 로컬 스캐폴드·빌드 성공을 자격증명보다 앞에 둔다.
- **성공은 "명령이 끝났다"가 아니라 "검증이 통과했다"** 로 정의한다.

URL을 가져올 수 없는 환경이면 사용자에게 `ONBOARDING.md` 내용을 붙여 달라고 요청한 뒤 동일하게 수행한다.
