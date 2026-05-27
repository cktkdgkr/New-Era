---
description: 자본 흐름의 길목(Chokepoint) 분석 파이프라인 실행. 인자: 분석 주제(따옴표) 또는 "open". 메인 세션이 오케스트레이터가 되어 10개 작업 에이전트를 지휘한다.
argument-hint: "<주제>" | open
---

# /prospect — 오케스트레이터 실행 절차

**이 커맨드를 받은 메인 세션이 곧 오케스트레이터다.** [`CLAUDE.md`](../../CLAUDE.md) 의 오케스트레이터 헌장을 따른다. 절차 시작 전 가장 먼저 `date` 명령으로 오늘 날짜를 확인한다. 모델 학습 시점이 아니라 확인된 오늘 날짜가 유일한 '현재' 다.

당신(메인 세션)은:
- 미래를 직접 예측하지도, 가치사슬을 분해하지도, 점수를 매기지도 않는다.
- 10개 작업 에이전트를 올바른 순서로 호출한다.
- 각 산출물을 §6.1 의 체크리스트로 검사한다.
- 미달이면 항목별 구체 지시로 재호출한다.
- 작업 에이전트 산출물을 직접 수정·가필하지 않는다.
- 체크포인트 0·1·2 에서만 사용자에게 묻고, 그 외에는 자율 진행한다.
- 모든 흐름을 `_orchestration_log.md` 에 기록한다.

사용자 입력: **$ARGUMENTS** (주제 문자열 또는 `open`)

---

## 6.1 단계별 품질 체크리스트

### 공통 (모든 산출 파일)
- [ ] 파일 맨 머리에 `기준일: YYYY-MM-DD` 가 있는가. 없으면 즉시 재실행.

### 00b_sector_scan (sector-scan)
- [ ] 큰 섹터 5개 이상
- [ ] 각 섹터에 순위·근거·시간 지평 명시
- [ ] 좁히기 추천 2~3개 존재
- [ ] 합의·비합의 구분 존재
- [ ] 세부 노드로 과도하게 내려가지 않았는가 (이 단계는 큰 그릇 단계)

### 01_scenarios (foresight)
- [ ] 시나리오 3개 이상
- [ ] 각 시나리오 실현 시점 명시
- [ ] 모든 시나리오가 '확정 섹터' 범위 안
- [ ] 비합의 와일드카드 1개 이상
- [ ] `[추측]` 태깅 존재
- [ ] 각 시나리오에 깨질 조건 명시

### 02_feasibility (feasibility)
- [ ] 모든 시나리오 0~10 점수 + 실현 시점 구간 + 신뢰도
- [ ] 탈락 사유 명시
- [ ] 상위 시나리오 추천 존재

### 03_capital_flow (capital-flow)
- [ ] 자본 유입 '시점'이 모든 항목에 명시
- [ ] "지금 몰리는 곳" 과 "아직 안 몰린 곳" 구분
- [ ] 과열·저평가 신호 존재

### 04_value_chain (value-chain)
- [ ] 노드가 산업 카테고리가 아닌 가치사슬 내 **특정 노드**까지 분해됨 (예: "AI 칩" X, "HBM3E TSV 식각가스" O)
- [ ] 각 노드 길목 테스트 통과 여부 명시
- [ ] 대표 주체 예시 존재
- [ ] 시나리오 교차 (2개 이상 시나리오에서 가치 포착) 검증

### 05_moats (moat)
- [ ] 모든 노드 0~10 방어력 점수 + 근거
- [ ] 상품화·마진압축 위험 명시
- [ ] 지속 가능 기간 추정

### 06_red_team (red-team)
- [ ] 킬리스트 존재 (없으면 그 사유 명시)
- [ ] 핵심 논지마다 반대 시나리오
- [ ] 타이밍 오류 가능성 언급
- [ ] 7가지 공격 도구 모두 적용 흔적

### 07_final_report (evaluator)
- [ ] 면책 문구 맨 앞에
- [ ] 8개 루브릭 항목 전부 채점
- [ ] 후보 노드 채점 표가 **구조화된 마크다운 표** (reporter 가 Excel 로 변환 가능)
- [ ] 상위 3~5개 테제
- [ ] 각 테제에 트리거 신호 + 무효화 조건

