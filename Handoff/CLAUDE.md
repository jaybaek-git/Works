# PA 벤치 · DPD 프로젝트 — Claude 인수인계 (CLAUDE.md)

이 파일은 이 폴더에서 일하는 Claude 가 맨 처음 읽는 안내서입니다.
2026-11 에 개인 계정 Claude 와 진행한 작업을 회사 계정 (벤치 PC) 으로 넘기며 정리했습니다.
기준 버전: **PA DPD Tester v0.2.3.154 · PA Bench Monitor v2.5.187 (Site2 v1.4.102)**

---

## 0. 한 줄 목표

3.4–3.8 GHz MMU 용 **3-way GaN Doherty PA** (피킹 디퍼런셜 · 동작은 2-way 와 같음) 에
**400 MHz 2CC 신호** 로 DPD 를 걸어, **기지국에 실제로 쓸 수 있는 GMP 계열 모델** 로
ACLR 을 최대한 낮추는 것. 신경망 (torch) 은 기지국에서 못 쓰므로 **"모델이 어디까지 갈 수 있나" 를 재는 기준선** 으로만 씀.

### 🎯 최우선 목표 (모든 판단의 기준)
| 무엇 | 조건 | 목표 |
|---|---|---|
| **기지국용 알고리즘 (GMP · DVR · 확장 모델)** | Pout 42 dBm · 400 MHz 2CC · ACLR worst | **−51 dBc 이하 = 쓸 수 있음 (최우선)** |
| ILC (상한) | 같은 조건 | **−55 dBc 보다 아래 · −60 이면 더 좋음** |

- ILC 는 **학습 방법이 아니라 '이 샘플 · 측정 조건이 어디까지 되나' 를 재는 지표**. ILC 가 −51 보다 안 좋으면 어떤 모델도 거기까지 어려움 → 측정 조건 (I/Q 평균 · 감쇠 · 고치는 대역) 을 손보라는 신호. 모델 학습은 ILC 정답을 꼭 쓰지 않아도 됨 (§8-6 길 A 직접 학습 · 길 C 새 방법).
- 지휘는 실험을 고를 때 **"−51 에 가장 가까이 가게 할 실험"** 을 먼저. 보고서 맨 위에 목표 대비 남은 dB (예: GMP −40 → 11 dB 남음).

현재 (Pout 42 dBm): ILC −49 dBc · torch 신경망 −46 · **GMP 계열 −38 ~ −40 → 목표 −51 까지 11 ~ 13 dB · ILC 목표 −55 까지 6 dB**

---

## 1. 절대 규칙 (어기면 안 됨)

1. **`psu_panel.py` 는 절대 수정 금지.** 파워서플라이 ON/OFF (바이어스) 순서 알고리즘. 병합 · 리팩터링 때도 그대로 둘 것.
   발행 전 항상 원본과 같은지 확인 (`diff -q`).
2. **바이어스 값을 자동으로 바꾸지 말 것.** 바이어스 순서는 매우 중요. 켜기 · 끄기는 기존 버튼 (`power_on` · `power_off`) 을 그대로 부르는 것만 허용.
3. **측정 중 이상 (Vdd 전류 등) → 먼저 SG RF OFF** (+ 드라이버 앰프 OFF). 바이어스는 그 뒤에도 전류가 이상할 때만 긴급 종료.
4. **기본값 = 예전 동작.** 새 기능은 기본으로 꺼 두거나, 켜도 예전과 똑같이 동작하게.
5. **UI 를 바꾸기 전엔 반드시 목업 (그림) 을 먼저 보여주고 확인받기.** 텍스트 설명 말고 시각 목업.
6. **카드 옵션에 넣을 값을 안내할 때도 항상 목업 (그림) 으로.**
7. 팝업은 화면 가운데.
8. 더 좋은 방법이 있으면 **선택지 몇 개로 먼저 물어보고**, 답할 때마다 반영.
9. 사용자에게는 **한국어로, 짧게** (휴대폰으로 읽는 경우 많음).
10. 사용자 이메일 등 개인정보를 외부 요청에 넣지 말 것.

## 2. 버전 · 발행 규칙

