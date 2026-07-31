# b.stage 커스텀 페이지 온보딩 (SDK_FULL) — 에이전트 실행 runbook

> 이 문서는 **AI 코딩 에이전트(Claude Code 등)가 읽고 그대로 수행**하는 실행 지시서다.
> 사람용 설명문이 아니라 상태 기계다. 각 단계를 **감지 → 검증 → 실행 → 재검증** 순으로 처리하고,
> 실패하면 **추측하지 말고 원문 오류를 그대로 보고한 뒤 멈춘다.**
> 사람이 처리해야만 하는 관문(자격증명 발급·조직 승인·내부 콘솔)은 **WAITING**으로 표시돼 있다.
> WAITING을 만나면 자동으로 넘기려 하지 말고, "무엇을 · 어디서 · 왜 받아야 하는지 + 받은 뒤 재개하는 방법"을 사용자에게 안내하고 멈춘다.
>
> ⚠️ **이 문서를 요약해서 읽지 마라.** 명령·코드블록이 그대로 있어야 수행할 수 있다. 요약 프록시(일부 WebFetch)가 코드블록을 접었다면 raw 원문을 다시 확보하라:
> `curl -s https://raw.githubusercontent.com/cheese-bmf/onboarding-test/main/ONBOARDING.md`

## 이 온보딩이 만드는 것

b.stage SaaS의 특정 URL을 **커스텀 풀페이지(SDK_FULL)** 로 갈아 끼우는 3rd-party 템플릿 프로젝트. React 컴포넌트를 Web Component(IIFE 번들)로 빌드해 b.stage에 배포한다.

이 runbook의 도달 지점을 정직하게 밝힌다:
- **로컬까지(Phase 0~1)**: b.stage **계정·스테이지·조직·콘솔은 필요 없다.** 단 하나, SDK 설치용 **`PARTNERS_TOKEN`(사람이 발급하는 GitHub PAT)** 은 필요하다 — 이게 Phase 1의 유일한 사람 관문(G1)이고, 토큰만 있으면 스캐폴드·빌드·로컬 렌더까지 자동으로 간다.
- **배포까지(Phase 2)**: b.stage API 키·조직 접근·Console 활성화가 필요하며, 각각 명시적 WAITING으로 멈춘다.

## 단계 표기 규약

각 단계는 다음을 판정한다.

- `status`: `ok` | `failed` | `waiting`
- `next_action`: 다음에 할 일
- `requires_human`: 사람 개입이 필요한가 (WAITING 단계는 항상 yes)
- 실패 시: 원문 오류 + 어디서 멈췄는지 보고. 반쯤 된 상태로 다음 단계를 실행하지 않는다.

핵심 설계 원칙(따를 것):
1. **생성 먼저, 연결은 나중에.** 로컬 스캐폴드·빌드를 먼저 성공시키고, b.stage 관문(토큰·조직·콘솔)은 뒤로 몬다.
2. **명령을 지어내지 않는다.** 이 문서에 없는 플래그·명령이 필요하면 `--help`로 실제 CLI를 확인하고, 확인 안 되면 멈춰서 사용자에게 묻는다.
3. **성공은 "명령이 끝났다"가 아니라 "검증이 통과했다"로 정의한다.**

---

## Phase 0 — doctor (환경 진단 · 인증 불필요)

사람 개입 없이 항상 실행 가능. 로컬 문제와 b.stage 계정 문제를 분리하기 위해 먼저 돌린다.

**검증**
```bash
node -v      # v20 이상이어야 함
npm -v       # (또는 pnpm -v)
git --version
```

- Node가 20 미만 → `status: failed`. 사용자에게 Node 20+ 설치를 안내하고 멈춘다.
- 위가 모두 되면 `status: ok`.

**사전 준비물 체크 (Phase 1 진입 전):** Phase 1은 계정·스테이지 없이 되지만 **`PARTNERS_TOKEN` 하나는 반드시 필요**하다. 지금 없으면 아래 G1이 곧 WAITING이 되니, 먼저 발급 요청을 걸어두는 걸 권한다.

---

## Phase 1 — init (로컬 스캐폴드 + 로컬 렌더까지)

목표: b.stage **계정·스테이지 없이**(단, 아래 G1의 `PARTNERS_TOKEN`은 필요) 프로젝트를 만들고 로컬 개발 서버에 첫 화면을 띄운다.

### ⛔ WAITING G1 — PARTNERS_TOKEN (classic PAT)

SDK 패키지(`@partners-bmf/*`)는 npmjs가 아니라 **GitHub Packages(`https://npm.pkg.github.com`) 사설 레지스트리**에 있다. 그래서 **CLI를 가져오는 `npx @partners-bmf/bstage-cli` 자체가 인증·레지스트리 설정 전에는 실패**한다(npm이 기본 레지스트리 npmjs에서 찾다 404/401). 이 토큰은 **사람이 발급받아야 한다(자동화 불가).**

