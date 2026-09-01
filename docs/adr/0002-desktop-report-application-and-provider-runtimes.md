# ADR-0002: Desktop Report Application and Provider Runtimes

- 상태: Accepted
- 날짜: 2026-09-01
- 대체 결정: [ADR-0001](0001-v1-runtime-and-boundaries.md)

## Context

ADR-0001은 인증된 Codex CLI와 GitHub Copilot CLI를 비대화형으로 호출하는 CLI-first 생성기를 v1 제품으로 해석했다. 그러나 CLI를 요구한 이유는 터미널 UX를 제공하기 위해서가 아니라 사용자가 이미 보유한 Provider 구독과 인증을 재사용하고, Report-Gen이 별도의 API Key를 취급하지 않게 하기 위해서다.

Report-Gen의 실제 제품 표면은 범용 AI 도구를 어려워하는 사용자를 위한 로컬 데스크톱 보고서 생성기다. 사용자는 보고서 중심 채팅 UI에서 요청과 자료를 제출하고 진행 상태, 근거, 미리보기와 결과를 확인해야 한다. 내부에서는 요청 평가, 정제, 근거 보완, 작성, 검토와 정적 렌더링이 명시적인 워크플로로 실행되어야 한다.

두 Provider는 통합 표면이 같지 않다. Codex는 App Server를 통해 인증, Thread/Turn, 스트리밍, 승인과 Skill 정보를 제공한다. GitHub Copilot은 인증된 CLI의 대화형 또는 programmatic 기능, custom agent, Skill과 Hook을 제공한다. 이 차이를 숨기기 위해 기능을 가장 낮은 공통분모로 축소하거나, Provider의 대화 상태를 애플리케이션 상태로 오인해서는 안 된다.

## Decision

1. Report-Gen v1은 로컬 데스크톱 애플리케이션이다. Node.js 22 이상과 TypeScript strict ESM을 공통 기반으로 사용하고, Electron main process와 격리된 renderer process로 패키징한다. 화면 프레임워크의 구체적인 선택은 이 ADR에서 고정하지 않는다.
2. renderer process는 UI 표시와 사용자 입력만 담당한다. Provider 프로세스, 인증 조정, 워크플로, 파일시스템, 검증, 렌더링과 로컬 저장은 main process 또는 main이 소유한 로컬 모듈에서 수행한다. 양쪽은 좁고 타입이 지정된 IPC 계약으로만 통신한다.
3. 애플리케이션이 Provider-neutral `ReportSession`과 보고서 생성 상태 머신의 원본이다. Provider Thread나 프로세스 세션 ID는 재개를 위한 외부 참조일 뿐 제품 상태의 원본이 아니다.
4. Codex Provider는 `codex app-server`를 로컬 자식 프로세스로 실행하고 기본 stdio JSONL JSON-RPC 전송을 사용한다. App Server의 account, thread, turn, streaming, approval, interruption과 Skill 표면을 어댑터 뒤에서 사용한다. `codex exec`는 대화형 제품 Core가 아니다.
5. GitHub Copilot Provider는 공식 `copilot` CLI의 기존 OAuth 인증과 programmatic 실행 표면을 사용한다. 출력과 진행 이벤트는 공통 애플리케이션 이벤트로 정규화하되, Codex에만 존재하는 세밀한 기능을 Copilot에 있는 것처럼 가장하지 않는다.
6. v1은 Codex와 GitHub Copilot의 기존 인증 또는 공식 사용자 주도 로그인만 지원한다. Report-Gen은 API Key, BYOK, Provider token 또는 환경변수 기반 token 입력을 받거나 저장하지 않는다. Provider credential 파일을 직접 읽지 않으며, 로그인 결과와 계정 상태만 공식 Provider 표면을 통해 관찰한다.
7. 보고서 워크플로는 최소한 `request-evaluation`, `prompt-refinement`, `evidence`, `composition`, `review`, `render` 단계를 구분한다. 각 LLM 단계는 버전이 지정된 입력·출력 Schema를 가지며, 실패·재시도·취소 상태를 명시적으로 기록한다. 역할을 서로 다른 Provider에 배정할 수 있지만, Provider 분리는 계약 검증을 대체하지 않는다.
8. 보고서 작성 지침, 근거 정책, 구조 Schema와 디자인 참조를 앱 소유 Agent package로 관리한다. Codex Skill/Hook과 Copilot agent/Skill/Hook은 같은 제품 정책의 Provider별 projection이다. 암묵적 자동 선택에만 의존하지 않고 가능한 경우 명시적으로 선택한다.
9. Hook은 입력 점검, 도구 제한 보조, 실행 기록, 결과 검증과 정리 같은 guardrail에 사용한다. 워크플로 상태 머신이나 유일한 보안 경계로 사용하지 않는다. 실제 권한은 process boundary, sandbox, approval, 경로 제한과 애플리케이션 검증에서 통제한다.
10. JSON Schema는 Provider-neutral 구조화 계약의 단일 원본이다. Provider 결과는 모두 애플리케이션 Validator를 통과해야 하며, 검증된 `ReportDocument`만 정적 HTML Renderer에 전달한다.
11. Renderer는 `DESIGN.md`를 따르는 self-contained borderless HTML을 결정적으로 생성한다. Provider나 LLM은 HTML, CSS 또는 레이아웃을 생성하거나 수정하지 않는다.
12. 보고서 세션, 단계 결과와 산출물은 로컬에 저장한다. 임시 산출물은 성공 결과와 분리하고, 검증과 렌더링이 모두 끝난 결과만 원자적으로 게시한다. v1 Core에는 원격 백엔드나 Report-Gen 계정이 필요하지 않다.

