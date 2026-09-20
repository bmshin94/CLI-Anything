# CLI-Anything 전수조사 분석 정리 (한국어)

> 작성일: 2026-09-20
> 분석 대상 저장소: **https://github.com/bmshin94/CLI-Anything**
> 원본(Upstream) 저장소: **https://github.com/HKUDS/CLI-Anything**
> 프로젝트 홈페이지(CLI-Hub): **https://hkuds.github.io/CLI-Anything/**
> 테크 리포트: **https://arxiv.org/abs/2606.03854**
> 라이선스: Apache License 2.0

---

## 목차

1. [한 줄 요약](#1-한-줄-요약)
2. [프로젝트 개요](#2-프로젝트-개요)
3. [폴더 전수조사 결과](#3-폴더-전수조사-결과)
4. [핵심 메커니즘: 7단계 파이프라인](#4-핵심-메커니즘-7단계-파이프라인)
5. [5대 설계 원칙](#5-5대-설계-원칙)
6. [쉬운 설명 (비유편)](#6-쉬운-설명-비유편)
7. [설치 및 사용법](#7-설치-및-사용법)
8. [플러그인 vs 스킬 vs MCP](#8-플러그인-vs-스킬-vs-mcp)
9. [API 토큰이 필요한가](#9-api-토큰이-필요한가)
10. [깃허브에서 유명한 이유](#10-깃허브에서-유명한-이유)
11. [로컬 에이전트 구축에 도움이 되는가](#11-로컬-에이전트-구축에-도움이-되는가)
12. [수익화 아이디어](#12-수익화-아이디어)
13. [React / PHP 로 만들 수 있는가](#13-react--php-로-만들-수-있는가)
14. [한계와 로드맵](#14-한계와-로드맵)
15. [참고 링크 모음](#15-참고-링크-모음)

---

## 1. 한 줄 요약

> **"세상의 모든 GUI 소프트웨어를 AI 에이전트가 사용할 수 있는 CLI로 자동 변환해 주는 공장"**

정식 명칭은 **CLI-Anything: Making ALL Software Agent-Native** 이며,
홍콩대학교(HKU) 데이터 인텔리전스 랩(**HKUDS**)에서 만든 오픈소스 연구 프로젝트다.
슬로건은 다음과 같다.

> *Today's Software Serves Humans. Tomorrow's Users will be Agents.*

---

## 2. 프로젝트 개요

### 2.1 문제의식 — The Agent-Software Gap

AI 에이전트는 추론은 잘하지만 실제 전문 소프트웨어를 다루지 못한다.
기존 해법은 (1) 깨지기 쉬운 UI 자동화, (2) 제한적인 API, (3) 기능의 90%가 빠진 재구현뿐이었다.

| 기존의 문제 | CLI-Anything 의 해결 |
|---|---|
| AI가 진짜 전문 툴(Blender, GIMP)을 못 씀 | 실제 소프트웨어 백엔드에 직접 연결 |
| UI 자동화(스크린샷 + 클릭)는 쉽게 깨짐 | 스크린샷 0, 클릭 0. 순수 커맨드라인 |
| 에이전트는 구조화된 데이터가 필요함 | 모든 명령어에 `--json` 플래그 내장 |
| 커스텀 통합은 비용이 큼 | 플러그인 하나로 어떤 코드베이스든 자동 생성 |
| 프로토타입과 프로덕션의 간극 | 2,464개 테스트, 실제 소프트웨어 검증 |

### 2.2 왜 CLI 인가

- **구조적/조합 가능** — 텍스트 명령은 LLM 형식과 맞고 파이프로 체이닝된다
- **가볍고 범용적** — 의존성 없이 모든 시스템에서 동작
- **자기 설명적** — `--help` 가 곧 자동 문서
- **검증됨** — Claude Code 가 매일 수천 개 워크플로우를 CLI로 처리
- **에이전트 우선 설계** — JSON 출력으로 파싱 복잡도 제거
- **결정론적** — 예측 가능한 에이전트 동작

### 2.3 규모 (실측)

| 항목 | 수치 |
|---|---|
| 최상위 디렉토리 | 83개 |
| 저장소 크기 (.git 제외) | 약 67MB |
| 파이썬 파일 | 1,267개 |
| 파이썬 코드 라인 | 약 310,796줄 |
| 테스트 파일 | 164개 |
| 문서화된 테스트 통과 수 | 2,464개 (100% pass) |
| 자체 하네스 레지스트리 | 79개 (`registry.json`) |
| 공개 서드파티 CLI | 24개 (`public_registry.json`) |
| 통합 SKILL.md | 71개 (`skills/`) |

---

## 3. 폴더 전수조사 결과

### 3.1 생성기(Generator) — "공장 기계"

| 경로 | 역할 |
|---|---|
| `cli-anything-plugin/` | Claude Code 플러그인 본체. 슬래시 커맨드 5개 |
| `cli-anything-plugin/HARNESS.md` | **방법론 SOP (747줄)** — 단일 진실 공급원 |
| `cli-anything-plugin/guides/` | 심화 가이드 8종 (MCP 백엔드, 필터 변환, 타임코드 정밀도, 세션 락킹, PyPI 배포, 프리뷰 방법론, 스킬 생성, 자동저장/드라이런) |
| `cli-anything-plugin/skill_generator.py` | Click 데코레이터 파싱 → SKILL.md 자동 생성 |
| `cli-anything-plugin/repl_skin.py` | 전 CLI 공용 REPL 인터페이스 |
| `cli-anything-plugin/templates/SKILL.md.template` | 스킬 문서 템플릿 |
| `.claude-plugin/marketplace.json` | Claude Code 마켓플레이스 등록 매니페스트 |

**플러그인 커맨드**

| 커맨드 | 설명 |
|---|---|
| `/cli-anything <path-or-repo>` | 7단계 전체 파이프라인으로 CLI 하네스 생성 |
| `/cli-anything:refine <path> [focus]` | 갭 분석 후 커버리지 확장 (비파괴적, 반복 실행 가능) |
| `/cli-anything:test <path>` | 테스트 실행 + TEST.md 갱신 |
| `/cli-anything:validate <path>` | HARNESS.md 표준 준수 검증 |
| `/cli-anything:list` | 전체 도구 목록 |

### 3.2 멀티 플랫폼 포팅

| 경로 | 대상 플랫폼 |
|---|---|
| `.pi-extension/cli-anything/` | Pi Coding Agent |
| `cursor-plugin/`, `.cursor-plugin/` | Cursor Desktop |
| `codex-skill/` | OpenAI Codex (bash / PowerShell 설치 스크립트) |
| `hermes-skill/` | Hermes Agent |
| `reasonix-skill/` | Reasonix |
| `qoder-plugin/` | Qodercli |
| `opencode-commands/` | OpenCode (실험적) |

→ 하나의 방법론을 7개 이상 에이전트 생태계에 동시 배포한 구조.

### 3.3 생성된 CLI 하네스 (60여 개)

표준 구조:

```
<software>/agent-harness/
├── <SOFTWARE>.md                  # 소프트웨어 전용 SOP
├── setup.py                       # cli-anything-<software> 패키지
└── cli_anything/<software>/
    ├── <software>_cli.py          # Click 기반 메인 CLI
    ├── __main__.py
    ├── core/                      # project, layers, filters, canvas, media, export, session ...
    ├── utils/
    │   ├── repl_skin.py           # 공용 REPL 스킨
    │   └── <software>_backend.py  # 실제 소프트웨어 호출 래퍼
    ├── skills/SKILL.md            # 패키지 내장 스킬 문서
    └── tests/                     # test_core.py, test_full_e2e.py, TEST.md
```

**분야별 목록**

| 분야 | 하네스 |
|---|---|
| 크리에이티브 | blender, gimp, krita, inkscape, audacity, kdenlive, shotcut, comfyui, live2d, musescore, wavetone, rekordbox, openscreen, videocaptioner, sketch, seaclip |
| 과학 / CAD | freecad, QGIS, 3MF, cloudcompare, unimol_tools |
| 게임 / 그래픽스 | godot, sbox, renderdoc, nsight-graphics, unrealinsights, slay_the_spire_ii |
| 지식 관리 | obsidian, joplin, zotero, calibre, siyuan, mubu, notebooklm |
| AI / API | ollama, exa, minimax, novita, anygen, dify-workflow, chromadb |
| 개발 / 인프라 | pm2, n8n, wiremock, lldb, adguardhome, jumpserver, iterm2, cc-switch, tigris, macrocli |
| 오피스 / 비즈니스 | libreoffice, mailchimp, zoom, firefly-iii, openrefine, drawio, mermaid, intelwatch, rms |
| 웹 / 브라우저 | browser(DOMShell MCP), safari, web-yu-pri, quietshrink |

**테스트 규모 예시**

```
blender       208 passed   (150 unit + 58 e2e)
inkscape      202 passed   (148 unit + 54 e2e)
sbox          244 passed   (157 unit + 17 orchestrator + 50 e2e + 20 exit-code)
audacity      161 passed   (107 unit + 54 e2e)
libreoffice   158 passed   (89 unit + 69 e2e)
gimp          107 passed   (64 unit + 43 e2e)
joplin        134 passed   (107 unit + 27 e2e)
─────────────────────────────────────────────
TOTAL       2,464 passed   100% pass rate
```

### 3.4 CLI-Hub — 패키지 매니저 & 마켓플레이스

| 경로 | 내용 |
|---|---|
| `cli-hub/cli_hub/` | PyPI 패키지 `cli-anything-hub` 소스 (cli.py 42KB, preview.py 67KB, installer.py 21KB, matrix.py 21KB) |
| `registry.json` | 자체 하네스 79개 등록 (카테고리 31종) |
| `public_registry.json` | 서드파티 공개 CLI 24개 (pip / npm / go / brew 설치 지원) |
| `cli-hub-meta-skill/SKILL.md` | 에이전트가 스스로 CLI를 탐색·설치하게 하는 메타 스킬 |
| `cli-hub-matrix/` | 워크플로우 매트릭스 5종 (video-creation, 3d-cad, game-development, image-design, knowledge-research) |
| `docs/hub`, `.github/workflows/deploy-pages.yml` | 웹 허브 프론트엔드 및 배포 |

**워크플로우 매트릭스 개념**
단일 CLI 하나가 아니라 *capability × provider* 매핑으로 워크플로우 전체를 구성한다.
예) `video-creation` 매트릭스는 `text.transcribe`, `visual.generate` 같은 의도(intent)를
하네스 CLI / 공개 CLI / 파이썬 라이브러리 / 네이티브 바이너리 / 클라우드 API 중에서 골라 연결한다.

```bash
cli-hub matrix install   video-creation
cli-hub matrix info      video-creation
cli-hub matrix preflight video-creation
```

### 3.5 통합 스킬 카탈로그 `skills/`

71개의 `cli-anything-*` SKILL.md 가 단일 디렉토리에 통합되어 있어
다음 한 줄로 설치 가능하다.

```bash
npx skills add HKUDS/CLI-Anything --skill <skill-name> -g -y
```

### 3.6 CI/CD (`.github/workflows/`)

| 워크플로우 | 역할 |
|---|---|
| `check-root-skills.yml` | 루트 스킬 검증 |
| `check-codex-skill.yml` | Codex 스킬 리소스 동기화 확인 |
| `check-cursor-plugin.yml` | Cursor 플러그인 동기화 확인 |
| `publish-cli-hub.yml` | PyPI 자동 배포 |
| `deploy-pages.yml` | GitHub Pages(웹 허브) 배포 |
| `pr-labeler.yml`, `pr-labeler-tests.yml` | PR 자동 라벨링 |

이슈 템플릿으로 `bug_report`, `feature_request`, `cli-wishlist`, `contributor-signup` 이 운영된다.

---

## 4. 핵심 메커니즘: 7단계 파이프라인

`/cli-anything ./gimp` 한 줄이 아래 전 과정을 수행한다.

| Phase | 이름 | 내용 |
|---|---|---|
| 1 | **Analyze** | 소스코드 스캔, GUI 액션 ↔ API 매핑 |
| 2 | **Design** | 커맨드 그룹, 상태 모델, 출력 포맷 설계 |
| 3 | **Implement** | Click CLI 구현 + REPL + JSON 출력 + undo/redo |
| 4 | **Plan Tests** | `TEST.md` 에 단위/E2E 테스트 계획 작성 |
| 5 | **Write Tests** | 테스트 코드 실제 구현 |
| 6 | **Document** | 실행 결과를 `TEST.md` 에 기록 |
| 6.5 | **SKILL.md 생성** | Click 메타데이터 추출 → 에이전트용 스킬 문서 |
| 7 | **Publish** | `setup.py` 생성 → `pip install -e .` → PATH 등록 |

**테스트 4계층**

| 계층 | 검증 내용 |
|---|---|
| Unit | 모든 코어 함수를 합성 데이터로 개별 검증 |
| E2E (native) | 프로젝트 파일 생성 파이프라인 (ODF ZIP 구조, MLT XML, SVG 적합성) |
| E2E (true backend) | 실제 소프트웨어 호출 + 산출물 검증 (LibreOffice → `%PDF-` 매직바이트, Blender → 렌더된 PNG) |
| CLI subprocess | 설치된 명령을 `subprocess.run` 으로 호출해 JSON 출력 검증 |

---

## 5. 5대 설계 원칙

1. **Authentic Software Integration** — 재구현 금지.
   유효한 프로젝트 파일(ODF, MLT XML, SVG)을 만들고 렌더링은 실제 앱에 위임한다.
   *"We build structured interfaces TO software, not replacements."*
2. **Flexible Interaction Models** — 모든 CLI가 이중 모드.
   인자 없이 실행하면 상태를 가진 REPL, 인자를 주면 원샷 서브커맨드.
3. **Consistent User Experience** — `repl_skin.py` 공유로 배너/프롬프트/히스토리/진행표시 통일.
4. **Agent-Native Design** — 모든 명령에 `--json`. 에이전트는 `--help` 와 `which` 로 능력을 발견한다.
5. **Zero Compromise Dependencies** — 백엔드가 없으면 테스트는 skip 이 아니라 **fail**.
   가짜 구현과 우아한 성능저하(graceful degradation)를 허용하지 않는다.

---

## 6. 쉬운 설명 (비유편)

### 6.1 "손 없는 천재" 비유

AI 에이전트를 *말은 할 줄 알지만 손이 없는 천재*라고 하자.

- **기존 방식(GUI 자동화)**: 스크린샷을 보여주며 "여기 클릭, 아니 3픽셀 왼쪽" 이라고 지시.
  창 크기만 바뀌어도 전부 깨진다.
- **CLI-Anything 방식**: 전화기를 쥐여준다.
  `gimp filter add brightness --factor 1.3` 이라고 말하면 끝.

**CLI-Anything = 소프트웨어마다 그 "전화기"를 자동으로 만들어 주는 기계.**

### 6.2 키오스크 비유

| 요소 | 비유 |
|---|---|
| GUI 소프트웨어 | 메뉴판 없는 식당 |
| CLI 하네스 | 키오스크 메뉴판 |
| 7단계 파이프라인 | 키오스크를 자동으로 만들어 주는 기계 |
| CLI-Hub | 전국 키오스크를 모아 둔 배달 앱 |
| SKILL.md | AI가 읽는 키오스크 사용설명서 |
| 메타 스킬 | 알아서 맛집 찾아 주문까지 해 주는 비서 |

### 6.3 3층 구조

```
┌──────────────────────────────────────────┐
│ 3층: CLI-Hub                              │
│   설치된 리모컨을 관리하는 앱스토어          │
│   pip install cli-anything-hub            │
├──────────────────────────────────────────┤
│ 2층: 60여 개 완성된 CLI 하네스              │
│   이미 만들어진 리모컨들                    │
│   blender, gimp, obsidian, ollama ...     │
├──────────────────────────────────────────┤
│ 1층: 생성기 플러그인                        │
│   리모컨을 만드는 공장 기계                 │
│   /cli-anything ./내소프트웨어              │
└──────────────────────────────────────────┘
```

어느 층부터 써도 된다. 바로 쓰고 싶으면 3층, 직접 만들려면 1층.

### 6.4 절대 오해하면 안 되는 점

> **CLI-Anything 은 Blender 를 다시 만들지 않는다. Blender 에 리모컨을 달아 준다.**

`cli-anything-libreoffice` 로 PDF 를 만들면 진짜 LibreOffice 엔진이 돌아 진짜 PDF 가 나온다.
그래서 테스트도 "PDF 앞 4바이트가 `%PDF-` 인가" 까지 검증한다.

---

## 7. 설치 및 사용법

### 7.1 그냥 쓰고 싶을 때 (가장 쉬움)

```bash
pip install cli-anything-hub

cli-hub list
cli-hub search image

cli-hub install gimp
cli-hub info gimp
cli-hub launch gimp
```

| 명령어 | 기능 |
|---|---|
| `cli-hub list` | 레지스트리 전체 목록 |
| `cli-hub search <키워드>` | 검색 |
| `cli-hub info <이름>` | 상세 정보 |
| `cli-hub install <이름>` | 설치 |
| `cli-hub update <이름>` | 업데이트 |
| `cli-hub uninstall <이름>` | 삭제 |
| `cli-hub launch <이름> [args]` | 실행 |

> **주의**: `cli-hub install blender` 를 해도 실제 Blender 본체는 별도 설치가 필요하다.
> CLI 는 리모컨일 뿐이며 TV 본체는 따로 있어야 한다.

### 7.2 에이전트에게 자율권을 줄 때 (메타 스킬)

```bash
npx skills add HKUDS/CLI-Anything --skill cli-hub-meta-skill -g -y
```

그 뒤 프롬프트:

```
Find appropriate CLI software in CLI-Hub and complete the task: <하고 싶은 작업>
```

지원 에이전트: OpenClaw, Nanobot, Claude Code, Codex, Reasonix, Antigravity 등
SKILL 호환 에이전트 전반.

### 7.3 새 CLI 를 직접 만들 때 (Claude Code 기준)

```bash
/plugin marketplace add HKUDS/CLI-Anything
/plugin install cli-anything

/cli-anything ./내소프트웨어
/cli-anything https://github.com/blender/blender

/cli-anything:refine ./gimp "이미지 배치 처리와 필터 강화"
/cli-anything:test ./gimp
/cli-anything:validate ./gimp
```

**전제 조건**

- Python 3.10 이상
- 대상 소프트웨어의 소스 또는 로컬 설치본
- 프론티어급 코딩 에이전트
- Windows 는 Git for Windows(`bash`, `cygpath`) 또는 WSL 필요

**문제 해결**: `Unknown skill: cli-anything` 이 나오면
`/reload-plugins` → `/help cli-anything` → 필요 시 마켓플레이스 재추가/재설치 순으로 확인.

### 7.4 수동 설치 (마켓플레이스를 쓰지 않을 때)

```bash
git clone https://github.com/HKUDS/CLI-Anything.git
cp -r CLI-Anything/cli-anything-plugin ~/.claude/plugins/cli-anything
# Claude Code 에서 /reload-plugins
```

### 7.5 생성된 CLI 사용

```bash
cd gimp/agent-harness && pip install -e .

# 원샷 모드
cli-anything-gimp project new --width 1920 --height 1080
cli-anything-gimp --json layer add-from-file photo.jpg --name "배경"

# 인자 없이 실행하면 REPL 모드 진입
cli-anything-gimp
```

---

## 8. 플러그인 vs 스킬 vs MCP

**정답: 셋 다 해당하지만, 본질은 "방법론 + 그 결과물"이다.**

| 형태 | 해당 부분 | 설명 |
|---|---|---|
| **플러그인** | `cli-anything-plugin/`, `cursor-plugin/`, `.pi-extension/`, `qoder-plugin/` | 슬래시 커맨드를 추가하는 정식 플러그인 |
| **스킬** | `skills/` 71개, `cli-hub-meta-skill/`, `codex-skill/`, `hermes-skill/`, `reasonix-skill/` | SKILL.md 표준 문서. `npx skills` 로 설치 |
| **MCP** | `browser/`, `safari/`, `firefly-iii/`, `cc-switch/` | **백엔드 방식 중 하나로만** 사용 |
| **순수 패키지** | 60여 개 하네스 | `pip install` 로 설치되는 일반 CLI |

### 8.1 MCP 의 포지션

`cli-anything-plugin/guides/mcp-backend.md` 의 정의:

> *"For services that expose an MCP (Model Context Protocol) server instead of a traditional CLI."*

즉 MCP 는 **대체재가 아니라 재료**다.
대상 소프트웨어에 CLI 가 없고 MCP 서버만 있으면, MCP 를 감싸서 CLI 로 만든다.
실제로 `browser/agent-harness/cli_anything/browser/utils/domshell_backend.py` 는
`mcp.ClientSession` / `stdio_client` 로 MCP 서버를 호출한 뒤 동기 래퍼를 제공한다.

### 8.2 프로젝트의 입장

이 프로젝트는 사실상 **"MCP 보다 CLI 가 낫다"** 는 주장을 담고 있다.
MCP 는 서버 기동과 프로토콜 정합이 필요하지만, CLI 는 바이너리 하나와 `--help` 면 충분하다.

**정리**
- 설치 관점 → 플러그인 / 스킬
- 사용 관점 → CLI
- MCP → 선택적 백엔드

---

## 9. API 토큰이 필요한가

**대부분 불필요. 일부만 필요.** (소스 grep 으로 실측)

### 9.1 토큰이 필요 없는 하네스 (다수)

로컬 소프트웨어를 감싸는 것들:
`blender`, `gimp`, `inkscape`, `audacity`, `libreoffice`, `freecad`, `QGIS`, `krita`,
`kdenlive`, `shotcut`, `musescore`, `drawio`, `3MF`, `cloudcompare`, `lldb`,
`renderdoc`, `godot`, `calibre`, `ollama`(로컬 LLM) 등.

→ **CLI-Anything 자체와 CLI-Hub 는 어떤 토큰도 요구하지 않는다.**

### 9.2 토큰이 필요한 하네스 (실제 환경변수)

| 환경변수 | 대상 |
|---|---|
| `EXA_API_KEY` | Exa 웹 검색 |
| `MAILCHIMP_API_KEY` | Mailchimp Marketing |
| `MINIMAX_API_KEY` | MiniMax AI |
| `NOVITA_API_KEY` | Novita AI |
| `OPENAI_API_KEY` | OpenAI 호환 백엔드 |
| `N8N_API_KEY` | n8n 워크플로우 |
| `OBSIDIAN_API_KEY` | Obsidian Local REST API |
| `SIYUAN_TOKEN` | SiYuan 노트 |
| `RMS_API_TOKEN` | Teltonika RMS |
| `MACROCLI_API_KEY` | MacroCLI |
| `DOMSHELL_TOKEN` | DOMShell 브라우저 자동화 |
| `AGH_PASSWORD` | AdGuard Home |
| `WIREMOCK_PASSWORD` | WireMock |

외부 클라우드 서비스를 호출하는 CLI 만 해당 서비스의 키를 요구한다.

### 9.3 반드시 확인할 사항 — 텔레메트리

`cli-hub/cli_hub/analytics.py` 에 사용 통계 수집 코드가 포함되어 있다.

```python
ANALYTICS_PROVIDER = "posthog"
POSTHOG_API_HOST = "https://us.i.posthog.com"
POSTHOG_PROJECT_TOKEN = "phc_ovP8d5..."
```

- 파일 상단 주석은 *"Lightweight, opt-out-able analytics"* 로, 비활성화가 가능하다고 명시한다.
- `CLAUDE_CODE`, `CLAUDECODE` 등 환경변수를 감지해 에이전트 실행 환경을 태깅한다.
- **사내망·보안 환경에서 사용할 경우 반드시 opt-out 여부를 확인하고 꺼야 한다.**

---

## 10. 깃허브에서 유명한 이유

1. **타이밍** — "에이전트에게 도구를 어떻게 쥐여줄 것인가"가 최대 화두인 시점에,
   MCP 의 대안이자 보완재 포지션을 정확히 잡았다.
2. **강력한 한 줄 카피** — *"Today's Software Serves Humans. Tomorrow's Users will be Agents."*
3. **말이 아니라 물량** — 60여 개 하네스, 31만 줄 파이썬, 2,464개 테스트 100% 통과.
   레퍼런스 구현이 아니라 완성된 제품 수준이다.
4. **대학 연구실 + 논문 백업** — HKUDS 는 LightRAG 등 히트작을 낸 랩이라 기존 팔로워 기반이 있고,
   arXiv 테크 리포트가 신뢰도를 더한다.
5. **낮은 기여 진입장벽** — 컨트리뷰터 신청 이슈 템플릿, CLI 위시리스트 템플릿,
   레지스트리 JSON 의 `contributors` 필드(기여자 이름 박제).
   실제 PR 번호가 400번대에 이를 만큼 활발하다.
6. **네트워크 효과** — Claude Code, Cursor, Codex, Pi, OpenClaw, Hermes, Reasonix,
   OpenCode, Qodercli 등 거의 모든 에이전트 생태계에 동시 배포.
7. **비주얼 마케팅** — Trendshift 배지, 실제 데모 GIF(Blender 드론, FreeCAD 큐리오시티 로버,
   Slay the Spire 게임플레이, 자막 before/after), 4개 국어 README(영/중/일/독).

---

## 11. 로컬 에이전트 구축에 도움이 되는가

**매우 도움이 된다. 다만 "부품"으로 활용해야 한다.**

| 도움 되는 점 | 설명 |
|---|---|
| 실제 능력 부여 | 로컬 에이전트의 최대 약점인 "할 줄 아는 일이 없음"을 60개 도구로 해소 |
| 설계 교과서 | `HARNESS.md` 747줄이 에이전트 친화적 도구 설계 SOP |
| 세션/상태 관리 패턴 | `core/session.py` 의 undo/redo, 3층 영속성 모델을 그대로 차용 가능 |
| SKILL.md 자동 생성기 | Click 데코레이터 → 스킬 문서 변환 로직을 자체 툴에 적용 가능 |
| 메타 스킬 | 에이전트가 스스로 도구를 탐색·설치하는 패턴의 레퍼런스 구현 |
| `--json` 규약 | 모든 출력이 파싱 가능해 에이전트 루프에 즉시 연결 |
| 매트릭스 패턴 | capability × provider 매핑 — 멀티툴 오케스트레이션 설계의 정석 |

### 11.1 한계 (공식 Limitations)

1. **프론티어급 모델 필요** — 생성 품질이 모델 성능에 크게 의존한다.
   소형 로컬 모델로 `/cli-anything` 을 돌리면 결과가 불완전하다.
   단, **이미 생성된 CLI 를 실행하는 것은 작은 모델로도 충분하다.**
2. **소스코드 의존** — 컴파일된 바이너리만 있고 디컴파일이 필요한 경우 품질이 크게 저하된다.
3. **반복 보강 필요** — 단일 실행으로 전 기능이 커버되지 않아 `/refine` 반복이 필요하다.

### 11.2 권장 아키텍처

```
[로컬 LLM (Ollama)]
      ↓ 도구 호출
[에이전트 루프 (Python)]
      ↓ subprocess
[CLI-Anything 하네스]  ← --json 으로 구조화된 결과 수신
      ↓
[실제 소프트웨어 (Blender, GIMP, LibreOffice ...)]
```

**핵심 전략: 생성은 클라우드 프론티어 모델로, 실행은 로컬 모델로.**
`cli-anything-ollama` 하네스가 이미 있어 로컬 LLM 관리까지 CLI 로 통합할 수 있다.

---

## 12. 수익화 아이디어

> 전제: 라이선스가 **Apache 2.0** 이므로 상업적 이용·수정·재배포가 모두 가능하다.
> 저작권 고지와 변경사항 명시 의무만 지키면 된다.

### 12.1 Tier 1 — 즉시 시작 가능 (난이도 하)

#### (1) 사내 툴 에이전트화 컨설팅 / SI — **가장 현실적**

| 항목 | 내용 |
|---|---|
| 타겟 | 레거시 ERP·그룹웨어·내부 어드민을 보유한 중견기업 |
| 제공 | 대상 소프트웨어에 `/cli-anything` 적용 → 전용 CLI + 에이전트 구축 |
| 가격대 | 툴 1개당 500~2,000만원, 월 유지보수 100~300만원 |
| 강점 | 7단계 파이프라인이 공수를 대폭 절감 (2개월 → 1주 수준) |

핵심 세일즈 포인트: *"우리 회사 시스템은 API 가 없어서 AI 도입을 못 한다"* 는 고객에게 해답을 제공.

#### (2) 교육 / 강의 콘텐츠

- 온라인 강의: "AI 에이전트에게 도구 만들어주기 — CLI-Anything 완전정복"
- 기업 출강: "우리 회사 소프트웨어를 에이전트 네이티브로" (일 80~150만원)
- 유튜브: Blender CLI 로 3D 자동 생성하는 데모는 조회 성과가 좋은 소재

#### (3) 에이전트 자동화 제작소 (외주)

하네스를 조합해 완성형 워크플로우를 판매:

- "영상 → 자동 자막 + 썸네일 + 숏폼 3종" (videocaptioner + kdenlive + gimp)
- "제품 사진 100장 일괄 보정 + 배경 제거 + 카탈로그 PDF" (gimp + libreoffice)
- "3D 모델 → 다각도 렌더 + 제품 페이지" (blender + drawio)

건당 50~300만원, 템플릿화 후 재판매 가능.

### 12.2 Tier 2 — 제품화 (난이도 중)

#### (4) CLI-Hub GUI 데스크탑 앱 — **추천**

현재 CLI-Hub 는 터미널 전용이라, 비개발자 크리에이터·디자이너 접근성이 낮다.

| 기능 | 구현 방식 |
|---|---|
| 카드형 CLI 브라우징 | `registry.json` + `public_registry.json` 렌더링 |
| 원클릭 설치 | `cli-hub install <name>` 자식 프로세스 호출 |
| 시각적 워크플로우 빌더 | 매트릭스를 드래그앤드롭 UI 로 |
| API 키 금고 | 9장의 13개 환경변수 안전 관리 |
| 실행 로그 뷰어 | `--json` 출력을 테이블/차트로 시각화 |

수익 모델: 무료 티어(설치·실행) + Pro $9~19/월(워크플로우 저장, 팀 공유, 클라우드 실행).

#### (5) Managed CLI Cloud (SaaS) — 확장성 최대

```
문제: Blender 를 쓰려면 Blender 를 설치해야 한다 → 무겁고 어렵다
해결: 서버에 전부 설치해 두고 API 로 제공한다
```

- `POST /api/blender/render` → 서버 GPU 가 렌더 후 결과 URL 반환
- 60여 개 소프트웨어를 설치 없이 API 로 제공, 실행 시간/크레딧 과금
- 타겟: AI 앱 개발사, 자동화 스타트업
- 가격: $29 / $99 / $499 티어 또는 크레딧 종량제
- **주의: 각 소프트웨어의 라이선스를 반드시 개별 확인해야 한다.**
  (GPL 계열의 SaaS 제공은 대체로 가능하나, 상용 소프트웨어는 불가)

#### (6) 버티컬 특화 패키지 판매

- 의료 영상(DICOM) 처리 CLI 팩
- 건축 BIM/CAD 자동화 팩 (freecad + QGIS 확장)
- 회계/세무 자동화 팩 (firefly-iii + libreoffice)
- 게임 에셋 파이프라인 팩 (blender + godot + sbox)

팩당 $199~999 일회성 또는 $49/월 구독.

### 12.3 Tier 3 — 장기 / 고위험 (난이도 상)

#### (7) 마켓플레이스 운영 (중개 수수료)
CLI-Hub 를 포크해 유료 하네스 거래소 운영, 판매액의 20~30% 수수료.
단, 원본이 무료 오픈소스라 차별화가 어렵다.

#### (8) 엔터프라이즈 온프레미스
텔레메트리 완전 제거 + 감사 로그 + SSO + RBAC + 폐쇄망 배포 지원.
연 3,000만~1억원 규모.

#### (9) 에이전트 벤치마크 사업
공식 로드맵의 *"Benchmark suite for agent task completion rates"* 항목과 연결된다.
CLI 로 태스크 생성·평가를 자동화할 수 있으므로,
AI 모델 기업에 "실제 소프트웨어 사용 능력" 평가 서비스를 제공할 수 있다. 블루오션 영역.

### 12.4 권장 실행 순서

```
1단계 (0~3개월)   : 데모 3종 제작 → 유튜브/블로그로 인지도 + 포트폴리오 확보
2단계 (3~6개월)   : 컨설팅/외주 수주로 현금흐름 확보
3단계 (6~12개월)  : Electron + React GUI 앱 출시 (무료 → Pro)
4단계 (12개월~)   : 버티컬 SaaS 로 전환
```

리스크가 가장 낮고 현금흐름이 빠른 것은 (1) 컨설팅이며,
이를 수행하면서 (4) GUI 앱을 병행 개발하는 것이 최적 조합이다.

---

## 13. React / PHP 로 만들 수 있는가

질문을 세 가지로 나누어 답한다.

### 13.1 "React/PHP 프로젝트를 CLI 로 만들 수 있는가" → **가능. 오히려 최적**

```bash
/cli-anything ./내-react-프로젝트
/cli-anything ./내-laravel-앱
```

근거:

- React/Laravel 은 소스코드가 텍스트이므로 "소스코드 필요" 조건을 완벽히 충족한다
- Laravel 은 이미 Artisan CLI 가 있어 이를 감싸면 된다
- REST 라우트만 읽어도 커맨드 그룹이 자연스럽게 도출된다
- 저장소에 이미 `n8n`(Node), `dify-workflow`, `siyuan`, `mailchimp`(순수 REST API),
  그리고 **`firefly-iii`(PHP/Laravel 기반 가계부 앱)** 하네스가 존재한다
  → **PHP 앱도 가능하다는 실증 사례**

### 13.2 "생성기 자체를 React/PHP 로 포팅할 수 있는가" → **가능하지만 비권장**

| 이유 | 설명 |
|---|---|
| 생성기는 사실상 문서다 | 핵심은 `HARNESS.md` 이고 실제 작업은 LLM 이 수행한다 |
| 포팅 = 번역 작업 | 기존 7개 플랫폼 포팅도 마크다운 복사 + 설치 스크립트가 전부다 |
| 생태계 불일치 | 산출물이 `pip install` 파이썬 패키지라 전체가 파이썬 중심이다 |

**더 나은 대안**: 생성기를 바꾸지 말고 **출력 언어를 바꾼다.**
`HARNESS.md` 를 수정해 "Click 대신 Node.js Commander 로 생성" 하도록 지시하면 된다.
실제로 `sketch/agent-harness` 는 Node.js + Jest(19 테스트) 로 구현되어 있다.

### 13.3 "이 생태계 위에 React/PHP 로 무엇을 만들 수 있는가" → **여기가 진짜 기회**

| 아이디어 | 스택 | 설명 |
|---|---|---|
| CLI-Hub 웹 대시보드 | React + Vite | registry.json 기반 검색/필터/비교 UI |
| Electron 데스크탑 앱 | React + Electron | 12.2 (4)번 아이템. 가장 유망 |
| 워크플로우 비주얼 빌더 | React Flow | 노드 연결로 CLI 체이닝 → 실행 (n8n 스타일) |
| 웹 실행 콘솔 | React + xterm.js + WebSocket | 브라우저에서 CLI 실행/결과 스트리밍 |
| PHP 백엔드 오케스트레이터 | Laravel + Queue/Horizon | 작업 큐 → 워커가 CLI 실행 → 결과 DB 저장 (SaaS 백엔드) |
| PHP 관리자 패널 | Laravel Filament / Nova | 팀별 사용량, API 키 관리, 과금 |

**권장 조합**

```
프론트엔드 : React + TypeScript + TailwindCSS
백엔드     : Laravel (PHP) 또는 FastAPI (Python)
실행층     : Python subprocess → cli-anything-* (--json)
큐         : Redis + Worker
```

PHP(Laravel)는 `symfony/process` 로 CLI 를 호출하고 Queue/Horizon 으로 비동기 처리하면 된다.
**CLI 가 JSON 을 출력하므로 어떤 언어에서도 붙일 수 있다는 점이 이 방식의 최대 장점이다.**

---

## 14. 한계와 로드맵

### 14.1 공식 Limitations

- 프론티어급 모델 의존 (약한 모델은 불완전한 CLI 생성)
- 소스코드 가용성 의존 (바이너리 전용 소프트웨어는 품질 저하)
- 반복 정제 필요 (`/refine` 다회 실행 권장)

### 14.2 공식 Roadmap

- [ ] 더 많은 애플리케이션 카테고리 지원 (CAD, DAW, IDE, EDA, 과학 도구)
- [ ] 에이전트 작업 완수율 벤치마크 스위트
- [ ] 커뮤니티 기여 하네스 (사내/커스텀 소프트웨어)
- [ ] Claude Code 외 추가 에이전트 프레임워크 통합
- [ ] 클로즈드소스 소프트웨어 및 웹 서비스의 API 패키징 지원
- [x] 에이전트 스킬 탐색/오케스트레이션용 SKILL.md 생성

---

## 15. 참고 링크 모음

| 구분 | 링크 |
|---|---|
| **이 저장소 (포크)** | https://github.com/bmshin94/CLI-Anything |
| **원본 저장소 (Upstream)** | https://github.com/HKUDS/CLI-Anything |
| CLI-Hub 웹 허브 | https://hkuds.github.io/CLI-Anything/ |
| 테크 리포트 (arXiv) | https://arxiv.org/abs/2606.03854 |
| 기여 가이드 | https://github.com/HKUDS/CLI-Anything/blob/main/CONTRIBUTING.md |
| 컨트리뷰터 신청 | https://github.com/HKUDS/CLI-Anything/issues/new?template=contributor-signup.yml |
| CLI 위시리스트 | https://github.com/HKUDS/CLI-Anything/issues/new?template=cli-wishlist.yml |
| ClawHub | https://clawhub.ai/yuh-yang/cli-anything-hub |
| SkillHub | https://www.skillhub.club/web/skills/itsyuhao-cli-anything-hub |
| SkillHub.cn | https://skillhub.cn/skills/cli-hub-meta-skill |

### 저장소 내부 주요 문서

| 문서 | 설명 |
|---|---|
| `README.md` | 프로젝트 전체 개요 (영문, 1,818줄) |
| `README_CN.md` / `README_JA.md` / `README_DE.md` | 중국어 / 일본어 / 독일어 번역 |
| `cli-anything-plugin/HARNESS.md` | 방법론 SOP — 단일 진실 공급원 (747줄) |
| `cli-anything-plugin/QUICKSTART.md` | 5분 시작 가이드 |
| `cli-anything-plugin/PUBLISHING.md` | 배포 가이드 |
| `cli-anything-plugin/guides/` | 심화 가이드 8종 |
| `CONTRIBUTING.md` | 기여 방법 |
| `SECURITY.md` | 보안 정책 |
| `CITATION.cff` | 인용 정보 |
| `registry.json` / `public_registry.json` | CLI-Hub 레지스트리 |

### 인용 정보

```bibtex
@misc{yang2026clianythingagentnativecomputeruse,
      title={CLI-Anything: Towards Agent-Native Computer Use},
      author={Yuhao Yang and Tianyu Fan and Chao Huang},
      year={2026},
      eprint={2606.03854},
      archivePrefix={arXiv},
      primaryClass={cs.HC},
      url={https://arxiv.org/abs/2606.03854},
}
```

---

*본 문서는 `bmshin94/CLI-Anything` 저장소를 전수조사하여 정리한 분석 자료입니다.*
*문서 위치: 저장소 루트 `CLI-ANYTHING-ANALYSIS-KR.md` (`/docs/` 는 .gitignore 로 제외되어 있어 루트에 배치했습니다.)*