- 모든 결과물은 버전을 붙여 발행. 큰 변경이 아니면 끝에 `.N` 을 하나씩 올림 (예 v0.2.3.149 → v0.2.3.150).
- 세 프로그램을 같이 올림: DPD Tester (`v0.2.3.x`), Bench (`v2.5.x`, PA Analyzer 연동판 AnalyzerLink 만), Site2 (`v1.4.x`).
  DPD 코드가 바뀌면 `PA-DPD-Tester/*.py` 를 `PA-Bench-Monitor/` 에도 똑같이 복사 (Bench 의 DPD 탭이 같은 코드를 씀).
- 버전 문자열 위치: `pa_dpd_tester.py` `VERSION`, `pa_bench_monitor.py` `VERSION`, `site2.py` `SITE_VER`, 각 `build_exe*.bat`, README 맨 위.
- README 맨 위에 `■ vX 변경점 - …` 블록 추가 (Bench · Site2 README 는 "DPD Tester vX 와 같음" 한 줄).
- 사용자는 **빌드하지 않고 `run_without_build*.bat` 로 실행**.
- (개인 계정 때) zip 을 GitHub `jaybaek-git/Works` 의 `PA-DPD-Tester/` · `PA-Bench-Monitor/` · `PA-Bench-Monitor-Site2/` 에 올리고 issue #1 에 댓글. PPT 는 `PPT/`.
  → 회사 환경에선 회사 규정에 맞는 곳으로 (사용자에게 물어볼 것).

## 3. 벤치 · 장비

| 장비 | 역할 | 연결 |
|---|---|---|
| R&S SMW200A | 신호발생기 (DPD 파형 재생) | LAN/VISA |
| R&S FSW | 스펙트럼 · I/Q 캡처 (±393 MHz), ACLR | LAN/VISA |
| R&S RTO2044 | 펄스 전류 (오실로스코프) | LAN |
| Keysight E36155A | Vdd 48 V | GPIB |
| Keysight N6705C | 게이트: 노랑 = DRV, 초록 = CA, 파랑 = PK (음전압) | GPIB |
| R&S NRP50S | 파워미터 (Mod. 측정 Pout) | USB |

- 신호: **400 MHz 2CC**, 반송파 ±133 MHz × 100 MHz, **983.04 MSps**. (1CC 100 MHz NR 도 씀)
- 측정 설정: FSW I/Q 평균 12, RBW 50 kHz. 반송파 사이 바닥 ≈ −100 dBm vs 반송파 −34 dBm (≈ −66 dBc) → 측정 바닥은 문제 아님.
- 샘플 종류 4 가지 (Bench ① 샘플 종류) · 종류마다 바이어스 · 트리거 · 4 카드 값 등을 따로 기억.
- **샘플별 FSW 설정 (사용자 값 · 2 카드)**: 3B0 = Ref −3 dBm · Ref 여유 3 dB · 감쇠 10 dB / 3X0 = Ref −5 dBm · Ref 여유 3 dB · 감쇠 7 dB.
  감쇠는 이 값 아래로 내리지 말 것 (FSW 입력 보호 · 9 카드 'FSW 최소 감쇠' 가 막음).
- **신호는 고정**: 400 MHz 2CC DPD 만, 사용자의 신호 파일 (.wv) 로만 시험. 신호 · 주파수 · 손실 · 장비 주소는 자동 실험이 못 바꿈 (잠김).
  예외: 1 신호 '1000 MSps 넘으면 절반으로' (`file_half400`) 는 바꿔 볼 수 있음 — 기본 켬 (983.04 MSps). 결과가 잘 나오면 끄고 (1966.08 MSps) 다시 찾아볼 것.
  끄면 신호가 바뀌므로 **ILC 부터 다시** (이전 결과와 직접 비교 X).

## 4. 코드 지도