- **무엇을**: `PARTNERS_TOKEN` = GitHub **classic** PAT, scope 최소 `read:packages`
  - ⚠️ **Fine-grained PAT는 GitHub Packages를 아직 지원 안 함 → 반드시 classic 토큰**
  - 생성 후 토큰 목록에서 **Configure SSO → `partners-bmf` 조직 Authorize**까지 눌러야 함. 안 하면 private 패키지가 404/401
- **어디서**: GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic). 발급 경로 문의는 b.stage 파트너 담당자
- **왜**: 없으면 CLI를 받는 `npx`도, 프로젝트 의존성 설치도 GitHub Packages 401/404로 실패
- **재개 방법**: 토큰을 받은 뒤 `export PARTNERS_TOKEN=ghp_xxxx` 하고, 아래 "레지스트리 설정" 단계부터 진행

> 존재 여부만 안전하게 확인한다(값을 출력하거나 로그에 남기지 말 것):
> ```bash
> [ -n "$PARTNERS_TOKEN" ] && echo "PARTNERS_TOKEN: set" || echo "PARTNERS_TOKEN: missing -> WAITING"
> ```
> `missing`이면 `status: waiting`으로 멈춘다.
> (이 관문 + 아래 레지스트리 수동 설정은 TO-BE 개선 대상이다 — TOBE.md G1 참조.)

### 실행 — 레지스트리 설정 (`.npmrc`) · `npx` 전에 반드시

`@partners-bmf` 스코프를 GitHub Packages로 매핑해야 CLI를 가져올 수 있다. **프로젝트를 만들 상위 디렉터리(지금 `npx`를 실행할 곳)에 `.npmrc`를 먼저 만든다.**

```bash
cat > .npmrc <<'NPMRC'
@partners-bmf:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${PARTNERS_TOKEN}
//npm.pkg.github.com/:always-auth=true
NPMRC
```
- `${PARTNERS_TOKEN}`은 npm이 실행 시 환경변수에서 읽는다. **토큰 원문을 `.npmrc`에 직접 쓰지 말 것.**
- 어드민 계보(`@frontend-bmf`)까지 쓸 거면 같은 방식으로 `@frontend-bmf:registry=...` 한 줄을 추가. SDK_FULL 유저 템플릿만이면 위 세 줄로 충분.
- **검증**: `npm view @partners-bmf/bstage-cli version` 이 401/404 없이 버전을 출력하면 통과. 실패하면 G1(토큰·SSO·classic 여부)로 돌아간다.

### 실행 — 프로젝트 생성
```bash
npx @partners-bmf/bstage-cli@latest init --yes --space <space>
# <space>: 프로젝트 식별자. 대상 스테이지가 정해지지 않았으면 임시로 demo 사용
```
- 인터랙티브로 물으면(비TTY 실패 시) `--yes --space <space>`로 비대화형 실행. API 키는 **"없어도 시작"** 을 선택(나중 connect 단계에서 설정).
- 완료되면 새 폴더에 의존성 설치·git 초기화·보일러플레이트가 생성된다.
  > ⚠️ **폴더명을 넘겨짚지 말 것.** SDK 문서 버전에 따라 `bstage-<space>-templates/` 또는 `<space>-custom-templates-<phase>/`로 다르다. init 출력에 찍힌 실제 폴더명을 확인하거나 `ls -d */`로 새로 생긴 디렉터리를 찾아 그리로 `cd` 한다.

### 실행 — 템플릿 작성 (SDK_FULL 풀페이지)
`src/templates/<name>/template.tsx` 를 만든다. **파일명은 반드시 `template.tsx`**.
```tsx
import { createTemplate } from '@partners-bmf/bstage-react'

export default function DemoTemplate() {
  return <div style={{ padding: 24 }}><h1>Hello, b.stage!</h1></div>
}

createTemplate(DemoTemplate, {
  space: '<space>',
  name: '<name>',
  // target 미지정 → 기본 'user'(유저 화면), slot 미지정 → 풀페이지(SDK_FULL)
})
```

### 검증 — 로컬 성공 신호
```bash
cd <init이 생성한 폴더>   # 위에서 확인한 실제 폴더명
npm run dev
```
- `http://localhost:5173` → 템플릿 목록, `http://localhost:5173/<name>` → 컴포넌트 렌더 확인.
- 여기까지 뜨면 **Phase 1 `status: ok`** (b.stage 계정 없이 도달하는 첫 성공 지점).

### Phase 1 자가진단 (렌더가 안 되거나 빌드가 깨질 때 먼저 확인)
- [ ] `createTemplate()` 이 **모듈 최상위**에서 호출되는가 (함수 내부 호출 금지 — 빌드 시 메타데이터 추출 실패)
- [ ] 컴포넌트를 **`export default`** 로 내보냈는가
- [ ] 파일 경로가 정확히 `src/templates/<name>/template.tsx` 인가
- [ ] CSS는 `?inline` 로 import해서 `<style>` 로 주입했는가 (Shadow DOM이라 외부 CSS 미적용)
- [ ] `.npmrc`·`PARTNERS_TOKEN` 설정이 맞는가 (설치 401/403이면 G1 재확인)

