# TEP 냉각계 고장 회복 분석

> 이 문서는 **화학공학을 전공하지 않은 사람도 읽을 수 있도록** 전문용어를 처음 등장할 때 쉽게 풀어 설명합니다.

Tennessee Eastman Process(TEP)의 냉각계 Fault 4와 Fault 14를 대상으로, **최초 정상범위 복귀시간만으로 공정 회복성을 평가해도 충분한가?**를 분석한 시계열 프로젝트입니다.

- **TEP(Tennessee Eastman Process)**: 실제 화학공장을 단순화해 컴퓨터에서 재현한 대표적인 공정 시뮬레이션 모델입니다.
- **공정(Process)**: 원료를 반응·분리·냉각하는 등 여러 단계를 거쳐 제품을 만드는 전체 과정입니다.
- **Fault**: 공정에 생긴 고장이나 비정상 상황입니다.
- **냉각계**: 반응기 온도를 낮추기 위해 냉각수를 보내고 조절하는 장치와 제어계통입니다.
- **시계열 데이터**: 시간 순서대로 기록된 데이터입니다.

초기 공개 reference data 분석에서 시작해, 최종적으로 동일한 30개 random seed를 사용한 **60회 paired closed-loop simulation**까지 확장했습니다.

여기서:

- **random seed**: 컴퓨터가 만드는 랜덤 변화를 다시 똑같이 재현하기 위한 번호입니다.
- **paired simulation**: Fault 4와 Fault 14를 같은 seed로 짝지어 고장 전 조건을 최대한 같게 맞춘 비교 실험입니다.
- **closed-loop**: 센서값을 보고 제어기가 자동으로 밸브 등을 조절하는 제어 방식입니다.
- **비모수 통계검정**: 데이터가 특정 분포를 따른다고 강하게 가정하지 않고 두 조건의 차이를 비교하는 통계 방법입니다.
- **effect size(효과크기)**: 단순히 “차이가 있다/없다”가 아니라 그 차이가 얼마나 크고 일관적인지 보여주는 값입니다.
- **민감도 분석**: 분석 기준을 조금 바꿔도 결론이 유지되는지 확인하는 과정입니다.

---

## 1. 핵심 결과

**Recovery Time(최초 복귀시간)**은 공정 회복의 한 측면만 보여주며, 안정적 회복을 대표하는 단독 지표로는 불안정할 수 있습니다.

Recovery Time은 “고장 이후 값이 정상범위 안에 **처음 한 번 들어온 시점**”만 보기 때문에 이후 다시 크게 흔들리는지까지는 알려주지 못합니다.

주 분석에서는 각 run의 고장 전 데이터를 이용해 `평균 ± 2표준편차(±2σ)`를 정상범위로 정했습니다.

| 지표 | Fault 4 | Fault 14 |
|---|---:|---:|
| Recovery Time 중앙값 | 6.0분 | 19.5분 |
| 정상범위 체류율 | 90.0% | 8.75% |
| IAE | 0.0418 | 0.4291 |
| ISE | 0.0022 | 0.1190 |
| Control Burden | 7.4401 | 12.1264 |
| 4시간 관측창 안에서 안정화 | 30/30 | 0/30 |

Fault 14는 Fault 4보다 훨씬 오래 흔들리고, 정상범위에 머무는 비율도 훨씬 낮았습니다.

그런데 **Recovery Time만 보면 항상 Fault 14가 더 나쁘게 나오지는 않았습니다.**

- Fault 14의 Recovery Time이 더 긴 경우: 20/30
- 같은 경우: 3/30
- Fault 14가 오히려 더 짧은 경우: 7/30

반면 아래 지표들은 30/30 모든 paired run에서 같은 방향으로 Fault 14가 더 나빴습니다.

- In-range Ratio
- IAE
- ISE
- Control Burden

즉, **처음 정상범위에 들어온 시점보다 이후에 얼마나 안정적으로 머무는지가 더 중요할 수 있습니다.**

### 대표 시각화

![Fault 4와 Fault 14의 First Recovery와 Stable Recovery 비교](ack_images/08_final_recovery_figure_v2.png)

![Recovery Time과 In-range Ratio 분포](ack_images/06_recovery_inrange_distribution.png)

![IAE, ISE and Control Burden 분포](ack_images/07_error_control_distribution.png)

---

## 2. 왜 Fault 4와 Fault 14를 비교했나?

두 fault는 모두 **반응기 냉각계**에 영향을 주지만 고장 방식이 다릅니다.

- **Fault 4**: 반응기로 들어가는 냉각수의 입구 온도가 갑자기 한 단계 변하는 외란(step disturbance)입니다.
- **Fault 14**: 반응기 냉각수를 조절하는 밸브가 걸려서(sticking) 부드럽게 움직이지 않는 고장입니다.
- **Reactor(반응기)**: 화학반응이 실제로 일어나는 장치입니다.
- **Cooling water(냉각수)**: 반응기의 열을 빼기 위해 순환시키는 물입니다.
- **XMEAS(9)**: TEP에서 9번째 측정변수로 기록되는 반응기 온도입니다.
- **XMV(10)**: TEP에서 10번째 조작변수로 기록되는 반응기 냉각수 관련 제어입력입니다.