```
PA-DPD-Tester/            (단독 DPD 프로그램 · Bench 의 DPD 탭도 같은 파일)
  pa_dpd_tester.py        UI (tk). 카드 1~8, 로그, 그래프 창, 기록, 엑셀 저장. VERSION
  dpd_engine.py           측정 · 학습 흐름 (Engine). run / run_ilc / run_ilc_model / run_more,
                          ACLR 계산, 진단 코드 (diag_summary), SG 파워 맞추기 (_up_cap), 2-tone · 백오프 스윕
  dpd_core.py             모델 · 수학. DPDModel(GMP) · KneeGMPModel · BandGMPModel(2D) · HybridGMPModel ·
                          SparseGMPModel(GMP 항 고르기, OMP) · DVRModel · NNModel(numpy) · welch_psd 등
  dpd_torch.py            TorchNNModel (torch 신경망, ACLR 가중 학습 _fit_fw)
  instruments.py          SMW · FSW 제어 (+ demo 장비)
PA-Bench-Monitor/
  pa_bench_monitor.py     메인 (Pulse · Mod. · DPD 탭, 파워 스윕, 이상 처리, 샘플 종류). VERSION
  psu_panel.py            ★ 파워서플라이 순서 (수정 금지)
  rto_current_meter.py    RTO 전류 읽기 (트리거 자동 추적 · '예전 추적 방식' 기본 켬)
  excel_export.py · analyzer_link.py · analyzer_child.py · pa_analyzer/ (PA Analyzer 연동)
  site2.py                Site2 (E3632A 2 대 직렬 24 V+24 V, E3631A 게이트) 변형
  ext_psu.py · visa_reconnect.py · bench_link.py
```