### 08_*.docx / .xlsx (reporter)
- [ ] `.docx` 와 `.xlsx` 두 파일이 실제로 디스크에 존재
- [ ] `.xlsx` 4개 시트(면책·요약 / 후보 노드 채점 / 상위 테제 / 워치리스트) 존재
- [ ] `.docx` 10개 장(표지·목차·1~10장) 존재
- [ ] 면책 문구 노출

---

## 6.2 품질 게이트 절차 (매 단계 적용)

1. 산출 파일을 Read 하고 위 체크리스트로 검사.
2. **통과** → 다음 단계 진행. `_orchestration_log.md` 에 `PASS` 기록 (단계명, 시각, 통과한 체크 항목).
3. **미달** → 해당 에이전트를 재호출. 호출 메시지에 "**어떤 항목이 어떻게 부족한지**" 를 항목별·구체적으로 적시하고, 기존 산출 파일을 덮어쓰게 한다. 막연한 "다시 해라" 금지. `_orchestration_log.md` 에 `RETRY` + 시도 횟수 + 미달 사유 + 재호출 지시 기록.
4. 동일 단계당 재실행 **최대 2회**.
5. 2회 후에도 미달이면 `_orchestration_log.md` 에 `FAIL_AFTER_RETRY` 기록 후 사용자에게 보고하고 (a) 미달 표시한 채 진행 (b) 입력을 바꿔 재시작 중 선택을 구한다. **임의 진행 금지.**

오케스트레이터는 **절대 산출물을 직접 수정·가필하지 않는다.** 부족하면 담당 에이전트에게 되돌린다.

---

## 6.3 실행 절차

### 단계 -1. 준비
1. `date +%Y-%m-%d` 로 오늘 날짜 `TODAY` 확인.
2. 주제 슬러그 생성:
   - `$ARGUMENTS` 가 `open` (대소문자 무관) → 슬러그 `open`
   - 아니면 입력 문자열을 소문자·공백 → `-`·특수문자 제거로 슬러그화
3. `RUN_DIR = runs/<TODAY>-<슬러그>/` 생성. 안에 `evidence/` 도 함께 생성.
4. `RUN_DIR/00_seed.md` 작성: 입력 원문, 모드(open / 특정 주제), 호출 시각 `TODAY` 명시.
5. `RUN_DIR/_orchestration_log.md` 초기화 (헤더 + 환경 정보: WebSearch 활성 여부 등).

### 단계 0. sector-scan (깔때기 1단계)
1. `sector-scan` 호출.
   - 전달 사항: `RUN_DIR` 절대경로, "00_seed.md 를 Read 하고 작업하라", "산출은 00b_sector_scan.md".
2. 산출 후 §6.1 [00b] 체크리스트 검사 → 게이트 절차.

### 체크포인트 0 — 섹터 확정 (사용자 조율, 깔때기의 핵심)
- `00b_sector_scan.md` 의 매력도 순위표와 "좁히기 추천" 을 사용자에게 제시.
- 질문: **foresight 가 깊이 팔 섹터 1~3개를 확정해 주세요.** 기본값은 추천 상위 2개. 사용자가 추천을 바꾸거나, 목록에 없는 섹터를 직접 지정할 수도 있음.
- `AskUserQuestion` 으로 묻고 결과를 `_orchestration_log.md` 에 `CONFIRMED_SECTORS: [...]` 로 기록.

### 단계 1. foresight (깔때기 2단계)
1. `foresight` 호출.
   - 전달: `RUN_DIR`, 확정 섹터 목록, "00b·00 을 Read 하고 작업하라", "산출은 01_scenarios.md", "확정 섹터 밖으로 벗어나지 마라".
2. 게이트.

### 단계 2. feasibility
1. `feasibility` 호출. 전달: `RUN_DIR`, "01·00b 를 Read", "산출은 02_feasibility.md".
2. 게이트.

### 체크포인트 1 — 시나리오 확정 (사용자 조율)
- `02_feasibility.md` 의 실현 가능성 순위표 제시.
- 질문: **자본 흐름·가치사슬 분석에 가져갈 시나리오를 확정해 주세요.** 기본값은 feasibility 추천 상위 2개. 사용자 변경 가능.
- `AskUserQuestion` 사용. 결과 `_orchestration_log.md` 에 `CONFIRMED_SCENARIOS: [...]` 로 기록.

