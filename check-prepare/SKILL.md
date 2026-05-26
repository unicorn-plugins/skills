---
name: check-prepare
description: 교육생 로컬 개발 환경 사전준비 점검 및 자동 설치 (Git·gh·Node.js·VSCode·Claude Code·MCP·Python·bun·Docker·IntelliJ)
type: setup
user-invocable: true
allowed-tools: Bash, Read, Write, AskUserQuestion
---

# Check-Prepare

## 목표

Claude Code와 NPD 플러그인 사용을 위한 로컬 개발 환경을 점검함.  
자동화 가능한 항목은 직접 설치하고, 수동 설치가 필요한 항목은 가이드 링크를 안내함.

## 활성화 조건

사용자가 `/check-prepare` 호출 시 또는 "사전준비 점검", "환경 점검", "설치 확인" 키워드 감지 시.

## 진행상황 업데이트 및 재개

현재 디렉토리에 `AGENTS.md`가 있으면 각 Phase 완료 시 아래 형식으로 저장.  
최종 완료 시 `Done`으로 표기. 없으면 저장 생략.

```md
## 워크플로우 진행상황
- check-prepare: Phase3
```

진행상황 정보가 있는 경우 마지막 완료 단계 이후부터 자동 재개.

---

## 워크플로우

### Phase 0. 가이드 다운로드 및 분석

#### Step 1: 가이드 다운로드

아래 명령으로 최신 사전준비 가이드를 다운로드:

```bash
mkdir -p .temp
curl -fsSL https://raw.githubusercontent.com/unicorn-plugins/npd/refs/heads/main/resources/guides/setup/prepare.md > .temp/prepare.md
```

#### Step 2: 가이드 분석

다운로드한 `.temp/prepare.md`를 Read 도구로 읽어 아래 항목을 파악:

1. **설치 범위 구분**: 공통 필수 / 설계 단계 / 개발·배포 단계로 분류된 설치 항목 목록
2. **각 항목별 체크 명령**: 가이드에 기술된 설치 확인 명령 추출  
   (예: `git -v`, `gh --version`, `node -v` 등)
3. **버전 요구사항**: 특정 버전이 명시된 경우 해당 버전 기준으로 체크  
   (예: Python 3.13.3)
4. **OS별 특이사항**: Mac/Windows/Linux별로 다른 설치·확인 방법 파악
5. **자동 설치 가능 여부**: 가이드에 설치 명령이 제공된 항목 식별

파악한 내용을 바탕으로 Phase 1~4에서 체크를 수행함.

---

### Phase 1. 체크 범위 선택

#### Step 1: 설치 범위 안내

Phase 0에서 분석한 설치 항목을 범위별로 표 형식으로 사용자에게 안내.

#### Step 2: 사전준비 체크 범위 선택

<!--ASK_USER-->
{"title":"사전준비 체크 범위","questions":[
  {"question":"어떤 범위로 사전준비를 체크할까요?","description":"체크 범위에 따라 사전 프로그램 설치 여부를 검사합니다.","type":"radio","options":["공통 필수만 체크","설계 단계까지 체크","개발/배포 단계까지 체크","사전 프로그램 미설치"]}
]}
<!--/ASK_USER-->

선택에 따라 실행 경로 분기:
- **공통 필수만 체크** → Phase 2 → Phase 5
- **설계 단계까지 체크** → Phase 2 → Phase 3 → Phase 5
- **개발/배포 단계까지 체크** → Phase 2 → Phase 3 → Phase 4 → Phase 5
- **사전 프로그램 미설치** → 설치 가이드 안내 후 수행 중단  
  설치 가이드: [로컬 개발 환경 구성](https://github.com/unicorn-plugins/npd/blob/main/resources/guides/setup/prepare.md)

---

### Phase 2. OS 감지

```bash
uname -s
```

결과에 따라 OS 판별:
- `Linux` → Linux
- `Darwin` → Mac
- `MINGW*` / `MSYS*` / `CYGWIN*` → Windows (Git Bash)

내부 변수 `{OS}`에 저장하여 이후 Phase에서 분기에 활용.

---

### Phase 3. 공통 필수 설치 체크

`{NOT_INSTALLED}` 변수를 빈 리스트로 초기화.

Phase 0에서 분석한 **공통 필수 설치 항목**을 순서대로 체크:

- 각 항목마다 가이드에서 추출한 체크 명령을 실행
- 설치 확인 → ✅ 설치됨
- 미설치 → ⚠️ `{NOT_INSTALLED}`에 항목 추가
- 가이드에 설치 명령이 제공된 항목은 OS에 맞는 명령으로 자동 설치 시도
- 버전 요구사항이 있는 항목은 해당 버전 충족 여부도 함께 확인
- OS별 추가 설정(PATH, alias 등)이 필요한 항목은 가이드 내용에 따라 수행

---

### Phase 4. 설계 단계 설치 체크

Phase 0에서 분석한 **설계 단계 설치 항목**을 순서대로 체크.  
미설치 항목은 `{NOT_INSTALLED}`에 추가.

---

### Phase 5. 개발/배포 단계 설치 체크

Phase 0에서 분석한 **개발/배포 단계 설치 항목**을 순서대로 체크.  
미설치 항목은 `{NOT_INSTALLED}`에 추가.

---

### Phase 6. 사전준비 결과 보고

Phase 0에서 분석한 전체 항목을 기준으로 아래 형식으로 결과 보고:

```
## 사전준비 점검 결과

### 설치 확인 결과
- 체크 범위: {선택한 범위}

| 프로그램 | 설치 여부 | 자동 설치 여부 |
|---------|----------|--------------|
| {항목}  | ✅/⚠️    | ✅/➖        |

### 수동 설치 필요 항목
| 프로그램 | 설치 가이드 |
|---------|------------|
| {미설치 항목} | [로컬 개발 환경 구성](https://github.com/unicorn-plugins/npd/blob/main/resources/guides/setup/prepare.md) |

```

---

## MUST 규칙

| # | 규칙 |
|---|------|
| 1 | 스킬 시작 시 반드시 가이드를 curl로 다운로드하고 Read 도구로 읽어 분석할 것 |
| 2 | 체크 항목과 명령은 하드코딩하지 않고 다운로드한 가이드에서 추출할 것 |
| 3 | 버전 요구사항이 명시된 항목은 해당 버전을 기준으로 체크할 것 |
| 4 | OS를 먼저 감지하여 OS별 설치·확인 명령을 분기할 것 |
| 5 | PATH 추가 시 중복 여부(`grep -q`)를 확인하여 중복 추가하지 않을 것 |
| 6 | MCP 등록 시 기존 서버를 덮어쓰지 않고 미등록 서버만 추가할 것 |
| 7 | 결과 보고 후 반드시 다음 단계(`/npd:create`) 안내를 포함할 것 |
| 8 | `<!--ASK_USER-->` 발견 시 AskUserQuestion 도구를 호출할 것 (텍스트 출력 금지) |

## MUST NOT 규칙

| # | 금지 사항 |
|---|----------|
| 1 | SKILL.md에 체크 항목을 하드코딩하지 않을 것 (가이드 분석 결과를 사용) |
| 2 | 가이드에 수동 설치로 안내된 프로그램을 자동 설치하지 않을 것 |
| 3 | 미설치 항목이 있는 상태에서 이후 단계를 강제 진행하지 않을 것 |
| 4 | PATH 설정을 중복으로 추가하지 않을 것 |
| 5 | 이미 등록된 MCP 서버를 덮어쓰지 않을 것 |
