# REPORT — TEP Fault 4 / Fault 14 시계열 분석

> 이 리포트는 **화학공학을 전공하지 않은 독자도 이해할 수 있도록** 전문용어를 처음 사용할 때 쉬운 뜻을 함께 설명합니다.

## 1. 무엇을 분석했나?

이 프로젝트는 Tennessee Eastman Process(TEP)의 두 냉각계 고장인 Fault 4와 Fault 14를 비교한 프로젝트입니다.

- **TEP(Tennessee Eastman Process)**: 실제 화학공장의 반응·분리·냉각 과정을 단순화해 컴퓨터에서 재현한 대표적인 공정 시뮬레이션 모델입니다.
- **공정(Process)**: 원료가 여러 장치를 거쳐 제품으로 바뀌는 전체 과정입니다.
- **Fault**: 공정에 생긴 고장 또는 비정상 상태입니다.
- **냉각계**: 반응기의 열을 빼기 위해 냉각수를 보내고 조절하는 장치와 제어계통입니다.

처음에는 공개된 reference data를 이용해 두 fault의 시계열 변화를 살펴봤고, 이후에는 결과가 우연이 아닌지 확인하기 위해 **같은 조건으로 여러 번 반복 시뮬레이션**했습니다.

최종적으로:

- Fault 4: 30회
- Fault 14: 30회
- 총 60회

를 비교했습니다.

가장 중요한 질문은 다음과 같습니다.

> **Reactor Temperature가 정상범위에 처음 들어온 시점만 보고 “회복했다”고 판단해도 될까?**

### 분석 질문

1. Fault 4와 Fault 14의 Reactor Temperature는 시간에 따라 어떻게 다르게 움직이는가?
2. First Recovery와 Stable Recovery는 같은 결론을 주는가?
3. Recovery Time, In-range Ratio, IAE, ISE, Control Burden 중 어떤 지표가 두 fault를 더 일관되게 구분하는가?
4. 정상범위를 ±1σ, ±2σ, ±3σ로 바꿔도 결론이 유지되는가?
5. Moving Average처럼 데이터를 부드럽게 만드는 방법이 fault 신호를 가릴 수 있는가?

---

## 2. 분석한 Fault와 변수

### Fault 4

**Reactor(반응기)**는 화학반응이 일어나는 장치입니다.

반응 중에는 열이 발생할 수 있어 **Cooling Water(냉각수)**를 이용해 온도를 조절합니다.

Fault 4는 반응기로 들어가는 냉각수의 온도가 갑자기 한 단계 변하는 **disturbance(외란)**입니다.

외란은 정상운전을 방해하는 갑작스러운 변화라는 뜻입니다.

### Fault 14

반응기 냉각수의 양을 조절하는 **Valve(밸브)**가 명령대로 부드럽게 움직이지 않고 걸리는 **sticking(고착)** fault입니다.

두 fault 모두 반응기 냉각계와 관련되어 있지만 고장 방식이 다릅니다.

주요 변수:

- **XMEAS(9)**: TEP의 9번째 측정변수로, Reactor Temperature(반응기 온도)를 뜻합니다.
- **XMV(10)**: TEP의 10번째 조작변수로, 반응기 냉각수 관련 **manipulated variable(제어기가 실제로 조절하는 값)**입니다.

---

## 3. 데이터와 시뮬레이션 조건

### 초기 reference data

- 파일: `d04_te.dat`, `d14_te.dat`
- 960 samples
- 3분 간격
- 약 48시간 데이터
- Fault 발생: 8시간
- Fault 전 정상구간: 160 samples

### 반복 시뮬레이션

사용한 simulator:

`jkitchin/tennessee-eastman-profbraatz`

재현에 사용한 commit:

`9a6c8e5fcef4a2850778704e7793c87b0a187005`

조건:

- pure Python backend
  - **backend**: 시뮬레이션 계산을 실제로 수행하는 내부 구현 방식입니다.
