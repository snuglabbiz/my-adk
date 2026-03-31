# 오케스트레이션 로그

이 문서는 OhMyOpenCode 스타일 오케스트레이션을 통해 작업이 어떻게 진행되었는지 기록하는 실행/감사 로그다.

## 기록 규칙

- 워크플로우 계획이 의미 있게 바뀌었을 때, 전문 에이전트를 사용했을 때, 구현 관련 산출물이 생성·수정되었을 때 날짜 기준으로 새 항목을 추가한다.
- 기록은 사실 위주로 남긴다: 요청, 수행한 작업, 수집한 근거, 내린 결정, 변경된 산출물.
- 이 문서는 chain-of-thought를 적는 곳이 아니라 실행 기록과 의사결정 이력을 남기는 곳이다.
- 앞으로 이 문서는 **항상 한글로 작성**한다.

---

## 2026-04-01 세션 기록

### 작업 범위

- 목표: `moai-adk`에서 꼭 필요한 부분만 역공학해서, 사용자의 실제 회사 개발 워크플로우에 맞는 가벼운 `my-adk`를 설계한다.
- 대상 워크플로우는 아래와 같이 정리되었다.
  1. 요구사항 명확화
  2. 구현 계획 수립
  3. spec 분해
  4. 구현
  5. 인간 코드리뷰
  6. 문서 동기화/업데이트
  7. 커밋 전 검증

### 저장소 맥락

- 참조 저장소: `plugin/rocky/moai-adk`
- 대상 저장소: `plugin/rocky/my-adk`
- `my-adk`는 존재하며 현재 비어 있는 상태임을 확인했다.

### 수행한 오케스트레이션

#### 1. 내부 탐색 및 비교

- 로컬 `moai-adk` 구조를 읽었다. 특히 `.claude`, `.moai`, commands, skills, output styles, settings, hook 관련 파일을 중점적으로 확인했다.
- `moai-adk`는 v1 기준으로는 훨씬 넓은 범위를 갖고 있으며, 많은 부분이 선택 기능이라는 점을 확인했다.
- commands는 대체로 얇은 wrapper이고, 더 강한 오케스트레이션은 skill/rules/output-style 쪽에 있다는 점을 확인했다.

#### 2. 사용한 전문 에이전트

- `explore`: 로컬 하네스 패턴과 필수/선택 요소를 식별하는 데 사용했다.
- `librarian`: Claude Code의 공식/공개 관행과 skills, commands, output styles, plugins 관련 예시를 수집하는 데 사용했다.
- `metis`: 범위 함정, 블라인드 스팟, 과설계 위험을 점검하는 데 사용했다.
- `oracle`: 제안한 v1 구조를 압박 검증하고, 가장 작은 유효 범위를 추천받는 데 사용했다.

#### 3. 구조화된 사용자 명확화

- 선택지 기반 질문으로 계획 범위를 점진적으로 좁혔다.
- 아래 사용자 우선순위를 확인했다.
  - 플랫폼 구축보다 단순함이 중요하다.
  - 가장 중요한 가치는 문서 일관성이다.
  - 검증 결과는 spec 안에 들어가야 한다.
  - 구현 상태는 기능 spec 체크박스로 추적하는 것이 좋다.
  - `codex:gpt-5-4-prompting` 원칙은 별도 명령이 아니라 plan/spec 규칙으로 흡수해야 한다.

#### 4. 계획 축소 결정

- 처음의 더 큰 workflow framework 구상에서, 더 작은 workflow pack 방향으로 줄였다.
- v1 기본값에서 제외하기로 한 항목:
  - 큰 agent 시스템
  - hook 중심 런타임
  - 별도 `reviews/` 디렉터리
  - 별도 `verification/` 디렉터리
  - 독립 `/my-sync` 명령
- 유지/채택한 항목:
  - `requirements/`
  - `plans/`
  - `specs/`
  - clarify / plan / spec / verify 중심 명령 구조

#### 5. Content vs Form 판단

- `my-adk` v1이 문서 템플릿 최적화인지, 더 강한 workflow medium 설계인지 검토했다.
- 사용자는 **Content now, note form** 방향을 선택했다.
- 따라서 v1은 가볍게 유지하고, 좋은 commands/templates/rules에 집중하며, 더 강한 workflow engine 아이디어는 후속으로 미루기로 했다.

### 현재까지의 권장 v1 형태

- 유지할 문서 자산:
  - `requirements/`
  - `plans/`
  - `specs/`
- 핵심 동작:
  - 요구사항을 먼저 명확화한다.
  - 명확화된 요구사항을 바탕으로 계획을 작성한다.
  - spec은 관리 가능한 기능 단위로 분해한다.
  - 구현 상태는 spec 체크박스로 추적한다.
  - 검증 결과는 spec 내부에 기록한다.
  - 리뷰 피드백은 plan/spec에 다시 반영한다.

### 재사용하기로 한 프롬프팅/템플릿 규칙

`codex:gpt-5-4-prompting`에서 아래 개념을 `my-adk` 템플릿 규칙으로 흡수하기로 했다.