두 fault는 같은 **reactor cooling system(반응기 냉각계)**에 영향을 주지만 **fault mechanism(고장이 생기는 방식)**이 다릅니다.

그래서 같은 계통에서 서로 다른 **post-fault dynamics(고장 이후 시간에 따른 움직임)**를 비교할 수 있습니다.

---

## 3. 반복 시뮬레이션 조건

- Simulator: `jkitchin/tennessee-eastman-profbraatz`
- Commit: `9a6c8e5fcef4a2850778704e7793c87b0a187005`
- Backend: pure Python
- Control mode: closed-loop
- Controller: decentralized multi-loop PI
  - **PI 제어기**: 현재 오차와 누적된 오차를 이용해 밸브 같은 조작값을 자동으로 조절하는 대표적인 제어기입니다.
  - **decentralized multi-loop**: 여러 개의 PI 제어기가 각각 담당 변수를 나누어 조절하는 구조입니다.
- Simulation time: 12시간
- Fault 발생: 8시간
- 기록 간격: 3분
- Seeds: 101~130
- Fault별 30회, 총 60회
- Shutdown: 0/60
  - **Shutdown**: 공정이 안전 또는 운전 조건을 벗어나 시뮬레이션이 강제로 정지한 경우입니다.

Fault 4와 Fault 14에는 같은 seed를 사용했습니다.

이렇게 하면 고장 전의 랜덤 조건을 최대한 같게 맞춘 상태에서 두 fault를 비교할 수 있습니다.

대표 seed 101에서 **pre-fault(고장 발생 전)** Reactor Temperature trajectory의 최대 절대차는 **0.0**이었습니다.

즉 두 실험은 고장 전에는 완전히 같은 온도 흐름에서 출발했고, 8시간 이후 서로 다른 fault만 적용되었습니다.

---

## 4. 사용한 지표

각 run의 0~8시간 pre-fault Reactor Temperature에서 `mean ± 2σ`를 주 분석 **normal band(통계적 정상범위)**로 사용합니다.

- **mean(평균)**: 고장 전 온도의 평균값입니다.
- **σ(시그마, 표준편차)**: 값들이 평균 주변에서 얼마나 흔들리는지를 나타내는 값입니다.
- **mean ± 2σ**: 평균에서 위아래로 표준편차의 2배만큼 잡은 범위입니다.

이 범위는 실제 산업 안전범위가 아니라 분석용 통계 기준입니다.

### Recovery Time

Fault 발생 후 Reactor Temperature가 **처음 정상범위에 들어오기까지 걸린 시간**입니다.

### Settling Time

값이 정상범위에 들어온 뒤 **30분 연속으로 그 안에 머물기 시작한 시점**입니다.

데이터가 3분 간격이므로 30분 연속 유지는 11개 연속 측정값으로 판단했습니다.

### In-range Ratio

Fault 후 첫 2시간 중 측정값이 정상범위 안에 있었던 비율입니다.

### IAE

**IAE(적분절대오차)**는 온도가 정상 평균에서 얼마나 벗어났는지를 절대값으로 바꾼 뒤 시간에 따라 모두 더한 값입니다.

값이 클수록 전체적으로 오차가 오래 또는 크게 지속됐다는 뜻입니다.

### ISE

**ISE(적분제곱오차)**는 오차를 제곱해 시간에 따라 더한 값입니다.

큰 오차에 더 큰 벌점을 주기 때문에 큰 흔들림에 특히 민감합니다.

### Control Burden

**Control Burden(제어입력 부담)**은 XMV(10)이 정상 평균에서 얼마나 벗어나 있었는지를 시간에 따라 누적한 값입니다.

주의: 실제 냉각수 사용량이나 에너지 소비량을 의미하지 않습니다.

---

## 5. 통계 결과

같은 seed끼리 짝지은 연속형 지표에는 **Wilcoxon signed-rank test**를 적용했고 5개 검정에는 **Holm correction**을 사용했습니다.

- **Wilcoxon signed-rank test**: 같은 조건으로 짝지어진 두 그룹의 값이 전반적으로 다른지 확인하는 비모수 통계검정입니다.
- **p-value**: 지금 같은 차이가 우연만으로 나타날 가능성을 판단하는 값입니다.
- **Holm correction**: 여러 지표를 동시에 검정할 때 우연히 유의한 결과가 나올 가능성을 줄이는 보정 방법입니다.
- **Rank-biserial r**: paired 차이가 어느 방향으로 얼마나 일관되게 나타나는지 보여주는 효과크기입니다.