- CLOSED_LOOP
  - **closed-loop**: 센서값을 보고 제어기가 자동으로 밸브 등 조작값을 바꾸는 방식입니다.
- decentralized multi-loop PI controller
  - **PI controller**: 현재 오차와 지금까지 누적된 오차를 이용해 조작값을 자동 조절하는 대표적인 제어기입니다.
  - **decentralized multi-loop**: 여러 PI 제어기가 각각 맡은 변수를 나누어 제어하는 구조입니다.
- 12시간 simulation
- 8시간에 Fault 발생
- 3분 간격 기록
- seed 101~130
  - **random seed**: 랜덤한 변화를 다시 똑같이 재현하기 위한 번호입니다.
- Fault별 30회
- 총 60회
- shutdown 0회

Fault 4와 Fault 14에는 같은 seed를 사용했습니다.

이렇게 하면 고장 전의 랜덤 조건을 맞춘 상태에서 고장 종류만 바꿔 비교할 수 있습니다.

대표 seed 101에서는 Fault 발생 전 Reactor Temperature의 최대 차이가 **0.0**이었습니다.

---

## 4. 결측치와 이상값은 어떻게 처리했나?

분석 대상 변수에서 결측치는 발견되지 않았습니다.

Fault 이후 갑자기 튀는 값이나 크게 흔들리는 값은 삭제하지 않았습니다.

그 값 자체가 고장 반응이기 때문입니다.

즉, fault 이후의 spike나 oscillation을 이상치라고 보고 제거하면 오히려 분석해야 할 정보를 없애게 됩니다.

---

## 5. 정상범위는 어떻게 정했나?

각 run의 Fault 전 0~8시간 Reactor Temperature를 이용해 평균과 표준편차를 계산했습니다.

주 분석에서는:

`정상범위 = 평균 ± 2σ`

로 정했습니다.

여기서:

- **평균(mean)**: 고장 전 온도의 평균값입니다.
- **σ(시그마, 표준편차)**: 값들이 평균 주변에서 얼마나 흔들리는지 나타내는 값입니다.
- **±2σ**: 평균에서 위아래로 표준편차의 2배만큼 잡은 범위입니다.

이 정상범위는 **실제 산업공정의 안전 기준이 아니라 분석을 위한 통계 기준**입니다.

그래서 결과가 이 기준에 너무 의존하지 않는지 확인하기 위해:

- ±1σ
- ±2σ
- ±3σ

도 추가로 비교했습니다.

---

## 6. 사용한 핵심 지표

### Recovery Time

**Recovery Time(최초 복귀시간)**은 Fault 이후 Reactor Temperature가 정상범위에 **처음 들어오기까지 걸린 시간**입니다.

### Settling Time

**Settling Time(안정화시간)**은 Reactor Temperature가 정상범위 안에 **30분 동안 계속 머물기 시작한 시점**입니다.

3분 간격 데이터이므로 11개 연속 측정값이 정상범위에 있을 때 Settling으로 판단했습니다.

### In-range Ratio

**In-range Ratio(정상범위 체류율)**은 Fault 후 첫 2시간 동안 정상범위 안에 있었던 비율입니다.

### IAE

**IAE(Integrated Absolute Error, 적분절대오차)**는 온도가 정상 평균에서 얼마나 벗어났는지를 절대값으로 바꾼 뒤 시간에 따라 모두 더한 값입니다.

값이 클수록 오차가 많이 또는 오래 발생했다는 뜻입니다.

### ISE

**ISE(Integrated Squared Error, 적분제곱오차)**는 오차를 제곱해서 시간에 따라 더한 값입니다.

큰 오차일수록 더 크게 반영됩니다.

### Control Burden

**Control Burden(제어입력 부담)**은 XMV(10)이 정상 평균에서 얼마나 벗어났는지를 시간에 따라 누적한 상대적 지표입니다.

이 값은 **실제 냉각수 사용량이나 에너지 소비량이 아닙니다.**

---

## 7. 초기 Reference 분석에서 본 특징

