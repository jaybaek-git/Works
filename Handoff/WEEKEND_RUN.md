
> 🎯 최우선 목표: 기지국용 알고리즘 (GMP · DVR 등) 으로 Pout 42 dBm · 400 MHz 에서 ACLR worst −51 dBc 이하. ILC 는 −55 dBc 아래 (−60 이면 더 좋음). 모든 판단 · 보고는 목표 대비 남은 dB 로 (CLAUDE.md §0).

# 주말 자동 DPD 실험 — 시작 문구 (Claude Code 에 붙여 넣기)

이 폴더의 CLAUDE.md 를 먼저 끝까지 읽어. 그다음 아래대로 주말 내내 자동 DPD 실험을 지휘해 줘.

- 사용자는 DPD 탭 9 카드 [▶ 큐 시작] 을 이미 눌렀어. 큐 폴더: %APPDATA%\PADPDTester\queue
- 너의 역할은 지휘: 실험 파일 쓰기 · 확장 모델 만들기 · 결과 분석 · 보고서. 장비를 직접 제어하거나 핵심 프로그램 파일 · psu_panel.py · 바이어스를 건드리지 마.
- 목표 (최우선): 기지국용 알고리즘 (GMP · DVR · 확장 모델) 으로 Pout 42 dBm · 400 MHz 2CC 에서 **ACLR worst −51 dBc 이하**. ILC 는 **−55 dBc 아래 (−60 이면 더 좋음)**.
  지금: ILC −49 · 신경망 −46 · GMP −38 ~ −40. 핵심은 B 의 인접 칸.
- **이 목표를 위해 수단과 방법을 가리지 마.** 안전 규칙 (Pout 상한 · 바이어스 자동 변경 금지 · 최소 FSW 감쇠 · psu_panel.py · 신호 잠금) 안에서라면
  ILC 조건 · 학습 방식 · 모델 · 스윕 · 확장 모델 (새 알고리즘) · 측정 조건 무엇이든 시도해. 지금까지의 가설이 틀렸을 수 있다고 보고 새 방향도 찾아.
- 준비: 이 폴더의 queue_examples\*.json 을 %APPDATA%\PADPDTester\queue\ 로, models_ext\*.py 를 %APPDATA%\PADPDTester\models_ext\ 로 복사 (이미 있으면 덮어쓰지 말 것).
- 순서:
  0. 400 MHz 2CC · 사용자 신호 파일만 써 (신호는 잠겨 있음). 샘플 FSW 감쇠 아래로 내리지 마 (3B0 10 dB · 3X0 7 dB).
  1. status.json · results.jsonl 확인. 첫 묶음: fsw_check → fsw_opt (감쇠) → ILC 조건 스윕 (011_ilc_sweep 참고: 보정 기준 × I/Q 평균) → 가장 좋은 ILC 에서 이어서 ILC (012) 는 네 판단.
     ILC 는 학습 방법이 아니라 '이 샘플이 어디까지 되나' 지표. ILC 가 −51 보다 안 좋으면 측정 조건부터 손봐. (CLAUDE.md §8-5 · §8-6)
  1-1. 모델 학습은 길을 가리지 마: ILC 없이 직접 학습 스윕 (060_direct_sweep_gmp 참고 · ila / dla · 보정 기준) · ILC 정답 학습 (ilcm) · 새 방법 (확장 모델) 을 같은 비중으로 비교해, 기지국용 모델의 실측 D 가 가장 좋은 쪽으로.
  2. 일이 생길 때마다 (python tools\wait_event.py 로 기다렸다가 — 실험 끝 · 상태 변화 · 대기 2 개 아래, 최대 9 분): dpd-analyst 로 분석 → 다음 실험 3 ~ 6 개 작성 (원인을 가릴 땐 한 번에 한 가지 · 넓게 찾을 땐 sweep 으로 거칠게 → 좋은 쪽을 촘촘히) → 큐에 넣기. 대기 실험이 2 개 아래로 떨어지지 않게.
  3. 기존 설정 조합으로 더 안 좋아지면 스스로 원인을 다시 생각해 (지금까지 가설이 틀렸을 수도 있음).
     새 GMP · DVR 방식이나 항은 dpd-modeler 로 확장 모델 (models_ext\이름.py, CLAUDE.md §8-4) 을 만들고 가짜 PA 로 검증 →
     실험 파일 set 에 "ext_model" 을 넣어 큐에 → 결과 보고 고치고 (VERSION 올림) 다시. 핵심 프로그램 파일은 고치지 마.
  4. 실험이 바뀌거나 결정을 내릴 때마다 dpd-narrator 로 queue\narration.txt · 지금_상황.md 갱신 — 결정 이유는 queue\plan.md 에 짧게.
  5. dpd-reporter: 실험 끝날 때마다 실험별 · 알고리즘별, 매일 밤 일일, 일요일 밤 주간 보고서 (CLAUDE.md §8-3).
     주말 동안 '다음 선택지' 는 네가 스스로 골라 바로 실험해 (전략 §8-6: ILC 는 지표 · 학습은 길 A 직접 학습 스윕 · 길 B ILC 정답 · 길 C 새 방법 · 이어서 ILC · 신경망 기준선은 네 판단).
  5-1. FSW 이상 (과입력 · 튐 · 갑자기 나빠짐) 은 dpd-equipment 가 판단 (fsw_check → fsw_reset → 확인 → fsw_opt).
  6. status.json 의 state 가 wait_bias 오래 · fail 이 연속 3 번 · on=false 면 실험 추가를 멈추고 보고서에 이유를 써.
- 일요일 밤 (자정 전) 에 큐 폴더에 STOP 파일을 만들어 마무리하고 최종 보고서를 완성해.
- 계속 깨어 있으려면: wait_event 로 기다리기 → 판단 · 행동 → 다시 wait_event 를 반복해 (끝없이). 세션이 끊기면 Claude Code 반복 실행 (/loop) 으로 10 분마다 이 문서를 다시 따르게 해.
