# ADR-0001: v1 Runtime and Architecture Boundaries

- 상태: Superseded by [ADR-0002](0002-desktop-report-application-and-provider-runtimes.md)
- 날짜: 2026-09-01

## Context

Report-Gen은 인증된 Codex CLI와 GitHub Copilot CLI에서 일관된 구조화 보고서를 받아야 한다. Provider 출력 차이가 최종 레이아웃에 누출되면 안 되며, v1 HTML은 정적이고 재현 가능해야 한다. 현재 저장소에는 제품 및 디자인 계약만 있고 런타임과 구현 경계가 정해지지 않았다.

2026-09-01 로컬 환경에서 Node.js 24.18.0, npm 11.14.1, pnpm 10.34.1, Codex CLI 0.150.1과 GitHub Copilot CLI 1.0.78을 확인했다. Codex CLI는 `--output-schema`와 `--output-last-message`를 제공한다. Copilot CLI는 prompt mode와 clean text 출력을 제공하지만 동일한 Schema 강제 옵션은 제공하지 않는다.

## Decision

1. Node.js 22 이상, TypeScript strict ESM과 npm을 v1 기준선으로 사용한다.
2. JSON Schema를 Provider 중립적인 보고서 계약의 단일 원본으로 사용한다.
3. 모든 Provider 응답은 동일한 런타임 Validator를 통과해야 한다.
4. Provider 실행, 응답 검증과 HTML 렌더링을 별도 경계로 분리한다.
5. Renderer는 검증된 데이터만 받아 외부 네트워크나 LLM 호출 없이 self-contained HTML을 생성한다.
6. 두 CLI는 shell 없이 자식 프로세스로 실행하고, 시간·출력·권한을 제한한다.
7. v1에서 A4와 Slide Renderer는 구현하지 않는다.

## Consequences

- 두 Provider는 실행 방식이 달라도 동일한 Schema와 Renderer를 공유한다.
- Copilot 출력은 프롬프트만 신뢰하지 않고 반드시 애플리케이션에서 검증한다.
- Codex의 native Schema 검증을 사용하더라도 공통 Validator를 생략하지 않는다.
- UI 프레임워크나 서버가 필요 없으므로 배포 결과와 공격 표면이 작아진다.
- Node.js와 npm이 추가된 개발 전제 조건이 된다.
- 실제 CLI 버전 변화는 Provider contract test와 지원 범위 문서의 갱신을 요구한다.

## Alternatives considered

### Provider가 직접 HTML 생성

Provider별 레이아웃 차이, escape 누락과 비결정적 결과 때문에 거부했다.

### Provider별 Schema

최종 보고서 계약이 분기되고 Renderer가 Provider를 알아야 하므로 거부했다.

### 브라우저 앱 또는 서버 우선 구현

v1은 정적 HTML 생성이 핵심이며 운영 복잡성만 늘어나므로 보류했다.

### Python runtime

충분히 가능하지만 HTML 및 향후 Slide/A4 생태계, 타입 기반 계약 공유와 현재 로컬 도구 구성을 고려해 Node.js/TypeScript를 선택했다.

### pnpm

현재 설치되어 있으나 사용자가 별도 설치해야 할 전제 조건을 줄이기 위해 Node.js에 포함되는 npm을 선택했다.

## Validation

- 두 CLI의 로컬 `--help`와 버전을 확인했다.
- Codex 인증 상태가 기존 ChatGPT 세션을 사용함을 확인했다.
- 공식 Codex CLI 및 GitHub Copilot CLI 문서에서 비대화형 실행, 출력 캡처와 권한 제한 옵션을 대조했다.

## References

- [OpenAI Codex CLI command reference](https://developers.openai.com/codex/cli/reference/)
- [GitHub Copilot CLI programmatic usage](https://docs.github.com/en/copilot/how-tos/copilot-cli/automate-copilot-cli/run-cli-programmatically)
- [GitHub Copilot CLI command reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)