- 명시적인 작업 정의
- 명시적인 출력 계약
- 중간에 멈추지 않는 완료 기준
- 완료 전에 검증 루프 수행
- 근거 기반 주장 규칙
- 아래 안티패턴 방지:
  - 모호한 작업 프레이밍
  - 출력 계약 없는 조사 요청
  - 서로 무관한 작업을 한 번에 섞는 방식

### 파일 생성 이력

- 생성: `my-adk/orchestration-log.md`
- 목적: 계획/구현 과정에서 오케스트레이션이 어떻게 수행되었는지 누적 기록하기 위함

---

## 2026-04-01 추가 기록 — Content vs Form 검증

### 외부 근거 검토

- Claude Code 공식 문서와 공개 저장소 예시를 수집해 skills, output styles, hooks, plugins, workflow pack 관행을 확인했다.
- 결론은 현재 추천 방향과 일치했다.
  - 가벼운 pack은 보통 standalone `.claude/` 파일들로 시작한다.
  - 더 강한 workflow form은 enforcement, generation, distribution, measurement 계층이 추가될 때 등장한다.

### my-adk v1에 적용한 해석

- 현재 `my-adk`는 아래 범위에 머무는 것이 적절하다.
  - commands
  - skills
  - output style 1개
  - 문서 템플릿/규칙
- 아직 아래 단계로 키우지 않는다.
  - plugin 배포 시스템
  - hook 기반 강제 워크플로우 런타임
  - 생성/컴파일형 workflow framework

### 이렇게 판단한 이유

- Claude Code 공식 가이드는 빠르거나 프로젝트 특화된 워크플로우에는 standalone `.claude/` 사용을 권장하고, 공유/버전관리 필요가 커질 때 plugins로 확장하는 구조다.
- hooks는 결정적 강제(enforcement) 수단인데, 이는 현재 사용자의 v1 필요보다 강하다.
- 공개 예시들도 비슷한 분리를 보여줬다.
  - template pack = markdown 콘텐츠 + 복사 가능한 `.claude` 자산
  - workflow system = 생성 스크립트, 검증 게이트, 설치기, packaged plugin

### 잠금 결정

- `my-adk` v1은 **content-first, form은 후속 메모**로 유지한다.
- 아래 조건이 생길 때만 더 강한 form으로 확장한다.
  - 여러 저장소/팀에 공유되는 버전 관리 배포가 필요할 때
  - 품질 게이트를 결정적으로 강제해야 할 때
  - workflow 상태를 생성/동기화하는 도구가 실제로 필요해질 때

---

## 2026-04-01 추가 기록 — v1 최종 확정

### 최종 확정된 v1 구조

- 디렉터리:
  - `requirements/`
  - `plans/`
  - `specs/`
- 기본 `.claude` 구성:
  - `commands/`
  - `skills/`
  - `output-styles/`

### 최종 확정된 v1 명령

- `/my-clarify`
- `/my-plan`
- `/my-spec`
- `/my-verify`

### 각 명령의 책임

- `/my-clarify`: 모호한 요구사항을 구조화해서 `requirements/` 기준 문서로 정리한다.
- `/my-plan`: 요구사항을 구현 계획으로 바꾸고, 변경 범위/검증 기준/작업 순서를 정리한다.
- `/my-spec`: plan을 기능 단위 spec으로 분해하고, 구현/미구현/검증 체크박스를 가진 실행 문서로 만든다.
- `/my-verify`: spec 내부 검증 체크리스트를 기준으로 테스트, 로컬 확인, 리뷰 반영 여부를 점검한다.

### v1 운영 원칙

- 구현은 spec 기준으로 진행한다.
- 검증 결과는 별도 `verification/` 문서가 아니라 spec 내부에 기록한다.
- 리뷰 결과는 별도 `reviews/` 폴더보다 plan/spec 역반영을 우선한다.
- `/my-sync`, hooks, agent 중심 오케스트레이션, plugin 배포는 v1 기본 범위에서 제외한다.
- `codex:gpt-5-4-prompting`의 핵심 규칙은 `/my-plan`, `/my-spec` 템플릿 규칙으로 흡수한다.

### 이번 수정 이력

- `orchestration-log.md` 전체를 한글 기준으로 재작성했다.
- 앞으로 이 로그는 계속 한글로 누적 기록한다.

### 다음 기록 시점

아래 시점에 새 항목을 추가한다.

- v1 파일 구조가 실제로 생성될 때
- command/skill 템플릿 초안이 작성될 때
- 구현이 시작될 때
- 전문 에이전트 검토로 계획이 크게 바뀔 때

---

## 2026-04-01 추가 기록 — 한글 작성 규칙 확정

### 결정 내용

- `my-adk`에서 사용자와 직접 맞닿는 도구 내용은 한글로 작성한다.
- 여기서 한글 대상은 아래를 포함한다.
  - command 설명 문구
  - skill 본문 지침
  - 문서 템플릿 내용
  - output style 지침
  - plan/spec/verify 산출물 기본 문장

### 적용 원칙

