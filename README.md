# 자본 흐름의 길목(Chokepoint) 하니스

> "자본을 효율적으로 축적하려면 내 노력보다 자본이 흐르는 곳에서 길목을 지켜야 한다."
> 금광 시대엔 곡괭이·청바지 장수가, AI 시대엔 반도체·소재 기업이 돈을 벌었다.
> 이 하니스는 **지금 시점에서 미래의 곡괭이가 어디인지** 를 찾기 위한 다중 에이전트 도구다.

## 무엇을 하는가

`/prospect <주제>` 한 줄로 다음 9단계 분석을 수행한다.

1. **섹터 스캔** — 자본이 크게 흐를 큰 그릇(거시 테마) 5~8개를 펼친다.
2. **체크포인트 0** — 사용자와 함께 깊이 팔 섹터 1~3개를 확정한다.
3. **미래 시나리오** — 확정 섹터 안에서 3·7·15년 지평 시나리오 3~5개를 만든다.
4. **실현 가능성·타이밍** — 공상은 죽이고 현실적인 것엔 날짜를 붙인다.
5. **자본 흐름 지도** — VC·PE·국부펀드·정부·기업 capex 가 실제로 어디로 흐르는지.
6. **가치사슬 분해** — 원료→소재→장비→부품→인프라→플랫폼→응용→서비스.
7. **방어력(해자) 검증** — 그 노드가 돈을 *지킬* 수 있는가.
8. **레드팀** — 거품·하이프·합의에 이미 반영된 가격을 공격한다.
9. **최종 평가 + 보고서 생성** — Markdown·Word·Excel 3종.

## 아키텍처

- **오케스트레이터 = 메인 세션** — `/prospect` 를 받는 최상위 Claude 자신이 오케스트레이터다. 별도 에이전트가 아니다. 그 정체성·원칙은 [`CLAUDE.md`](./CLAUDE.md) 에, 실행 절차는 [`.claude/commands/prospect.md`](./.claude/commands/prospect.md) 에 있다.
- **작업 에이전트 10개** — `.claude/agents/` 의 서브에이전트. 각자 자기 분석만 하고 메인 세션에 보고한다.
  - `sector-scan` — 큰 섹터를 **넓게** 펼친다 (깔때기 1단계)
  - `foresight` — 확정 섹터 안에서 **세부 시나리오** (깔때기 2단계)
  - `feasibility` — 실현 가능성·시점 판정
  - `capital-flow` — 자본이 어디로 흐르는지
  - `value-chain` — 가치사슬 분해 + 길목 노드 발굴
  - `moat` — 방어력 검증
  - `red-team` — 악마의 변호인
  - `research` — 공유 근거 수집(필요 시 호출)
  - `evaluator` — 최종 점수화·순위화
  - `reporter` — Word·Excel 보고서 생성

## 미래 탐색 — 2단계 깔때기

```
사용자 입력
   │
   ▼
[sector-scan]  ────  큰 섹터 5~8개 (넓게)
   │
   ▼
[체크포인트 0] ────  사용자가 1~3개 섹터 확정
   │
   ▼
[foresight]    ────  확정 섹터 안에서 세부 시나리오 3~5개 (좁게)
```

`/prospect open` 처럼 좁은 주제를 미리 정하지 않아도 하니스가 스스로 큰 → 작은으로 좁힌다.

## 모든 에이전트는 작업 전 오늘 날짜를 확인한다

오케스트레이터를 포함한 모든 에이전트는 분석을 시작하기 전에 가장 먼저 `date` 명령(없으면 WebSearch)으로 오늘 날짜를 확인하고, 모든 산출 파일 맨 머리에 "기준일: YYYY-MM-DD"를 명시한다. **모델 학습 시점이 아니라 확인된 오늘 날짜가 유일한 '현재'다.** "실현 시점", "자본 유입 시점" 등 모든 타이밍 판단의 기준점이 된다.

## 사용법

```
/prospect "휴머노이드 로봇"
/prospect "전력망 재구성과 AI 데이터센터"
/prospect open
```

