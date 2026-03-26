# Harness

**Agent Team & Skill Architect** — Claude Code 플러그인

도메인/프로젝트에 맞는 하네스를 구성하고, 전문 에이전트를 정의하며, 에이전트가 사용할 스킬을 생성하는 메타 스킬.

## 개요

Harness는 Claude Code의 에이전트 팀 시스템을 활용하여 복잡한 작업을 전문 에이전트 팀으로 분해·조율하는 아키텍처 도구다. "하네스 구성해줘"라고 말하면, 사용자의 도메인에 맞는 에이전트 정의(`.claude/agents/`)와 스킬(`.claude/skills/`)을 자동 생성한다.

## 핵심 기능

- **에이전트 팀 설계** — 파이프라인, 팬아웃/팬인, 전문가 풀, 생성-검증, 감독자, 계층적 위임 등 6가지 아키텍처 패턴 지원
- **스킬 생성** — Progressive Disclosure 패턴으로 컨텍스트를 효율 관리하는 스킬 자동 생성
- **오케스트레이션** — 에이전트 간 데이터 전달, 에러 핸들링, 팀 조율 프로토콜 포함
- **검증 체계** — 트리거 검증, 드라이런 테스트, With-skill vs Without-skill 비교 테스트

## 워크플로우

```
Phase 1: 도메인 분석
    ↓
Phase 2: 팀 아키텍처 설계 (에이전트 팀 vs 서브 에이전트)
    ↓
Phase 3: 에이전트 정의 생성 (.claude/agents/)
    ↓
Phase 4: 스킬 생성 (.claude/skills/)
    ↓
Phase 5: 통합 및 오케스트레이션
    ↓
Phase 6: 검증 및 테스트
```

## 설치

### 마켓플레이스 등록 후 설치

#### 마켓플레이스 추가
```shell
/plugin marketplace add revfactory/harness
```

#### 플러그인 설치
```shell
/plugin install harness@harness
```

### 글로벌 스킬로 직접 설치

```shell
# skills 디렉토리를 ~/.claude/skills/harness/에 복사
cp -r skills/harness ~/.claude/skills/harness
```

## 플러그인 구조

```
harness/
├── .claude-plugin/
│   └── plugin.json                 # 플러그인 매니페스트
├── skills/
│   └── harness/
│       ├── SKILL.md                # 메인 스킬 정의 (6 Phase 워크플로우)
│       └── references/
│           ├── agent-design-patterns.md   # 6가지 아키텍처 패턴
│           ├── orchestrator-template.md   # 팀/서브에이전트 오케스트레이터 템플릿
│           ├── team-examples.md           # 실전 팀 구성 예시 5종
│           ├── skill-writing-guide.md     # 스킬 작성 가이드
│           ├── skill-testing-guide.md     # 테스트/평가 방법론
│           └── qa-agent-guide.md          # QA 에이전트 통합 가이드
└── README.md
```

## 사용법

Claude Code에서 다음과 같이 트리거한다:

```
하네스 구성해줘
하네스 설계해줘
이 프로젝트에 맞는 에이전트 팀 구축해줘
```

### 실행 모드

| 모드 | 설명 | 권장 상황 |
|------|------|----------|
| **에이전트 팀** (기본) | TeamCreate + SendMessage + TaskCreate | 2개 이상 에이전트, 협업 필요 |
| **서브 에이전트** | Agent 도구 직접 호출 | 단발성 작업, 통신 불필요 |

### 아키텍처 패턴

| 패턴 | 설명 |
|------|------|
| 파이프라인 | 순차 의존 작업 |
| 팬아웃/팬인 | 병렬 독립 작업 |
| 전문가 풀 | 상황별 선택 호출 |
| 생성-검증 | 생성 후 품질 검수 |
| 감독자 | 중앙 에이전트가 동적 분배 |
| 계층적 위임 | 상위→하위 재귀적 위임 |

## 산출물

하네스가 생성하는 파일:

```
프로젝트/
├── .claude/
│   ├── agents/          # 에이전트 정의 파일
│   │   ├── analyst.md
│   │   ├── builder.md
│   │   └── qa.md
│   └── skills/          # 스킬 파일
│       ├── analyze/
│       │   └── skill.md
│       └── build/
│           ├── skill.md
│           └── references/
```

## 요구사항

- Claude Code CLI
- 에이전트 팀 기능 활성화: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`

## 라이선스

Apache 2.0