---

## Phase 2 — connect (b.stage 관문 · 배포)

여기서부터는 **b.stage 계정·조직·콘솔**이 필요하다. 대상 스테이지가 없으면 대부분 WAITING이며, 그 자체가 정상이다(로컬까지는 이미 성공). 각 관문을 사용자에게 안내하고 멈춘다.

### ⛔ WAITING G2 — API 자격증명 (appId / appSecret)
인증이 필요한 API를 호출하려면 필요.
- **무엇을**: `appId`(`bsa_...`), `appSecret`(`bsp_...`)
- **어디서**: 파트너 콘솔에서 발급
- **왜**: `src/shared/client.ts`의 `BstageClient` 인증에 사용
- **재개**: 발급 후 `client.ts`에 설정. **주의: `bsa_`=appId, `bsp_`=appSecret. 뒤바꿔 넣으면 인증 에러**(대표 지뢰)
- API 호출이 필요 없는 순수 화면 커스텀이면 이 관문은 건너뛸 수 있다.

### ⛔ WAITING G3 — GitHub 조직 접근 (배포 리포)
- **무엇을**: 배포용 GitHub 조직(`partners-bmf` 등) 멤버십
- **어디서**: flex 기안으로 조직 권한 신청 (승인에 시간 걸림 — 제일 먼저 신청)
- **왜**: 배포는 이 조직의 리포에 push하면 CI가 빌드·업로드한다. 권한 없으면 리포 링크가 GitHub 404
- **재개**: 승인 후 리포 클론·push 진행

### ⛔ WAITING G4 — b.stage Console 커스텀 템플릿 활성화
- **무엇을**: b.stage Console에서 해당 커스텀 템플릿/URL **활성화 ON**
- **어디서**: b.stage Console (**bemyfriends 내부 제품 — 외부에서 구조적으로 못 켠다**)
- **왜**: 이걸 안 켜면 **배포가 성공해도 접속 시 전부 404**. 포털 어디에도 "먼저 켜라"는 안내가 없고 켜졌는지 확인할 방법도 없다(대표 지뢰 1번)
- **재개**: 내부 담당자에게 활성화 요청 → 켜진 뒤 배포 접속 확인
- (이 관문은 외부 자동화가 원천 불가라 TO-BE 최우선 대상 — TOBE.md G4)

### 실행 — 배포 (권한이 갖춰진 경우)
빌드·업로드는 CI가 자동 처리한다. 개발자가 직접 빌드할 필요 없음.
```bash
# main push → dev, qa 환경 자동 배포
git push origin main

# real 환경 배포 → 태그 push
git tag v1.0.0 && git push origin v1.0.0
```
수동으로 산출물만 확인하려면 `npx bstage build` → `dist/<target>/<space>/<name>/{template.js, manifest.json}` 생성.

### Phase 2 배포 자가진단 (증상 → 원인)
| 증상 | 원인 / 확인 |
|---|---|
| 배포는 성공인데 접속하면 전부 404 | **Console 활성화 안 켬** (G4). 가장 흔함 |
| 성공 체크마크인데 화면이 안 바뀜 | **업로드 ≠ 릴리즈**. 업로드만 하고 릴리즈(라우팅 적용)를 안 걸었다 (별개 단계) |
| 업로드(빌드) 실패 | SDK 매핑 경로가 `manifest.json`이 실제로 놓이는 디렉토리명과 안 맞음 |
| 재배포 버튼이 비활성 | 노출 중이라서. 미노출로 내려야 열림. (미노출 저장이 반영 안 되는 버그 존재 — 막히면 담당자에게) |
| API 호출이 인증 에러 | `bsa_`/`bsp_`를 뒤바꿔 넣었거나 `client` 연결 한 줄 누락 (G2) |
| 레포 링크 눌렀는데 GitHub 404 | 조직 접근 권한 없음 → flex 기안 (G3) |

---

## 완료 정의

- **로컬 완료(계정·스테이지 없이, `PARTNERS_TOKEN`만 있으면 도달)**: Phase 0 ok + Phase 1 ok (localhost:5173에 컴포넌트 렌더).
- **배포 완료(관문 통과 필요)**: G2~G4 해소 + 배포 접속 시 커스텀 화면 정상 표시 + 자가진단 표에 걸린 항목 없음.

대상 스테이지가 아직 없으면 **로컬 완료까지가 이번 온보딩의 성공 지점**이고, Phase 2는 관문 목록을 사용자에게 남기고 멈추는 것이 올바른 종료다.

## 이 runbook에서 만난 WAITING을 기록하자

각 WAITING(G1~G4)에서 실제로 어디서·왜 막혔는지 한 줄씩 남기면, 그게 포털이 없앨 마찰의 근거가 된다. 개선 방향은 [TOBE.md](./TOBE.md)에 정리돼 있다.