- 주제를 구체적으로 줄 수도, 그냥 `open` 으로 시작할 수도 있다.
- 실행 결과는 `runs/<YYYY-MM-DD>-<슬러그>/` 에 모인다.
- 사용자에게 묻는 분기점은 **체크포인트 0(섹터 확정)**, **1(시나리오 확정)**, **2(레드팀 후 조건부)** 세 곳뿐. 나머지는 자율 진행.

## 파이프라인 다이어그램

```
00_seed.md
  ↓                                     [sector-scan]
00b_sector_scan.md ─── 체크포인트 0 ──→ [foresight]
                                          ↓
                                       01_scenarios.md
                                          ↓             [feasibility]
                                       02_feasibility.md ─── 체크포인트 1
                                          ↓             [capital-flow]
                                       03_capital_flow.md
                                          ↓             [value-chain]
                                       04_value_chain.md
                                          ↓             [moat]
                                       05_moats.md
                                          ↓             [red-team]
                                       06_red_team.md ─── 체크포인트 2(조건부)
                                          ↓             [evaluator]
                                       07_final_report.md
                                          ↓             [reporter]
                                       08_final_report.docx
                                       08_chokepoint_scores.xlsx
```

보조: `evidence/<주제>.md`(research), `_orchestration_log.md`(흐름 로그).

## 품질 게이트 · 재실행 메커니즘

매 단계 산출물은 단계별 체크리스트(prospect.md §6.1)로 검사된다.

- **PASS** → 다음 단계 진행.
- **RETRY** → 미달 항목을 항목별로 구체적으로 지적해 동일 에이전트 재호출(같은 파일 덮어쓰기). 동일 단계당 최대 2회.
- **FAIL_AFTER_RETRY** → 사용자에게 보고하고 (a) 미달 표시한 채 진행 (b) 입력을 바꿔 재시작 중 선택을 구한다. 임의 진행 금지.

모든 판정은 `_orchestration_log.md` 에 시각·재호출 지시 내용과 함께 기록된다.

## 최종 산출물 3종

| 파일 | 형식 | 용도 |
|------|------|------|
| `07_final_report.md` | Markdown | 분석가용 원본. evaluator 산출. |
| `08_final_report.docx` | Word | 10장 구성의 서술형 종합 보고서. 표지·목차·면책 포함. |
| `08_chokepoint_scores.xlsx` | Excel | 4시트(면책·요약 / 후보 노드 채점 / 상위 테제 / 워치리스트). 정량·실행 데이터. |

## 환경 권장 사항

- **WebSearch / WebFetch 활성화 권장** — 없으면 모델 지식만으로 동작하나 자본 흐름·VC 펀딩·정책 등 시점 의존 정보의 품질이 크게 떨어진다. 비활성 상태로 실행한 경우 `_orchestration_log.md` 에 그 사실이 기록된다.
- **Word·Excel 생성** — `reporter` 단계에서 `.docx`·`.xlsx` 를 만들려면 환경에 `/mnt/skills/public/docx/SKILL.md`·`/mnt/skills/public/xlsx/SKILL.md` 가 있고, 그 스킬이 의존하는 라이브러리(`python-docx`, `openpyxl` 등)가 설치되어 있어야 한다. 환경이 갖춰지지 않은 경우 reporter 가 안내 후 가능한 형식만 생성한다.
- **사용자의 책임** — 이 하니스의 출력은 추가 검증이 필요한 **리서치 가설**이며 **투자 자문이 아니다.**

## 파일 구조

```
.
├── CLAUDE.md                     # 오케스트레이터 헌장 + 공통 원칙
├── README.md                     # 이 문서
├── .claude/
│   ├── agents/                   # 10개 작업 에이전트
│   │   ├── sector-scan.md
│   │   ├── foresight.md
│   │   ├── feasibility.md
│   │   ├── capital-flow.md
│   │   ├── value-chain.md
│   │   ├── moat.md
│   │   ├── red-team.md
│   │   ├── research.md
│   │   ├── evaluator.md
│   │   └── reporter.md
│   └── commands/
│       └── prospect.md           # /prospect 슬래시 커맨드 = 오케스트레이터 실행 절차
└── runs/                         # 실행 시 runs/<날짜>-<슬러그>/ 생성
```