## Consequences

- 비기술 사용자는 터미널 대신 보고서 작업에 맞춘 채팅, 진행 상태, 근거와 미리보기 UI를 사용한다.
- 사용자는 자신의 Codex 또는 GitHub Copilot 구독과 공식 인증을 사용하며 Report-Gen에 별도 비밀값을 제공하지 않는다.
- 데스크톱 패키징, IPC, renderer 격리, 프로세스 수명주기와 앱 업데이트가 새로운 구현 및 검증 범위가 된다.
- Provider 간 기능 비대칭을 명시적으로 다루는 capability model과 contract test가 필요하다.
- 동일한 워크플로 계약을 여러 Provider customization 형식으로 투영하므로 의미 동기화 검증이 필요하다.
- App Server와 Copilot CLI 버전 변화가 제품 호환성에 영향을 주므로 최소 지원 버전, 진단과 graceful failure가 필요하다.
- 기존 Schema, 정적 Renderer, 결정성, 콘텐츠 안전과 Provider-neutral 경계 결정은 유지된다.

## Alternatives considered

### CLI-first 제품 유지

대상 사용자가 터미널과 범용 Agent 사용법을 알아야 하므로 제품 목적과 맞지 않는다.

### OpenAI 또는 GitHub API Key 직접 사용

Report-Gen이 비밀값 저장, 과금 경로와 키 지원 책임을 갖게 되며 기존 구독 인증을 재사용하려는 요구를 위반하므로 거부했다.

### Codex만 제품 Core로 사용하고 Copilot을 보조 도구로 제한

구현은 단순하지만 v1에서 두 Provider를 지원한다는 제품 계약을 약화하므로 거부했다. 애플리케이션이 Provider-neutral 세션을 소유하고 두 런타임을 어댑터로 통합한다.

### 두 Provider의 모든 기능을 동일한 인터페이스로 강제

가짜 기능, 취약한 에뮬레이션과 최저 공통분모 UX를 만들기 때문에 거부했다. 공통 보고서 동작과 선택적 capability를 분리한다.

### 로컬 웹 서버와 브라우저 UI

가능하지만 로컬 포트, 브라우저 수명주기와 별도 보안 경계를 추가한다. Provider 자식 프로세스와 OS 통합을 소유하는 v1 패키지에는 Electron main/renderer 경계가 더 직접적이다.

## Validation

- OpenAI 공식 App Server 문서에서 product integration, 인증, Thread/Turn, streaming, approval, interruption과 Skill API를 확인했다.
- GitHub 공식 Copilot CLI 문서에서 OAuth 로그인, programmatic 호출, custom agent, Skill과 Hook 지원을 확인했다.
- 실제 구현 전 Architecture spike에서 설치된 지원 버전으로 양쪽 Provider의 인증, 취소, structured output과 프로세스 종료 동작을 고정해야 한다.

## References

- [Codex App Server](https://learn.chatgpt.com/docs/app-server)
- [Codex authentication](https://learn.chatgpt.com/docs/auth)
- [GitHub Copilot CLI authentication](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/authenticate-copilot-cli)
- [GitHub Copilot CLI overview](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/overview)
- [GitHub Copilot CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)
- [GitHub Copilot CLI hooks](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/use-hooks)