아래 그림 1은 이 프로젝트의 핵심 주장인 **“처음 정상범위에 들어온 것과 안정적으로 회복한 것은 다르다”**를 보여주는 대표 시각화입니다.

Fault 4는 8시간에 Reactor Temperature가 크게 한 번 튄 뒤 빠르게 정상 부근으로 돌아왔습니다.

Reference Fault 4:

- First Recovery: 약 3분
- Settling Start: 약 57분

즉 처음 정상범위에 들어온 것은 빨랐지만, 실제로 안정적으로 머무르기까지는 더 오래 걸렸습니다.

Fault 14는 Fault 이후 Reactor Temperature가 계속 크게 흔들리는 모습을 보였습니다.

![Fault 4와 Fault 14의 First Recovery와 Stable Recovery 비교](ack_images/08_final_recovery_figure_v2.png)

**그림 1.** 동일 seed 101에서 Fault 4와 Fault 14의 Reactor Temperature를 비교한 그림입니다. 음영 영역은 고장 전 데이터에서 계산한 통계적 정상범위(mean ± 2σ)를 뜻합니다. Fault 4는 고장 후 9분에 처음 정상범위에 들어오고 같은 시점부터 30분 연속 안정화가 시작됩니다. Fault 14는 63분 후 처음 정상범위에 들어오지만 이후 다시 반복적으로 벗어나며, 4시간 관측창 안에서는 30분 연속 안정화가 관찰되지 않았습니다.

---

## 8. 30회 반복실험 결과

표의 **중앙값(median)**은 값을 크기순으로 놓았을 때 가운데 값입니다.

평균보다 극단적으로 큰 값이나 작은 값의 영향을 덜 받습니다.

**IQR(사분위범위)**은 데이터의 가운데 50%가 얼마나 넓게 퍼져 있는지를 보여주는 값입니다.

IQR이 크면 run마다 결과 차이가 컸다는 뜻입니다.

| 지표 | Fault 4 중앙값 [IQR] | Fault 14 중앙값 [IQR] |
|---|---:|---:|
| Recovery Time | 6.0 [6.0]분 | 19.5 [45.0]분 |
| In-range Ratio | 90.0 [6.875]% | 8.75 [4.375]% |
| IAE | 0.0418 [0.0054] | 0.4291 [0.0236] |
| ISE | 0.0022 [0.0006] | 0.1190 [0.0146] |
| Control Burden | 7.4401 [0.4069] | 12.1264 [0.7448] |
| 4시간 관측창 안에서 Settling | 30/30 | 0/30 |

Fault 14는 대부분의 지표에서 Fault 4보다 훨씬 나쁜 결과를 보였습니다.

![Recovery Time과 In-range Ratio 분포](ack_images/06_recovery_inrange_distribution.png)

**그림 2.** 30 paired runs의 Recovery Time과 In-range Ratio 분포입니다.

---

## 9. Recovery Time만 보면 어떤 문제가 있나?

같은 seed끼리 Fault 4와 Fault 14를 비교했습니다.

Recovery Time 기준:

- Fault 14가 더 늦음: 20/30
- 같음: 3/30
- Fault 14가 더 빠름: 7/30

즉 Fault 14가 더 심한 고장인데도 **일부 run에서는 더 빨리 정상범위에 처음 들어왔습니다.**

반면 아래 지표들은 30/30 모든 run에서 Fault 14가 더 나빴습니다.

- In-range Ratio
- IAE
- ISE
- Control Burden

따라서 **First Recovery 하나만으로 고장 회복을 평가하면 잘못된 인상을 줄 수 있습니다.**

---

## 10. 통계검정

같은 seed끼리 짝지어 비교했기 때문에 **Wilcoxon signed-rank test**를 사용했습니다.

이 검정은 “같은 조건으로 짝지어진 두 그룹의 값이 전반적으로 다른가?”를 확인하는 비모수 통계검정입니다.

여러 지표를 동시에 검정했기 때문에 **Holm correction**도 적용했습니다.

Holm correction은 여러 번 통계검정을 할 때 우연히 유의한 결과가 나올 가능성이 커지는 문제를 줄이기 위한 보정 방법입니다.

