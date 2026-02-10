---
name: unity-qa
description: Unity 게임 프로젝트의 종합 QA 스킬. 게임을 먼저 이해한 뒤, 그 맥락에서 문제를 찾는 Top-down 접근. 6개 도메인(Flow, Logic, Visual, Performance, Design Intent, Future Risk) 모듈을 선택적으로 실행하며, 이해 → 우려 도출 → 코드 검증 → 런타임 검증으로 점진적 심화가 가능합니다. 게임 QA, 버그 탐지, 기획 의도 검증, 성능 분석, 코드 품질 분석, 잠재 위험 탐지가 필요할 때 사용하세요.
---

## 핵심 원칙

1. **먼저 이해하고, 그다음 판단한다.** 코드 패턴을 검색하기 전에 게임이 무엇을 하는지, 어떻게 동작하는지를 먼저 파악한다.
2. **우려는 맥락에서 나온다.** "이 게임에서 이 부분이 문제가 될 수 있다"는 판단은 게임 이해 위에서만 가능하다.
3. **시나리오로 검증한다.** "사용자가 X하면 Y가 되어야 하는데, Z가 될 수 있다" 형태로 우려를 구체화하고 검증한다.
4. **위에서 아래로 내려간다.** 아키텍처 → 시스템 상호작용 → 개별 구현 순서로 분석한다. 개별 코드 패턴부터 시작하지 않는다.
5. **도메인 독립성.** 6개 모듈을 독립 선택/실행 가능.
6. **병렬 에이전트 활용.** 독립적인 분석은 Explore 에이전트를 병렬 launch.

---

## 인자

```
/unity-qa "[모듈,...] [--depth understand|concern|verify|runtime] [--design-doc 경로]"
```

| 인자 | 기본값 | 설명 |
|------|--------|------|
| 모듈 | `all` | `all` 또는 쉼표 구분: `flow,logic,visual,performance,design,future` |
| `--depth` | `concern` | `understand` / `concern` / `verify` / `runtime` |
| `--design-doc` | 없음 | 기획 문서 경로 (Design Intent 모듈용) |

**모듈 약칭 매핑:**
- `flow` → [flow-qa.md](references/flow-qa.md) — 게임 플로우, 씬 전환, 상태 머신
- `logic` → [logic-qa.md](references/logic-qa.md) — 게임 로직, 데이터 무결성, 비동기 안전성
- `visual` → [visual-qa.md](references/visual-qa.md) — UI/UX, 비주얼, 애니메이션, 카메라
- `performance` → [performance-qa.md](references/performance-qa.md) — 성능, 메모리, 렌더링
- `design` → [design-intent-qa.md](references/design-intent-qa.md) — 기획 의도 대비 구현
- `future` → [future-risk-qa.md](references/future-risk-qa.md) — 구조적 위험, 기술 부채

---

## 실행 워크플로우

### Phase 0: 게임 이해 (공통 — 모든 모듈이 공유)

> **모든 도메인 분석의 기반.** 코드를 보기 전에 게임이 무엇인지를 파악한다.
> 이 Phase의 결과는 모든 모듈에서 참조한다.

Explore 에이전트 2개를 병렬 launch한다:

**에이전트 A — 게임 구조 파악:**
- 씬 목록, 씬 전환 경로, 엔트리 포인트
- 핵심 매니저/시스템 클래스와 역할
- 데이터 흐름 (SO, XML/JSON, Addressables, PlayerPrefs)

**에이전트 B — 게임플레이 파악:**
- 메인 게임 루프 (턴제? 실시간? 이벤트 드리븐?)
- 유저 인터랙션 포인트 (버튼, 드래그, 입력)
- UI 계층 구조와 전환 패턴

결과를 종합하여 **게임 프로필**을 작성한다:

```
## 게임 프로필
- 장르/유형: [...]
- 핵심 루프: [...]
- 씬 구조: [씬 전환 그래프 요약]
- 주요 시스템: [매니저 목록과 역할]
- 데이터 구조: [SO/XML/JSON 등]
- UI 구조: [메인 Canvas/패널 구조]
```

`--design-doc`이 제공되었으면 기획 문서도 로드하여 게임 프로필에 **기획 의도**를 포함한다.
`--depth understand`이면 여기서 종료하고 게임 프로필만 출력한다.

---

### Phase 1: 우려 도출 (도메인별)