설정 · 로그 위치 (Windows): `%APPDATA%\PADPDTester\` — `logs\진단요약_YYYYMMDD.txt` (실행마다 진단 요약 + 결과 코드), 기록 · 프리셋.

### DPD 탭 카드 (순서)
1 신호 · 2 RF (FSW Ref/감쇠, I/Q 평균) · 3 파워 맞추기 (Pout 목표, SG 한 번에 올리는 양 `sg_up_max`) ·
**4 DPD 모델** (모델, K · M · L, 반복, μ, λ, 학습 샘플 수, 신경망 크기, ACLR 가중 학습, 나쁜 채널 자동, 고를 항 수 …
— **모델마다 · 샘플 종류마다 값을 따로 기억**) · 5 실행 (ILC, ILC 결과로 모델 만들기, 이어서 반복, 옵션) ·
6 결과 · 7 마무리 · 장비 (SMW · FSW 각각 연결/해제) · 8 조건 스윕 (투톤 간격, 백오프)

## 5. DPD 흐름 · 결과 코드

1. **ILC** (모델 없이 샘플을 직접 고침, `uₖ₊₁ = uₖ + μ(x − yₖ/G)`) → 이 PA · 조건의 상한.
2. **ILC 결과로 모델 만들기**: 모델이 `x → u*(ILC 입력)` 를 흉내 내게 학습 → 측정 → 다듬기 (ILA).
   로그 📌 줄: `주파수별 학습 오차 · 최종 모델` (코드 B), `모델 출력 vs ILC 입력 차이 (계산): 인접 채널 ○○ dBc`.
3. **이어서 반복**: ILA 로 몇 번 더.

진단 요약 코드 (예 `v149 R98 W79 O-25 D-46 K0 T-36 B36/41/33/26 …`):

| 코드 | 뜻 |
|---|---|
| D | DPD 뒤 ACLR 가장 나쁜 값 (dBc) — **최종 성적** |
| T | 학습 오차 (dB). 지금 −36 바닥은 측정 반복성 (모델 상관없음) |
| B | ILC 정답과 모델 차이를 채널 안 / **인접** / 그다음 / 그 바깥 으로 (dBc, 부호 뺌). **인접 칸이 D 를 정함** |
| S | 진폭 순위 구간별 오차 (S0-50 = 크기 하위 50 % 샘플) |
| N | 꺾임점 (%) · Y 중간 메모리 이득 · Z 넓힌 탐색 (best:검증/현재) · U 고배속 배수 · X 설명된 보정 비율 · K 최고값 걸린 샘플 수 |

## 6. 지금까지 결과 (자세한 건 `docs/DPD_알고리즘_정리_v0.2.3.149.pptx`)

Pout 40 dBm (11/25 까지): ILC −54 · 신경망 −44 · GMP+반송파별 −42 · 반송파별 GMP(2D) −41 · GMP+꺾임 −40 · GMP K7 M8 L6 −38
Pout 42 dBm (11/26): ILC −49 · 신경망+ACLR 가중 −46 · 신경망 M16 −45 · 신경망 기본 −43 · GMP+반송파별 −40 · GMP 항 고르기 −38 (가중 0 은 −37)

**핵심 발견**: B 코드 비교 — 신경망 `−35.5 / −41.3 / −32.6 / −26.2` vs GMP 항 고르기 `−32 / −35.2 / −31.4 / −27.6`.
**인접 칸만 6 dB 차이 = D 차이와 같음** → GMP 계열이 반송파 바로 바깥 어깨 왜곡을 표현할 항이 모자람.

### 효과 있었던 것
반송파를 나눠 보는 항 (2D, +2~4 dB) · 신경망 M 8→16 + 샘플 4 배 (+2) · ACLR 가중 학습 (신경망 +1, GMP 인접 오차 −29→−35) · SG 상승 제한 (과입력 해결)

### 확인해서 아니었던 것 (다시 시험하지 말 것)
I/Q 불균형 · 클리핑/최고값 제한 · 측정 잡음 · FSW 바닥 · 디지털 접힘 (고배속 계산) · 꺾임점 위치 · 중간 메모리 16–64 ns ·
K · M · L 크기 (1 dB 이내) · 반복 횟수 · 신경망 크기 · 앞쪽 샘플 · M 24 · ACLR 가중 30 (10 과 같음) · 반송파별 교차 지연 항 (4 개만 골라짐)

## 7. 다음 순서

1. **볼테라 교차 항** (v0.2.3.149, GMP 항 고르기 후보에 `x(n−a)x(n−b)x*(n−c)|x|^j`) — ACLR 가중 10 으로. 인접 B 가 −35 → −38 이하면 방향 맞음.
2. **차이 쪼개기**: 신경망 입력을 하나씩 빼 보며 (위상 정규화, |x|^k, 시점 범위) 어떤 성분이 6 dB 를 만드는지 → 그 꼴을 GMP 후보로.
3. **동작점**: 8 번 카드 백오프 · 투톤 간격 스윕 → 메모리 (바이어스 라인) 크기.
4. 하드웨어 (바이어스 라인 캡) 는 **마지막 수단**.

## 8. 자동 실험 · 오케스트레이션 (주말 무인 운전)

### 8-1. 프로그램 쪽 (DPD 탭 9 카드 '자동 실험 (큐)', v0.2.3.150 ~ · 확장 모델 v0.2.3.151 ~)
큐 폴더: `%APPDATA%\PADPDTester\queue\`  (Windows: `C:\Users\<사용자>\AppData\Roaming\PADPDTester\queue`)

| 파일 · 폴더 | 뜻 |
|---|---|
| `*.json` (큐 폴더 바로 아래) | 대기 중 실험. **이름 순서** 로 실행 → 이름 앞에 번호 (`010_…`, `020_…`) |
| `running\` · `done\` · `failed\` | 실행 중 · 성공 · 실패 (파일이 옮겨짐) |
| `results.jsonl` | 단계마다 한 줄: time, file, note, step, do, status (ok · timeout · pout_cap · bias_off · error · skip · not_started · fsw_fail · ext_selftest_fail), **D, B, T**, code, **diag (진단 요약 전문)**, pout, sg, aclr_l/u, model, ext_model, set (4 카드 값 전부), fsw_att, iq_avg, **run_id · run_file (측정 전체 데이터)** |
| `status.json` | 지금 상태 (2 초마다): on, state (run · empty · wait_bias · retry_on), waiting, ok, fail, current, last |
| `결과_요약.xlsx` | results.jsonl 정리: D 좋은 순 · 전체 시간 순 · 설정 |
| `xlsx\<실험>_D-46.xlsx` | 실험마다 항상 (측정 표 · 바이어스 · 설정) |
| `shots\<실험>_D-46_시각.png` | FSW 화면 — **D 상위 3 개만** 남김 (더 나쁜 건 그림 지우고 내용만 results 에) |
| `STOP` (빈 파일) | 지금 실험 끝나고 큐 멈춤 |

실험 파일 형식:
```json
{
  "note": "GMP 항 고르기 + 볼테라 교차, ACLR 가중 10",
  "set":  {"model": "omp", "K": 9, "M": 24, "L": 12, "omp_n": 600, "nn_aclr_w": 10, "aclr_auto": true},
  "steps": ["ilcm", {"more": 3}],
  "max_min": 120
}
```
- `set` 키 = 프로그램 설정 이름 (`self.v` 키). 모델 이름: `gmp kgmp bgmp hgmp omp sgmp dvr nn pnn tnn`. 4 카드 주요 키:
  `K M L iters mu lam n_fit fit_us fit_segs peak_lim_db nn_hidden nn_epochs nn_lead nn_aclr_w aclr_auto omp_n knee_pct knee_n odd_only os_model`.
  모르는 키는 로그에 경고 후 건너뜀. `model` 을 먼저 넣고 (모델별 기억이 채워짐) 나머지가 덮어씀.
- `steps`: `ilc` (ILC 상한 측정 · 오래 걸림), `{"ilc_more": N}` (**이어서 ILC** — 가장 좋은 ILC 입력에서 N 번 더 · 메모리에 없으면 저장된 최근 ILC 를 측정해 출발 · 끝나면 자동 저장, v0.2.3.154 ~), `ilcm` (ILC 결과로 모델 만들기 — 같은 세션의 ILC 또는 저장된 최근 ILC 사용), `{"more": N}` (이어서 N 번), `dpd` (ILC 없이 지금 4 카드 모델 · 학습 방식으로 직접 DPD), `off`.
- **스윕** (v0.2.3.154 ~): `"sweep": {"K": [5, 7, 9], "M": [4, 8]}` → 프로그램이 조합마다 실험 파일 `<이름>_s01.json …` 로 나눠 큐에 넣음 (최대 64 개 · 원래 파일은 done). `"group"` 은 결과 줄에 남아 묶어서 비교 (안 적으면 파일 이름).
  예: `queue_examples/011_ilc_sweep.json` (ILC 조건), `060_direct_sweep_gmp.json` (ILC 없이 GMP 직접 학습).
- ILC · 학습 방식 키: `method` (`ila` 간접 · `dla` 직접), `w_mode` (`time` 시간 · `focus` 주파수·집중 · `aclr` 주파수·ACLR), `focus_mhz` `focus_w` `ilc_iters` `ilc_gain` `ilc_band_mhz` `ilc_time` `iq_avg` `fsw_att_db` (최소 감쇠 아래 막힘).
- 안전: Pout 상한 42 / 43 dBm (3 카드 목표 Pout 도 상한으로 깎음 · 측정 Pout 이 상한 +0.3 넘으면 중지 · RF OFF) · 실험당 최대 시간 (기본 120 분, 파일 `max_min` 우선).
- 바이어스가 꺼지면 (Bench 의 이상 처리 등): SG RF OFF → 기다림 (기본 10 분) → **기존 POWER SUPPLY ON 순서** 로 다시 켬 (값 안 바꿈) → Idq (목표 CA+DRV ±15 %) 확인 → 그 실험을 처음부터. 벗어나거나 3 번 실패 → 바이어스 OFF 유지 · 큐 끝.
- 큐가 비면 SG RF OFF 로 기다림 → 새 파일이 들어오면 바로 이어감 · 빈 채로 3 시간 = 큐 끝 (RF OFF · 바이어스 OFF 선택).
- 자동 실험 중 프로그램의 확인 창은 뜨지 않고 로그에 남김 ('아니요' 로 진행).
- `set` 에 `"ext_model": "<이름>"` 을 넣으면 4 카드 모델 대신 **확장 모델** (§8-4) 을 씀. 안 넣으면 4 카드 모델.
  결과 줄에 `ext_model` · `ext_ver` · `ext_test` (자동 검사 결과) 가 같이 남음. 자동 검사 실패 = status `ext_selftest_fail` (장비 안 씀).

### 8-2. Claude 쪽 (지휘 · 보조 에이전트)
- 시작: 사용자가 9 카드 [▶ 큐 시작] 을 누른 뒤, Claude Code 에 `WEEKEND_RUN.md` 내용을 붙여 넣음.
- 보조 에이전트 정의: `.claude/agents/dpd-analyst.md` (결과 분석), `dpd-modeler.md` (새 모델 · 확장 모델 · 시뮬레이션 검증), `dpd-reporter.md` (보고서), `dpd-equipment.md` (장비 · FSW 상태 · 감쇠 최적 판단), `dpd-narrator.md` (해설: 지금 무엇을 · 왜 · 이전 기록 → `queue\narration.txt` (9 카드에 표시) · `queue\지금_상황.md`).
- 실험이 바뀔 때마다 · 결정을 내릴 때마다 dpd-narrator 를 불러 설명을 갱신 (최소 30 분마다). 결정 이유는 `queue\plan.md` 에 짧게 남겨 해설자가 읽게.
- **주기: 일이 생기면 바로.** `python tools\wait_event.py` 가 실험 · 단계가 끝나거나 큐 상태가 바뀌거나 대기 실험이 2 개 아래로 떨어질 때까지 (최대 9 분) 기다렸다가 무엇이 바뀌었는지 알려줌 → 판단 · 행동 → 다시 wait_event. 아무 일 없어도 9 분마다 한 번 깨어남.
- 실험 설계 원칙: 한 번에 한 가지만 바꿈 · 같은 ILC (같은 Pout · 같은 날) 끼리 비교 · 좋은 설정은 이어서 반복으로 확인 · 같은 걸 3 번 넘게 반복하지 않음.
- 새 모델 코드: `dpd_core.py` 에 추가 → **가짜 PA (numpy) 로 먼저 검증** → `python -m py_compile` → 버전 올림 (§2) → 프로그램은 **사용자가 다시 켜야** 새 코드가 들어감 (주말엔 큐 파일로 쓸 수 있는 기존 모델 · 설정 조합 위주로).
- 프로그램 코드를 고치는 건 주말 무인 중엔 하지 말 것 (실행 중인 프로그램엔 안 들어가고 위험). 아이디어 · 패치는 `proposals\` 에 써 두고 월요일에 사용자 확인.

### 8-3. 보고서 (폴더 4 개 · `queue\보고서\`)
| 폴더 | 언제 | 내용 |
|---|---|---|
| `실험별\` | 실험 하나 끝날 때마다 | `<시각>_<실험>.md` — 무엇을 · 왜 · 결과 (D · B · 스펙트럼 · AM-AM 특징) · 판정 한 줄 · 다음 |
| `알고리즘별\` | 그 알고리즘 실험이 끝날 때마다 갱신 | `<ILC · 신경망 · GMP · OMP · DVR · 확장모델이름>.md` — 해 본 설정 · 결과 표 · 최고 · 배운 것 · 남은 것 (누적) |
| `일일\` | 매일 밤 (자정 전) | `<날짜>.md` (+ xlsx) — 그날 한 일 · 결과 · 상위 3 (FSW 사진) · 안전 이벤트 · 내일 계획 |
| `주간\` | 일요일 밤 (월요일 아침 보기용) | `<금~일 날짜>.md` (+ xlsx · 가능하면 pptx) — 한 줄 결론 · 금 · 토 · 일 각각 한 일 · 상위 3 · 알고리즘별 요약 · 전략 (§8-6) 두 길의 지금 위치 · **사용자에게 드릴 다음 선택지 2 ~ 3 개** |

- 주말 동안의 '다음 선택지' 는 **지휘가 스스로 골라서 바로 실험** (가장 기대값 높은 것부터). 주간 보고서의 선택지만 사용자 몫.
- `queue\shots\` 의 FSW 사진 (상위 3) 을 일일 · 주간 보고서에 넣음.

### 8-4. 확장 모델 (Claude 가 직접 만드는 새 DPD 모델)
- 위치: `%APPDATA%\PADPDTester\models_ext\<이름>.py` (이름 = 영문 · 숫자 · `_`). 9 카드 '확장 모델' 칸 옆 📂 로 열림. 예시: `models_ext\example_omp_xlag.py`.
- 프로그램이 **실행할 때마다 파일을 새로 읽음** → 고친 내용이 다음 실험부터 바로 반영 (프로그램 재시작 필요 없음).
- 파일 규칙:
  - `make_model(p, eng, C)` → 모델 객체. `p` = 설정 dict (K · M · L · lam · omp_n · nn_aclr_w …), `C` = `dpd_core` 모듈 (기존 모델 클래스 상속 · 재사용), `eng` = 엔진 (`eng.sig.x/fs`, `eng.aclr_bands()`, `eng.aclr_band_w(배수)`, `eng.log(...)`).
  - 모델: `fit(u, d, n_fit=40000, rng=None, **kw) -> (w, 학습 오차 dB)` · `apply(x, w=None) -> 같은 길이 복소 배열` (w=None 이면 학습한 self.w).
    `_basis_seg(u, s0, n)` (계수 선형 모델) 이 있으면 ILC→모델 의 ACLR 가중 학습 · 진단이 자동으로 붙음 → **GMP 계열은 `C.DPDModel` · `C.SparseGMPModel` 을 상속하는 게 가장 쉬움**.
  - `VERSION = "..."` (고칠 때마다 올림 · 결과 줄에 남음), `DESC = "한 줄 설명"`.
  - 계산만: 장비 · 파일 · 네트워크 · 바이어스 접근 금지. 학습은 수 분 안 (자동 검사 제한 240 초).
- 자동 검사 (실험 전, 장비 안 씀): 2CC 가짜 신호 + 가짜 PA 로 학습 · 적용 → 숫자 (NaN) · 출력 크기 (0.3 ~ 3 배) · 시간 확인. 실패하면 그 실험은 건너뜀.
- 장비 쪽 안전 (Pout 상한 · 최고값 제한 · SG 최대 · 바이어스) 은 프로그램이 그대로 지킴.

### 8-5. 측정 조건도 바꿔가며 찾기 (사람이 없으니 다 해 봐도 됨)
- 바꿔 볼 수 있는 측정 · 학습 조건 (실험 파일 `set`):
  - `iq_avg` (I/Q 평균, 2 ~ 32) · `aclr_src` (`fsw_end` = 반복 중 I/Q · 끝만 FSW 화면 / `fsw` = 매번 FSW 화면 / `iq` = I/Q 만 — 사람이 안 보니 `iq` 로 빠르게 해도 됨, 최종 확인만 `fsw_end`)
  - `w_mode` (`time` 시간 영역 · `focus` 주파수 집중 · `aclr` 주파수 ACLR 채널) · `ilc_time` (ILC 를 시간 영역으로) · `ilc_iters` · `ilc_gain` · `ilc_band_mhz`
  - `b_target` (목표 Pout, 상한 42/43 안) · `fsw_att_db` (최소 감쇠 이상) · `fsw_ref_dbm` · `ref_margin_db` (2 이상) · `file_half400`
- **ILC 조건 찾기**: ILC 조건 (반복 · 이득 · 대역 · 시간/주파수 · I/Q 평균 · 감쇠) 을 바꿔 `ilc` 단계로 상한 D 를 비교 → 가장 좋은 조건으로 **ILC 를 다시 한 번** 뽑은 뒤 그다음 모델 실험 (`ilcm` 은 가장 최근 ILC 를 씀).
- FSW 단계 (9 카드 'Claude FSW 명령 허용' 켜져 있을 때 · SG RF OFF 뒤에만):
  - `"fsw_opt"` 또는 `{"fsw_opt": [10, 12, 14, 16]}` — 가장 좋은 DPD 파형 (마지막 DPD · ILC 결과) 을 보내며 감쇠마다 ACLR → 과입력 없이 가장 깨끗한 감쇠 (0.3 dB 안 평평하면 가운데) 를 2 카드에 넣음. 감쇠 목록 생략 = 최소 감쇠부터 2 dB 간격 6 개. DPD OFF 에선 PA 왜곡이 커서 차이가 안 보이니 DPD 결과가 있을 때 할 것.
  - `"fsw_check"` — 감쇠 · 주파수 · Ref · 프리앰프 · 과입력 상태 · 오류 읽어 결과 줄에.
  - `"fsw_reset"` — 사용자가 마지막으로 불러온 FSW 설정 (.dfl) 을 다시 보내고 프로그램 기억을 비움.
  - `{"fsw": ["SENS:FREQ:CENT 3.6GHz", "INP:ATT?"]}` — SCPI 명령 (? 는 읽어서 로그). 막음: 프리앰프 ON · 최소 감쇠 아래 · SHUT · MMEM:DEL 등. 끝나면 감쇠가 최소 아래면 자동으로 올림.
  - `"fsw_shot"` — 지금 FSW 화면 저장 (shots\).
- 장비 담당 에이전트 `dpd-equipment` 가 이 판단을 맡음.

### 8-6. 전략 (ILC 는 지표 · 학습 방법은 지휘가 무엇이든)
**ILC = 상한 지표** (샘플마다 한 번 · 조건 스윕으로 제대로): 보정 기준 · `ilc_time` · I/Q 평균 · 이득 · 고치는 대역 · FSW 감쇠. 가장 좋은 ILC 는 자동 저장 (5 카드 [📊 ILC 조건 비교]). 이어서 ILC (`ilc_more`) 는 지휘 판단.
측정 조건 (샘플 · 감쇠 · '1000 MSps 넘으면 절반' 끄기) 이 바뀔 때만 다시.

**모델 학습 — 길은 여러 개, 같은 비중으로 비교 (최종 판단 = 기지국용 모델의 실측 D)**
- 길 A · **ILC 없이 직접 학습**: `dpd` 단계 + `method` `ila` (간접) / `dla` (직접) + `w_mode` (시간 · 집중 · ACLR) 로 GMP · DVR · OMP · 확장 모델의 K · M · L · 구간 · 반복 · mu · 정규화 · ACLR 가중을 `sweep` (거친 → 촘촘). 좋은 결과는 `more` 로 이어서.
- 길 B · **ILC 정답으로 학습**: `ilcm` (ILC 다시 안 돌림) — ILC 입력을 정답 삼아 모델이 배움 → 다듬기.
- 길 C · **새 방법 (확장 모델 · 학습 방식)**: 예) 직접 학습의 반복을 측정 대신 모델 시뮬레이션으로 여러 번 · 반송파별 · 교차 항 · 피킹 구간 분리 · 더 긴 메모리 · 정규화 · 학습 손실을 ACLR 대역 가중으로 · 하이브리드 (GMP + 작은 보정) 등. 기지국 FPGA 에 올릴 수 있는 계수 선형 구조면 무엇이든.
- 신경망 (`tnn`) 은 선택: 막혔을 때 원인 가리기용 기준선 (지금 42 dBm 신경망 D-46 · GMP D-40).
- 보고서마다 맨 위: 목표 −51 대비 · ILC 지표 · 길 A · B · C 최고 D.

### 8-7. 분석 재료 (D · B 말고도)
- `results.jsonl` 의 `diag` (진단 요약 전문: S 진폭 구간 오차 · N · Y · X · K 최고값 걸린 수 · 주파수별 학습 오차 4 칸 …)
- `run_file` (= `%APPDATA%\PADPDTester\runs\<run_id>.json.gz`, `python tools\run_data.py <run_id>`): 측정마다 Pout · Pin · Gain · EVM · PAE · DE · ACLR L/U · 반송파별 인접 (aclr_pos) · PAPR · NMSE ·
  **스펙트럼** (`spec_f/spec_db` = I/Q, `mod_f/mod_db` = FSW Mod. 화면) · **AM-AM / AM-PM 점** (`am_x/am_g/am_p`)
- `queue\xlsx\` (실험마다 측정 표 · 바이어스) · `queue\shots\` (FSW 화면) · `logs\진단요약_날짜.txt`
- 볼 것 예: L/U 비대칭 (메모리 · AM-PM) · 어깨 모양 (반송파별 IM3 vs 교차) · AM-AM 퍼짐 (메모리 깊이) · 진폭 구간별 오차 (S: 꺾임 · 피킹) ·
  반복에 따른 수렴 · EVM · PAE 와의 맞바꿈 · 측정 조건 (감쇠 · I/Q 평균) 에 따른 바닥 변화 · 실험 사이 재현성 (같은 설정 반복 차이).

## 9. 사용자에 대해

- RF PA 설계 엔지니어 (GaN Doherty). 측정 · 튜닝 직접 함. 바쁘고 휴대폰으로 보는 일이 많음 → 결론 먼저, 짧게.
- 결과는 진단 요약 코드 (예 `D-46 B36/41/33/26`) 로 불러줌.
- 하드웨어 수정은 마지막 수단. 최종 목표는 **GMP 로 잘 되는 DPD**.