또한:

- **p-value**: 관찰한 차이가 우연으로 나타났을 가능성을 판단하는 값입니다.
- **Rank-biserial r**: 두 조건의 차이가 어느 방향으로 얼마나 일관적인지 보여주는 효과크기입니다.

| 지표 | Holm-adjusted p | Rank-biserial r |
|---|---:|---:|
| Recovery Time | 3.41e-4 | 0.788 |
| In-range Ratio | 3.15e-6 | -1.000 |
| IAE | 9.31e-9 | 1.000 |
| ISE | 9.31e-9 | 1.000 |
| Control Burden | 9.31e-9 | 1.000 |

In-range Ratio, IAE, ISE, Control Burden은 두 fault의 차이가 매우 일관되게 나타났습니다.

![IAE, ISE, Control Burden 분포](ack_images/07_error_control_distribution.png)

**그림 3.** 30 paired runs의 IAE, ISE, Control Burden 분포입니다.

4시간 관측창 내 Settling 결과:

- Fault 4: 30/30
- Fault 14: 0/30
- exact p = 1.86e-9

---

## 11. 정상범위를 바꾸면 어떻게 되나?

| 정상범위 | Fault | Recovery 중앙값 | In-range 중앙값 | 4시간 안 Settling |
|---|---|---:|---:|---:|
| ±1σ | F4 | 13.5분 | 61.25% | 40% |
| ±1σ | F14 | 45.0분 | 5.00% | 0% |
| ±2σ | F4 | 6.0분 | 90.00% | 100% |
| ±2σ | F14 | 19.5분 | 8.75% | 0% |
| ±3σ | F4 | 6.0분 | 97.50% | 100% |
| ±3σ | F14 | 4.5분 | 12.50% | 0% |

±3σ에서는 Recovery Time 중앙값만 보면 Fault 14가 더 빨라졌습니다.

하지만 paired 결과는:

- Fault 14가 더 늦음: 12
- 같음: 2
- Fault 14가 더 빠름: 16
- Wilcoxon p = 0.1666

즉 통계적으로 유의한 차이가 없고 방향도 섞여 있습니다.

이 결과는 **Recovery Time이 정상범위를 어떻게 정하느냐에 따라 흔들릴 수 있다**는 것을 보여줍니다.

---

## 12. 트렌드 / 계절성 / 노이즈

### 트렌드

Fault 발생 전 Reactor Temperature는 약 120.4 부근에서 유지되며 뚜렷한 장기 상승 또는 하락 추세는 보이지 않았습니다.

핵심 변화는 8시간의 Fault onset 이후에 나타납니다.

### 계절성

이 데이터는 달력 기준의 일·주·월 계절성을 분석하는 자료가 아닙니다.

따라서 일반적인 의미의 계절성은 뚜렷하지 않습니다.

Fault 14에서 나타나는 반복 진동은 시간대나 요일에 따른 계절성이 아니라 **고장 이후 나타나는 동적 반복 패턴**으로 해석했습니다.

### 노이즈

Fault 전에도 작은 단기 변동이 존재합니다.

Moving Average는 이런 변동을 부드럽게 만드는 데 유용하지만, 너무 크게 적용하면 실제 Fault signal도 같이 약해질 수 있습니다.

---

## 13. Moving Average 분석

**Moving Average(이동평균)**는 일정 시간 구간의 평균을 계속 계산해 그래프를 부드럽게 만드는 방법입니다.

**Smoothing(평활화)**은 이런 방식으로 짧고 빠른 흔들림을 줄여 전체 흐름을 보기 쉽게 만드는 처리입니다.

3분 raw data와 30분 Moving Average를 비교했습니다.

| Fault | Raw 최대 이탈 | 30분 MA 최대 이탈 | 정상 ±2σ 반폭 |
|---|---:|---:|---:|
| F4 | 0.2007 | 0.0267 | 0.0383 |
| F14 | 0.3799 | 0.0369 | 0.0373 |