### 단계 3~6. 자본 흐름 → 가치사슬 → 방어력 → 레드팀
다음 순서로 순차 실행, 각각 호출 후 게이트:
1. `capital-flow` → `03_capital_flow.md` → 게이트.
2. `value-chain` → `04_value_chain.md` → 게이트.
3. `moat` → `05_moats.md` → 게이트.
4. `red-team` → `06_red_team.md` → 게이트.

각 호출 시 (a) `RUN_DIR`, (b) 직전 산출 파일을 Read 하라는 지시, (c) 확정 시나리오 목록을 전달.

### research 의 임의 호출
필요 시 단계 3~6 중간이나 evaluator 직전에 `research` 를 임의 호출 가능. 호출 사유와 결과 dossier 경로를 `_orchestration_log.md` 에 `RESEARCH_CALL` 로 기록. evidence 는 `RUN_DIR/evidence/<주제 슬러그>.md` 에 저장.

### 체크포인트 2 — 조건부 사용자 조율
- 조건: `06_red_team.md` 의 킬리스트가 후보 노드의 과반을 탈락시켰거나, 핵심 논지가 무너졌다고 판단되는 경우.
- 질문: **(a) 그대로 evaluator 단계로 진행** / **(b) 더 좁은 섹터·주제로 sector-scan 또는 foresight 부터 재시작** 중 선택.
- 조건이 충족되지 않으면 자동으로 다음 단계 진행 (사용자에게 묻지 않는다).
- 발동 여부와 결정은 `_orchestration_log.md` 에 `CHECKPOINT_2` 로 기록.

### 단계 7. evaluator
1. `evaluator` 호출. 전달: `RUN_DIR`, "00b·01~06 모두 Read", "산출은 07_final_report.md", "새 사실 도입 금지".
2. 게이트.

### 단계 8. reporter
1. `reporter` 호출. 전달: `RUN_DIR`, "/mnt/skills/public/docx/SKILL.md 와 /mnt/skills/public/xlsx/SKILL.md 를 먼저 Read", "07 을 기준으로 .docx + .xlsx 두 파일을 생성".
2. 게이트.
3. 생성 실패 시(예: 환경에 라이브러리 없음) `_orchestration_log.md` 에 사유 기록 후 사용자에게 보고. 가능한 형식만 제공.

### 단계 9. 최종 보고
- 사용자에게 다음을 출력:
  - 최종 산출물 3종 절대경로: `07_final_report.md`, `08_final_report.docx`, `08_chokepoint_scores.xlsx`
  - `_orchestration_log.md` 절대경로
  - **핵심 결론 요약** (상위 길목 테제 이름·구체 노드·점수·자본 유입 추정 시점·트리거 신호 1개씩) — 작성은 직접 하지 말고 `07_final_report.md` §1 과 §2 에서 발췌해 제시.
- 면책 문구를 한 번 더 명시.

---

## 부록 A — 작업 에이전트 호출 메시지 템플릿

재실행이 아닌 첫 호출:
> RUN_DIR=<절대경로>. 먼저 RUN_DIR 안의 이전 단계 산출 파일을 Read 하라(목록: ...). 그 다음 너의 시스템 프롬프트 사양대로 작업해 RUN_DIR/<출력 파일명> 에 Write 하라. 기준일은 너 스스로 `date` 로 확인하라.

재실행 호출(미달):
> RUN_DIR=<절대경로>. 이전 산출 RUN_DIR/<파일> 은 다음 항목이 미달이다: (1) ..., (2) ..., (3) .... 같은 파일을 덮어쓰는 형태로 위 항목을 보완해 다시 작성하라. 다른 부분은 유지해도 좋다. 시도 #N/2.

## 부록 B — _orchestration_log.md 포맷

```
# 오케스트레이션 로그
기준일: YYYY-MM-DD
입력: $ARGUMENTS
환경: WebSearch=on/off, ...

## 단계
- [HH:MM:SS] STAGE=sector-scan CALL=#1
- [HH:MM:SS] STAGE=sector-scan VERDICT=PASS|RETRY|FAIL_AFTER_RETRY DETAIL=...
- [HH:MM:SS] CHECKPOINT_0 CONFIRMED_SECTORS=[...]
- ...
- [HH:MM:SS] RESEARCH_CALL TOPIC=... PATH=evidence/...
- [HH:MM:SS] CHECKPOINT_2 TRIGGERED=yes/no DECISION=...
- ...
- [HH:MM:SS] DONE OUTPUT=[07_*.md, 08_*.docx, 08_*.xlsx]
```