> **"이 게임에서 뭐가 잘못될 수 있는가?"**
> Phase 0의 게임 프로필을 바탕으로, 선택된 도메인별 우려 지점을 도출한다.
> 우려는 **시나리오 형태**로 구체화한다.

선택된 모듈의 reference 파일을 `Read`로 로드한다.
각 reference에는 해당 도메인에서 **발생할 수 있는 우려 유형**과 **판단 관점**이 정의되어 있다.

모듈별 Explore 에이전트를 병렬 launch한다 (최대 3개씩 batch):
- 각 에이전트는 게임 프로필 + 해당 도메인의 우려 유형을 받는다
- 게임 구조를 탐색하여 **이 게임에 해당하는 구체적 우려**를 시나리오로 도출한다

우려 시나리오는 [report-format.md](docs/report-format.md)의 **우려 시나리오 형식**을 따른다.

결과를 사용자에게 보고한다.
`--depth concern`이면 여기서 종료한다.

---

### Phase 2: 코드 검증 (도메인별)

> **도출된 우려를 코드 수준에서 추적하여 실제 문제인지 확인한다.**
> Phase 1에서 도출된 우려 시나리오를 순서대로 검증한다.

각 우려 시나리오에 대해:
1. 관련 코드를 `Read`/`find_symbol`로 읽고 실행 경로를 추적한다
2. 시나리오의 "우려"가 실제로 발생 가능한지 판단한다
3. 발생 가능하면 **확정 이슈**, 아니면 **기각** (기각 사유 기록)
4. 검증 과정에서 새로운 우려가 발견되면 추가한다

검증 결과는 [report-format.md](docs/report-format.md)의 **검증 결과 형식**을 따른다.

결과를 사용자에게 보고한다.
`--depth verify`이면 여기서 종료한다.

---

### Phase 3: 런타임 검증 (Unity MCP)

> **Phase 2에서 "런타임 확인 필요"로 표시된 이슈만 실증한다.**

1. Unity 도구 로드 및 플레이 모드 진입
2. 확정 이슈를 실제 게임에서 재현
3. 스크린샷, 상태값, 성능 데이터로 증거 수집
4. 최종 확정 또는 기각

---

### 통합 보고서

모든 Phase 완료 후 결과를 통합한다.
여러 모듈이 함께 실행되었으면 **교차 참조 분석**도 수행한다.

---

## 모듈 레지스트리

### 코어 모듈

| 모듈 | reference | 관점 |
|------|-----------|------|
| Flow QA | [flow-qa.md](references/flow-qa.md) | 게임 플로우가 의도대로 흐르는가 |
| Logic QA | [logic-qa.md](references/logic-qa.md) | 게임 로직이 정확하게 동작하는가 |
| Visual QA | [visual-qa.md](references/visual-qa.md) | 사용자에게 올바른 시각적 경험을 제공하는가 |
| Performance QA | [performance-qa.md](references/performance-qa.md) | 성능 병목 없이 부드럽게 실행되는가 |
| Design Intent QA | [design-intent-qa.md](references/design-intent-qa.md) | 기획 의도대로 구현되었는가 |
| Future Risk QA | [future-risk-qa.md](references/future-risk-qa.md) | 향후 문제가 될 구조적 위험이 있는가 |

### 확장 모듈 추가 방법
1. `references/`에 새 `{module-name}-qa.md` 추가
2. 우려 유형 + 판단 관점 + 검증 절차 구조를 따름
3. 이 레지스트리 테이블에 항목 추가

확장 후보: Physics, Audio, Input, Build/Platform, Save/Load, Network, Localization, AI/Behavior, Security, Scene/Hierarchy, Accessibility

---

## 보고서 출력 포맷

→ [report-format.md](docs/report-format.md) 참조

---

## Unity MCP 주의사항

Phase 3(런타임 검증)에서 `execute_code` 사용 시 적용.

### execute_code 제약
- `System.*` 타입 → CS0433 오류. Unity API만 사용.
- 프로젝트 타입 직접 참조 불가 → `FindAssets("ClassName t:MonoScript")` + GUID 방식.
- `.Count` → `.Count.ToString()` 또는 `$"{collection.Count}"` 사용.

### 한국어 경로
- `LoadAssetAtPath`에 한국어 경로 → null. `FindAssets`로 GUID 변환.

### 씬 전환
- `get_state` 최대 5회 재시도. 씬 전환 후 2-3초 대기.

### UI 요소
- `find_ui_elements`로 instanceId를 매번 새로 가져온다.
- ScrollView 클론 겹침 시 instanceId가 큰 것을 클릭.