Moving Average를 크게 잡으면 Fault spike나 빠른 oscillation이 작게 보일 수 있습니다.

따라서 smoothing은 유용하지만, 너무 강하면 Fault signal을 가릴 수 있습니다.

---

## 14. ACF / FFT 결과

- **ACF(Autocorrelation Function, 자기상관함수)**: 현재 값과 몇 시점 전 값이 얼마나 비슷한지 확인해 반복성을 보는 방법입니다.
- **FFT(Fast Fourier Transform)**: 시계열 안에 어떤 반복 주파수가 강하게 들어 있는지 찾는 방법입니다.
- **Nyquist limit**: 현재 sampling interval로 구분할 수 있는 가장 빠른 반복 변화의 한계입니다.

Fault 14의 반복적인 흔들림을 ACF와 FFT로 확인했습니다.

빠른 반복 성분이 존재했지만 dominant frequency가 sampling limit에 가까웠습니다.

따라서 정확한 물리적 주기를 단정하지 않고:

> **sampling limit에 가까운 빠른 반복 성분이 존재한다**

정도로 해석했습니다.

---

## 15. 직접 확인한 사실과 해석을 구분

### 직접 계산으로 확인한 사실

- 60 runs 모두 shutdown 없이 종료
- seed 101 pre-fault temperature 최대 차이 = 0.0
- ±2σ에서 Fault 4 Settling 30/30, Fault 14 0/30
- IAE/ISE/Control Burden은 30/30 pairs에서 Fault 14가 큼
- In-range Ratio는 30/30 pairs에서 Fault 14가 낮음
- ±3σ Recovery 방향은 12:2:16, p=0.1666

### 해석

- Recovery Time 하나만 사용하면 일시적 정상범위 통과를 안정적 회복으로 오해할 수 있음
- 여러 지표를 함께 보는 것이 더 안정적인 해석을 제공함

### 현재 데이터로 주장하면 안 되는 것

- Control Burden이 실제 물 사용량과 같다는 주장
- Control Burden이 실제 에너지 비용과 같다는 주장
- Fault 14가 모든 제어기에서 똑같이 행동한다는 주장
- 이 결과가 모든 TEP fault와 실제 공정에 그대로 일반화된다는 주장

---

## 16. 화학공학과 데이터분석에서의 의미

화학공학 관점에서는:

- disturbance rejection
- Reactor Temperature
- manipulated variable
- process recovery

를 분석한 프로젝트입니다.

쉽게 말하면 **고장이 생겼을 때 공정이 얼마나 흔들리고, 제어기가 어떻게 반응하며, 다시 안정적인 상태로 돌아오는지**를 본 것입니다.

데이터분석 관점에서는:

- industrial time-series benchmark
- 평가 지표의 신뢰성
- threshold sensitivity
- 반복실험
- 통계검정

을 다룬 프로젝트입니다.

---

## 17. 핵심 인사이트

### 인사이트 1 — First Recovery와 Stable Recovery는 다를 수 있다

- **관찰(Fact):** ±2σ 기준에서 Fault 4는 30/30 run이 4시간 관측창 안에서 Settling에 성공했지만 Fault 14는 0/30이었습니다.
- **해석(Why):** 정상범위에 한 번 들어오는 것만으로는 이후의 재이탈 여부를 알 수 없습니다.
- **행동(Action):** Recovery Time과 함께 Settling 또는 In-range Ratio를 같이 확인해야 합니다.

### 인사이트 2 — Recovery Time은 정상범위 설정에 민감하다

- **관찰(Fact):** ±3σ에서 Recovery 중앙값은 F4 6.0분, F14 4.5분이었지만 paired 방향은 12:2:16으로 섞였고 p=0.1666이었습니다.
- **해석(Why):** 정상범위가 넓어지면 크게 흔들리는 signal도 정상범위를 순간적으로 통과하기 쉬워집니다.
- **행동(Action):** 하나의 threshold에만 의존하지 않고 sensitivity analysis를 함께 확인해야 합니다.