- 사용자가 읽는 설명, 규칙, 질문, 체크리스트, 산출물 형식은 기본적으로 한글로 작성한다.
- 기술 용어, 파일 경로, 플래그, 코드 조각은 필요 시 원문 그대로 유지한다.
- 슬래시 명령 식별자나 디렉터리 이름은 안정성과 호환성을 위해 당분간 ASCII 기반을 유지하되, 명령 내부 설명과 결과물은 한글로 통일한다.

### 영향

- 이후 작성할 `/my-clarify`, `/my-plan`, `/my-spec`, `/my-verify` 관련 템플릿과 skill 내용은 한글 기준으로 만든다.
- 오케스트레이션 로그 역시 계속 한글로 누적 기록한다.

---

## 2026-04-01 추가 기록 — v1 스캐폴드 생성

### 생성한 파일 구조

- `.claude/settings.json`
- `.claude/commands/my-clarify.md`
- `.claude/commands/my-plan.md`
- `.claude/commands/my-spec.md`
- `.claude/commands/my-verify.md`
- `.claude/skills/my-clarify/SKILL.md`
- `.claude/skills/my-workflow-core/SKILL.md`
- `.claude/output-styles/my-adk/default.md`
- `requirements/_template.md`
- `plans/_template.md`
- `specs/_template.md`

### 이번 구현에서 적용한 원칙

- command 파일은 얇은 wrapper로 만들고 실제 규칙은 skill에 몰았다.
- 사용자용 설명, 템플릿, output style은 한글 기준으로 작성했다.
- 슬래시 명령 이름과 디렉터리 이름은 안정성을 위해 ASCII를 유지했다.
- 검증 결과는 별도 디렉터리 대신 spec 템플릿 내부 체크리스트로 수용했다.

### 명령/skill 역할 분리

- `my-clarify` command는 `my-clarify` skill로 직접 연결된다.
- `my-plan`, `my-spec`, `my-verify` command는 `my-workflow-core` skill에 하위 단계 인자를 넘긴다.
- `my-workflow-core`는 plan/spec/verify를 문서 중심으로 이어주는 최소 규칙만 가진다.

### 템플릿 설계 판단

- `requirements`, `plans`, `specs` 각각에 `_template.md`를 두어 바로 복사/수정 가능한 기본 골격을 만들었다.
- spec 템플릿에는 구현 체크리스트와 검증 체크리스트를 함께 둬서 사용자의 핵심 요구인 문서 일관성과 구현 상태 추적을 반영했다.

### 후속 예정 항목

- 실제 command/skill 문구를 한 번 더 다듬기
- 필요하면 `requirements/REQ-*`, `plans/PLAN-*`, `specs/SPEC-*` 네이밍 규칙 확정
- 이후 구현 단계가 시작되면 이 로그에 계속 누적 기록

---

## 2026-04-01 추가 기록 — 문구 및 템플릿 다듬기

### 다듬은 대상

- `.claude/commands/my-clarify.md`
- `.claude/commands/my-plan.md`
- `.claude/commands/my-spec.md`
- `.claude/commands/my-verify.md`
- `.claude/skills/my-clarify/SKILL.md`
- `.claude/skills/my-workflow-core/SKILL.md`
- `.claude/output-styles/my-adk/default.md`
- `requirements/_template.md`
- `plans/_template.md`
- `specs/_template.md`

### 이번 다듬기의 목적

- 실사용 시 더 자연스럽게 읽히도록 한글 문구를 정리했다.
- command 설명에 입력 예시 성격의 hint를 조금 더 분명하게 넣었다.
- skill에는 최소 출력 형식과 문서화 규칙을 보강했다.
- 템플릿에는 `마지막 수정일`, `다음 단계`, `문서 최신화 확인` 같은 운영 항목을 추가했다.

### 기대 효과

- 처음 사용하는 사람도 각 명령이 무엇을 입력받고 무엇을 내놓는지 더 쉽게 이해할 수 있다.
- requirement → plan → spec → verify 흐름이 문서 안에서도 더 자연스럽게 이어진다.
- 리뷰 이후 문서 최신화 누락을 줄일 수 있도록 spec 템플릿에 체크 포인트를 추가했다.

---

## 2026-04-01 추가 기록 — 저장소 초기화 및 원격 공개

### 수행 내용

- `my-adk` 디렉터리에서 `git init -b main`으로 저장소를 초기화했다.
- 현재 스캐폴드 전체를 첫 커밋으로 묶었다.
- 공개 GitHub 저장소를 생성하고 `origin/main`으로 푸시했다.

### 생성된 저장소

- 원격 저장소: `https://github.com/snuglabbiz/my-adk`
- 기본 브랜치: `main`

### 첫 커밋 정보

- 커밋 해시: `4061224`
- 커밋 메시지: `scaffold my-adk workflow pack`

### 의미

- 이제 `my-adk`는 로컬 초안이 아니라 원격에서 추적 가능한 최소 v1 저장소가 되었다.
- 이후 command/skill/템플릿 개선 작업은 이 저장소를 기준으로 이어갈 수 있다.
