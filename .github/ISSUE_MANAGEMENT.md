# Issue 및 Project 관리 규칙

이 저장소의 아이디어, 개발 계획과 버그는 GitHub Issues 및
[`Report-Gen` Project](https://github.com/users/SWBaek/projects/4)에서 관리한다.

## 등록과 분류

- 새 작업은 목적에 맞는 Issue Form으로 등록한다.
- Issue Form으로 만든 Issue는 `Report-Gen` Project에 자동 등록된다.
- 새 Issue에는 `needs-triage` 라벨을 붙이고 `Todo`에서 범위, 우선순위와 완료 조건을 확인한다.
- 분류가 끝나면 `needs-triage` 라벨을 제거하고 `Priority`, `Area`, `Size`를 지정한다.
- 한 Issue는 독립적으로 검증할 수 있는 하나의 결과를 다룬다.
- 진행하지 않기로 한 제안은 삭제하지 않고 사유를 남긴 뒤 `not planned`로 닫는다.

## Project 필드

- `Status`: Todo, In Progress, Done
- `Priority`: P0, P1, P2, P3
- `Area`: Provider, Schema, Renderer, Design, Tooling, Docs
- `Size`: S, M, L
- `Target date`: 일정이 필요한 작업에만 지정

진행 상태는 라벨과 중복 관리하지 않고 Project의 `Status` 필드로 관리한다.
`blocked`와 `needs-triage`는 상태 흐름을 보조하는 라벨로만 사용한다.

## 우선순위 기준

- `P0`: 데이터 손실, 보안 문제 또는 릴리스를 막는 장애
- `P1`: 현재 릴리스의 핵심 기능 또는 중대한 품질 문제
- `P2`: 계획된 일반 기능과 개선
- `P3`: 장기 개선, 실험 또는 낮은 영향도의 작업

## 계획과 구현

- 큰 개발 계획은 부모 Issue로 만들고 실제 구현 단위를 하위 Issue로 분리한다.
- 부모 Issue는 하위 Issue와 최종 완료 조건이 모두 끝날 때까지 열린 상태로 유지한다.
- 구현을 시작하기 전에 검증 가능한 완료 조건을 확정한다.
- 의존 작업은 Issue dependency로 연결하고 차단된 Issue에는 `blocked` 라벨을 붙인다.
- 구현을 시작할 때 담당자를 지정하고 `In Progress`로 옮긴다.
- PR 본문에 `Closes #<issue-number>`를 넣어 병합 시 Issue가 닫히게 한다.
- 구현과 검증이 끝난 항목은 `Done`으로 옮긴다.

## 제품 계약 확인

작업을 분류할 때 다음 영향을 반드시 확인한다.

- Provider 변경은 Codex CLI와 GitHub Copilot CLI 양쪽의 동작 및 공통 출력 계약을 확인한다.
- Schema 변경은 Provider 출력, 검증기, 샘플, 렌더러와 테스트의 동기화 범위를 적는다.
- Renderer 또는 Design 변경은 `DESIGN.md`와 콘텐츠 오버플로 검증 범위를 적는다.
- v1 기능은 borderless HTML 출력 범위를 벗어나지 않는다. A4와 Slide는 별도 계획으로 관리한다.

## Milestone

Milestone은 진행 상태가 아니라 `v0.1.0`, `v0.2.0`처럼 릴리스 범위를 묶을 때만 사용한다.

## 정기 정리

- `needs-triage` Issue를 주기적으로 검토한다.
- 완료된 Issue와 병합된 PR이 `Done`인지 확인한다.
- 오래된 완료 항목은 필요할 때 Project에서 보관한다.