### 인사이트 3 — 누적 오차와 제어입력 이탈은 더 일관된 차이를 보였다

- **관찰(Fact):** In-range Ratio, IAE, ISE, Control Burden은 30/30 paired runs에서 모두 Fault 14가 더 불리한 방향이었습니다.
- **해석(Why):** 한 시점이 아니라 일정 시간 동안의 상태를 누적해서 보기 때문에 지속적인 oscillation을 더 잘 반영합니다.
- **행동(Action):** Post-fault 상태를 평가할 때 단일 시점 지표보다 여러 시간누적 지표를 함께 사용합니다.

### 인사이트 4 — 강한 smoothing은 이상신호를 가릴 수 있다

- **관찰(Fact):** 30분 Moving Average의 최대 deviation은 F4와 F14 모두 각 normal ±2σ half-width보다 작아졌습니다.
- **해석(Why):** 짧은 spike와 빠른 oscillation이 평균화되기 때문입니다.
- **행동(Action):** 시계열 분석에서는 raw signal과 smoothed signal을 함께 확인합니다.

---

## 18. 한계

1. Fault 4와 Fault 14 두 fault만 분석했습니다.
2. 주요 output을 Reactor Temperature에 집중했습니다.
3. 정상범위는 실제 안전 운전범위가 아닙니다.
4. Settling은 Fault 후 4시간 관측창 안에서만 확인했습니다.
5. Control Burden은 실제 물 또는 에너지 비용이 아닙니다.
6. 결과는 사용한 closed-loop PI controller와 함께 나타난 response입니다.

---

## 19. 재현성

### 데이터 출처 / 시간범위 / 라이선스

- Reference data / simulator: `jkitchin/tennessee-eastman-profbraatz`
- License: BSD-3-Clause
- Reference data: 약 0~48시간, 3분 간격
- Repeated simulation: 0~12시간, 3분 간격

### 실행 순서

1. `pip install -r requirements.txt`
2. `tep_source/`에 simulator source 준비
3. 아래 commit 사용

```text
9a6c8e5fcef4a2850778704e7793c87b0a187005
```

4. `analysis.ipynb`를 위에서 아래 순서대로 실행
5. `ack_analysis.ipynb`를 위에서 아래 순서대로 실행
6. `images/`, `ack_images/`, `ack_results/`의 결과 확인

세부 조건:

- Backend: Python
- Control mode: CLOSED_LOOP
- Seeds: 101~130
- Fault onset: 8시간
- Record interval: 180초
- Main KPI window: 8~10시간
- Settling observation window: 8~12시간

---

## 20. 용어 사전