| 지표 | Holm-adjusted p | Rank-biserial r |
|---|---:|---:|
| Recovery Time | 3.41e-4 | 0.788 |
| In-range Ratio | 3.15e-6 | -1.000 |
| IAE | 9.31e-9 | 1.000 |
| ISE | 9.31e-9 | 1.000 |
| Control Burden | 9.31e-9 | 1.000 |

특히 In-range Ratio, IAE, ISE, Control Burden은 **30쌍 모두 같은 방향으로 Fault 14가 더 나빴습니다.**

---

## 6. 정상범위를 바꿔도 결과가 유지되는가?

정상범위를 `±1σ`, `±2σ`, `±3σ`로 바꿔 민감도 분석을 했습니다.

| 정상범위 | Fault | Recovery 중앙값 | In-range 중앙값 | 4시간 안에 안정화 |
|---|---|---:|---:|---:|
| ±1σ | F4 | 13.5분 | 61.25% | 40% |
| ±1σ | F14 | 45.0분 | 5.00% | 0% |
| ±2σ | F4 | 6.0분 | 90.00% | 100% |
| ±2σ | F14 | 19.5분 | 8.75% | 0% |
| ±3σ | F4 | 6.0분 | 97.50% | 100% |
| ±3σ | F14 | 4.5분 | 12.50% | 0% |

특히 ±3σ에서는 Recovery Time 중앙값만 보면 Fault 14가 더 빠르게 보였습니다.

하지만 paired comparison에서는:

- Fault 14가 더 긴 경우: 12
- 같은 경우: 2
- Fault 14가 더 짧은 경우: 16
- Wilcoxon p = 0.1666

즉 **Recovery Time의 순위는 정상범위를 어떻게 정하느냐에 따라 흔들릴 수 있습니다.**

반면 In-range Ratio와 Settling의 상대적 차이는 계속 유지됐습니다.

---

## 7. Moving Average가 고장을 가릴 수 있는가?

3분 **raw data(가공하지 않은 원 데이터)**와 30분 **Moving Average(이동평균)**를 비교했습니다.

Moving Average는 일정 시간 구간의 평균을 계속 계산해 그래프를 부드럽게 만드는 방법입니다.

| Fault | Raw 최대 이탈 | 30분 MA 최대 이탈 | 정상 ±2σ 반폭 |
|---|---:|---:|---:|
| F4 | 0.2007 | 0.0267 | 0.0383 |
| F14 | 0.3799 | 0.0369 | 0.0373 |

30분 Moving Average를 적용하면 두 fault 모두 최대 이탈이 정상범위 반폭보다 작아졌습니다.

즉 **너무 강하게 smoothing하면 짧은 spike나 빠른 oscillation이 눈에 잘 안 보일 수 있습니다.**

---

## 8. 프로젝트 구조

```text
.
├── analysis.ipynb
├── ack_analysis.ipynb
├── README.md
├── REPORT.md
├── requirements.txt
├── data/
├── images/
├── ack_images/
└── ack_results/
```

주요 결과 파일:

```text
ack_results/
├── kpi_30runs.csv
├── summary_30runs.csv
├── statistical_tests.csv
├── directional_consistency.csv
└── sensitivity_sigma.csv
```

---

## 9. 필요한 패키지

```text
numpy
pandas
matplotlib
jupyter
scipy
statsmodels
```

---

## 10. 실행 방법

1. 필요한 라이브러리를 설치합니다.

```bash
pip install -r requirements.txt
```

2. TEP simulator source를 `tep_source/`에 준비합니다.

3. 아래 simulator commit을 사용합니다.

```text
9a6c8e5fcef4a2850778704e7793c87b0a187005
```

4. `analysis.ipynb`를 위에서 아래 순서대로 실행합니다.

5. `ack_analysis.ipynb`를 위에서 아래 순서대로 실행합니다.

6. 결과 CSV는 `ack_results/`, 그림은 `images/`와 `ack_images/`에서 확인할 수 있습니다.

---

## 11. 데이터 출처와 라이선스

- Reference data / simulator: `jkitchin/tennessee-eastman-profbraatz`
- License: BSD-3-Clause
- Reference data: 약 0~48시간, 3분 간격
- 반복 simulation: 0~12시간, 3분 간격
- Fault onset: 8시간

---

## 12. 한계

- Fault 4와 Fault 14 두 조건에 집중한 case study입니다.
- 주요 output은 Reactor Temperature입니다.
- 정상범위는 실제 안전 운전범위가 아니라 통계적 기준입니다.
- Settling은 Fault 후 4시간 관측창 안에서만 판단했습니다.
- Control Burden은 실제 물 또는 에너지 비용이 아닙니다.
- 결과는 사용한 closed-loop PI controller와 함께 나타난 response입니다.

---

## 한 줄 결론

> **정상범위에 한 번 들어온 것과 안정적으로 회복한 것은 다르다. Recovery Time 하나보다 In-range Ratio, Settling, 누적 오차와 제어입력 부담을 함께 보는 것이 더 안정적인 평가를 제공한다.**