| 용어 | 쉬운 뜻 |
|---|---|
| TEP | 실제 화학공장을 단순화한 컴퓨터 시뮬레이션 모델 |
| Reactor | 화학반응이 일어나는 장치 |
| Cooling water | 반응기에서 열을 빼기 위한 냉각수 |
| Fault | 고장 또는 비정상 상태 |
| Disturbance | 정상운전을 방해하는 갑작스러운 변화 |
| Valve sticking | 밸브가 걸려 명령대로 움직이지 않는 고장 |
| XMEAS | 센서로 측정한 공정값 |
| XMV | 제어기가 직접 조절하는 값 |
| Closed-loop | 센서값을 보고 제어기가 자동으로 조절하는 방식 |
| PI controller | 현재 오차와 누적 오차를 이용해 자동 조절하는 제어기 |
| Random seed | 랜덤 조건을 다시 재현하기 위한 번호 |
| Paired design | 같은 seed끼리 짝지어 두 조건을 비교하는 실험설계 |
| Sampling interval | 데이터를 기록하는 시간 간격 |
| Trajectory | 시간에 따라 값이 움직인 전체 경로 |
| Mean | 평균 |
| Standard deviation / σ | 값들이 평균 주변에서 퍼진 정도 |
| Recovery Time | 처음 정상범위에 들어오기까지 걸린 시간 |
| Settling Time | 정상범위에 일정 시간 계속 머물기 시작한 시점 |
| In-range Ratio | 전체 관측시간 중 정상범위에 있었던 비율 |
| IAE | 절대오차를 시간에 따라 누적한 값 |
| ISE | 제곱오차를 시간에 따라 누적한 값 |
| Control Burden | 제어입력의 정상 수준 이탈을 누적한 상대 지표 |
| Median | 크기순으로 정렬했을 때 가운데 값 |
| IQR | 가운데 50% 데이터의 퍼진 정도 |
| p-value | 관찰된 차이가 우연인지 판단하는 통계값 |
| Wilcoxon test | 짝지어진 두 그룹을 비교하는 비모수 검정 |
| Holm correction | 여러 통계검정을 할 때 우연한 유의성을 줄이는 보정 |
| Effect size | 차이가 얼마나 크고 일관적인지 나타내는 값 |
| Moving Average | 일정 구간 평균으로 그래프를 부드럽게 만드는 방법 |
| Smoothing | 짧은 흔들림을 줄여 그래프를 부드럽게 만드는 처리 |
| ACF | 시차를 둔 값끼리 얼마나 비슷한지 보는 분석 |
| FFT | 반복되는 주파수 성분을 찾는 방법 |
| Nyquist limit | sampling 간격으로 구분 가능한 가장 빠른 반복 변화 |
| Backend | 계산을 수행하는 내부 구현 방식 |
| Commit | 특정 코드 버전을 가리키는 Git 식별자 |
| Shutdown | 공정이 조건을 벗어나 시뮬레이션이 강제로 멈춘 상태 |

---

## 21. AI 사용 로그

| 사용 작업 | 사용 이유 | 검증 방법 |
|---|---|---|
| Recovery / Settling 계산 코드 작성 | 반복 계산을 빠르게 구현하기 위해 | 시간축과 normal band를 직접 확인하고 30분 조건을 11개 연속 sample로 수정 |
| IAE / ISE 코드 작성 | 누적 오차 계산식을 정확히 구현하기 위해 | 각 run에서 직접 재계산하고 Fault별 분포 확인 |
| Control Burden 정의와 코드 | 제어입력의 정상상태 이탈을 정량화하기 위해 | XMV(10)과 pre-fault 정상평균으로 직접 계산 |
| 시각화 코드 | 여러 run의 차이를 한눈에 비교하기 위해 | 저장된 PNG와 원 데이터 값을 비교 |
| Wilcoxon / Holm / effect size | 단순 평균 비교보다 통계적으로 검증하기 위해 | scipy/statsmodels 결과와 paired 방향성 직접 확인 |
| ±1σ / ±2σ / ±3σ 민감도 분석 | 정상범위 정의에 따른 결과 변화 확인 | 동일 60 runs에서 threshold만 변경해 재계산 |
| ACF / FFT 해석 | Fault 14 반복 패턴을 확인하기 위해 | sampling interval과 Nyquist limit를 직접 확인 |
| 문장 다듬기 | 분석 결과를 읽기 쉽게 정리하기 위해 | 모든 수치를 notebook/CSV 결과와 대조 |

AI가 제안한 해석은 직접 계산결과와 대조한 뒤 사용했습니다.

데이터에서 확인할 수 없는 물리적 비용, 인과관계, 일반화는 주장하지 않았습니다.

---

## 22. AI 도움 없이 작성하는 최종 결론

이 부분은 제출자가 직접 작성합니다.

아래 질문에 본인의 말로 답하면 됩니다.

- Fault 4와 Fault 14의 가장 큰 차이는 무엇이었는가?
- Recovery Time 하나만 보면 무엇을 놓칠 수 있는가?
- 어떤 수치가 그 판단을 뒷받침하는가?
- 분석의 한계는 무엇인가?

---

## 23. 한 줄 요약

> **처음 정상범위에 들어온 것과 실제로 안정적으로 회복한 것은 다르며, Recovery Time 하나보다 여러 회복 지표를 함께 보는 것이 더 안정적이었다.**