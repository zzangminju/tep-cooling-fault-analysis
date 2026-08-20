# TEP Cooling-System Fault Analysis

### Tennessee Eastman Process 냉각계 고장의 시계열 동적 특성 및 제어성능 비교

Tennessee Eastman Process(TEP)의 냉각계 관련 이상인 **Fault 4**와 **Fault 14**를 비교한 시계열 데이터 분석 프로젝트입니다.

단순히 고장 전후 그래프를 비교하는 데서 끝내지 않고 다음을 정량적으로 분석했습니다.

- 정상범위 최초 복귀시간
- 실제 안정화 시간
- 정상범위 체류율
- 온도 제어오차
- 냉각수 관련 제어신호의 움직임
- 반복 진동
- 변수 간 시간관계
- 집계방법에 따른 해석 차이

---

## 1. 분석 대상

두 Fault는 모두 반응기 냉각계와 관련되어 있지만 원인이 다릅니다.

| Fault | 설명 |
|---|---|
| **Fault 4** | Reactor Cooling Water Inlet Temperature Step |
| **Fault 14** | Reactor Cooling Water Valve Sticking |

Fault 4는 냉각수 유입 온도 조건이 갑자기 변하는 이상이고, Fault 14는 냉각수 밸브가 원활하게 움직이지 않는 이상입니다.

본 프로젝트에서는 두 고장이 반응기 온도와 제어시스템에 서로 다른 시계열 패턴을 만드는지 비교했습니다.

---

## 2. 핵심 분석 질문

1. Fault 4와 Fault 14의 반응기 온도 패턴은 어떻게 다른가?
2. 정상범위에 처음 돌아오는 시간과 실제 안정화 시간은 같은가?
3. Fault 발생 후 온도 제어성능은 정상 운전보다 얼마나 나빠지는가?
4. 반응기 온도를 유지하기 위해 제어신호는 평소보다 얼마나 크게 움직이는가?
5. 이동평균이나 시간차 분석방법에 따라 같은 데이터를 다르게 해석할 가능성은 없는가?

---

## 3. Fault 4 vs Fault 14

![Fault 4 vs Fault 14 Temperature](images/10_temperature_fault_comparison.png)

두 Fault는 뚜렷하게 다른 형태를 보였습니다.

**Fault 4**

> 순간적인 큰 온도 변화 → 빠른 최초 회복 → 이후 안정화

**Fault 14**

> 지속적인 온도 진동 → 매우 낮은 정상범위 체류율 → 안정화 실패

Fault 4는 고장 직후 한 번 크게 움직인 뒤 비교적 빠르게 정상 수준으로 돌아왔습니다.

반면 Fault 14는 고장 이후에도 온도가 계속 위아래로 크게 움직였습니다.

---

## 4. 핵심 결과

| Metric | Fault 4 | Fault 14 |
|---|---:|---:|
| Recovery Time | **3 min** | **9 min** |
| Settling Time | **57 min** | **Not settled** |
| In-range Ratio | **77.5%** | **2.5%** |
| IAE | **0.0494** | **0.4415** |
| ISE | **0.00333** | **0.12130** |
| Control Burden | **7.364** | **12.679** |
| IAE / Normal | **1.66×** | **15.08×** |
| ISE / Normal | **4.56×** | **175.81×** |
| Control Burden / Normal | **8.34×** | **15.60×** |

가장 중요한 결과는 **정상범위에 한 번 들어온 것과 실제로 안정된 것은 다르다**는 점입니다.

Fault 4는 3분 만에 정상범위에 처음 돌아왔지만 안정적으로 정상범위에 머무르기 시작하기까지는 57분이 걸렸습니다.

Fault 14는 9분 후 한 번 정상범위에 들어왔지만 이후 계속 이탈하여 관측기간 동안 안정적인 정착이 확인되지 않았습니다.

---

## 5. 핵심 권장 시각화 3개

본 프로젝트에서 가장 중요한 시각화는 다음 세 가지입니다.

| 시각화 | 목적 |
|---|---|
| **Fault 4 vs Fault 14 원본 시계열** | 고장 종류에 따라 온도 반응 형태가 어떻게 달라지는지 직접 비교 |
| **30분 이동평균** | 작은 변동을 줄여 전체 흐름을 확인하고 평활화의 한계도 확인 |
| **정상 대비 성능 저하 비교** | IAE, ISE, Control Burden이 정상 운전보다 얼마나 증가했는지 비교 |

### 5.1 원본 시계열

![Fault Comparison](images/10_temperature_fault_comparison.png)

Fault 4는 짧고 강한 변화가 중심인 반면 Fault 14는 반복적인 진동이 지속되었습니다.

### 5.2 30분 이동평균

![30-minute Moving Average](images/02_moving_average.png)

이동평균은 전체적인 흐름을 보기 쉽게 하지만 순간적인 Fault 신호를 작게 보이게 할 수 있습니다.

따라서 이동평균만으로 이상 여부를 판단하지 않고 원본 데이터와 함께 분석했습니다.

### 5.3 정상 대비 성능 저하

![Normalized Performance](images/09_normalized_fault_comparison.png)

정상 운전을 1배로 두었을 때 Fault 14는 IAE, ISE, Control Burden 모두에서 Fault 4보다 큰 성능 저하를 보였습니다.

---

## 6. 집계방법에 따른 반례

같은 데이터라도 집계방법에 따라 결과가 다르게 보일 수 있는지 확인했습니다.

Fault 발생 후 첫 2시간의 반응기 온도를 **원본 3분 데이터**와 **30분 이동평균**으로 비교했습니다.

| Fault | Raw Max Abs. Deviation | 30-min MA Max Abs. Deviation | Normal ±2σ Half Width |
|---|---:|---:|---:|
| Fault 4 | **0.2007 °C** | **0.0267 °C** | **0.0383 °C** |
| Fault 14 | **0.3799 °C** | **0.0369 °C** | **0.0373 °C** |

원본 데이터에서는 두 Fault 모두 정상범위를 크게 벗어나는 변화가 확인되었습니다.

그러나 30분 이동평균에서는 최대 편차가 정상범위 반폭보다 작아졌습니다.

즉 이동평균만 확인하면 순간적인 spike나 빠른 진동을 실제보다 작게 평가할 가능성이 있습니다.

따라서 본 프로젝트에서는 **이동평균은 전체 흐름 확인용으로 사용하고, Fault 분석에는 원본 3분 데이터도 함께 사용했습니다.**

---

## 7. 공정 출력과 제어신호

Fault 4에서는 반응기 온도가 비교적 빠르게 정상 수준으로 돌아왔지만 냉각수 관련 조작변수 `XMV(10)`은 이전보다 높은 수준에서 움직였습니다.

![Control Response](images/04_control_response.png)

Fault 발생 순간 XMV(10)은 약 **16.1% 증가**했습니다.

이를 통해:

> **공정 출력값이 정상처럼 보여도 제어시스템은 평소보다 더 크게 움직이고 있을 수 있다.**

는 점을 확인했습니다.

---

## 8. Control Burden

제어신호의 움직임을 정량적으로 비교하기 위해 **Control Burden**을 정의했습니다.

Control Burden은 XMV(10)이 정상 평균에서 벗어난 절대적인 크기를 시간에 따라 누적한 값입니다.

![Control Burden](images/08_control_burden.png)

정상 2시간 구간과 비교하면:

- Fault 4: **8.34× normal**
- Fault 14: **15.60× normal**

이었습니다.

Control Burden은 실제 냉각수 사용량이나 에너지 소비량이 아니라 **제어신호의 움직임을 비교하기 위한 분석지표**입니다.

---

## 9. Fault 14의 반복 진동

Fault 14의 반복적인 온도 움직임이 단순한 random noise인지 확인하기 위해 자기상관(Autocorrelation)을 분석했습니다.

![Fault 14 Autocorrelation](images/11_fault14_autocorrelation.png)

여러 시간차에서 높은 자기상관이 반복적으로 나타났습니다.

이는 Fault 14의 온도 변화가 단순한 무작위 흔들림보다는 **반복적인 구조를 포함하고 있음을 보여줍니다.**

FFT 분석에서도 빠른 반복 성분이 확인되었습니다.

다만 데이터가 3분 간격이기 때문에 계산된 약 6.6분을 정확한 물리적 진동주기로 단정하지 않았습니다.

---

## 10. 사용한 분석 방법

- Moving Average
- First Difference
- Rolling Standard Deviation
- Recovery Time
- Settling Time
- In-range Ratio
- IAE
- ISE
- Control Burden
- Cross-correlation
- Autocorrelation
- FFT

분석기법을 많이 사용하는 것보다 **각 방법으로 얻은 결과가 실제 공정에서 무엇을 의미하는지 해석하는 것**에 중점을 두었습니다.

---

## 11. Chemical Engineering Relevance

본 프로젝트는 다음 화학공학 분야와 연결됩니다.

**Process Monitoring**  
센서값과 조작변수를 함께 이용한 공정상태 모니터링

**Fault Diagnosis**  
고장별 시간적 패턴을 이용한 이상 유형 구분

**Process Control**  
Recovery Time, Settling Time, IAE, ISE를 이용한 제어성능 평가

**Predictive Maintenance**  
밸브 및 냉각계 제어신호의 비정상적인 움직임 탐색

**Process Safety**  
냉각계 이상 이후 순간적인 정상 복귀가 아닌 실제 안정적인 회복 여부 평가

---

## 12. Data

Tennessee Eastman Process reference test data를 사용했습니다.

```text
data/d04_te.dat
data/d14_te.dat
```

각 데이터는 다음과 같이 구성됩니다.

- 960 time points
- 52 process variables
- 41 XMEAS variables
- 11 XMV variables
- 3-minute sampling interval
- 약 48시간의 시계열

Fault는 8시간 시점부터 발생합니다.

---

## 13. Data Source & License

Original repository:

`https://github.com/jkitchin/tennessee-eastman-profbraatz`

License:

**BSD-3-Clause License**

공개된 TEP reference data를 분석 목적으로 사용했습니다.

---

## 14. Missing Values & Outliers

두 데이터에서 결측치는 발견되지 않았습니다.

Fault 이후 나타나는 큰 온도 변화와 반복 진동은 일반적인 분석에서는 이상치처럼 보일 수 있지만 본 프로젝트에서는 바로 이러한 변화가 분석 대상입니다.

따라서 값이 크다는 이유만으로 제거하지 않았습니다.

---

## 15. AI Use & Verification

생성형 AI는 다음 작업의 보조도구로 사용했습니다.

- 분석 개념 학습
- Python 코드 구현 보조
- Recovery / Settling Time 계산방법 검토
- IAE / ISE 개념 이해
- Cross-correlation 및 Autocorrelation 구현
- FFT 해석과 sampling 한계 확인
- 보고서 구조 검토

AI가 제안한 분석 결과를 그대로 사용하지 않고 모든 코드를 로컬 Jupyter 환경에서 직접 실행하여 수치와 그래프를 확인했습니다.

예를 들어 온도와 압력 peak 사이의 6분 차이를 처음에는 시간지연으로 생각했지만 Cross-correlation 결과가 이를 충분히 지지하지 않아 초기 해석을 수정했습니다.

자세한 AI 활용 기록과 검증 과정은 [`REPORT.md`](REPORT.md)에 정리했습니다.

---

## 16. Repository Structure

```text
tep-cooling-fault-analysis/
├── analysis.ipynb
├── README.md
├── REPORT.md
├── requirements.txt
├── data/
│   ├── d04_te.dat
│   └── d14_te.dat
└── images/
    ├── 02_moving_average.png
    ├── 04_control_response.png
    ├── 07_temperature_recovery.png
    ├── 08_control_burden.png
    ├── 09_normalized_fault_comparison.png
    ├── 10_temperature_fault_comparison.png
    └── 11_fault14_autocorrelation.png
```

---

## 17. Reproducibility

필요한 라이브러리는 `requirements.txt`에 기록했습니다.

```bash
pip install -r requirements.txt
```

Jupyter Notebook을 실행합니다.

```bash
jupyter notebook
```

`analysis.ipynb`를 첫 번째 셀부터 순서대로 실행하면 주요 분석 결과를 재현할 수 있습니다.

---

## 18. Detailed Report

분석 과정, 지표 정의, 수치 결과, 분석의 한계 및 AI 사용 기록은 아래 보고서에서 확인할 수 있습니다.

**[REPORT.md](REPORT.md)**

---

## 19. Main Conclusion

공정 출력값이 순간적으로 정상범위로 돌아온 것과 실제 공정이 안정적으로 회복된 것은 서로 다를 수 있습니다.

또한 같은 냉각계 관련 고장이라도 원인에 따라 순간 spike, 지속적인 진동, 제어신호의 움직임 등 서로 다른 시계열 특징이 나타났습니다.

따라서 공정 이상을 평가할 때 현재 센서값 하나만 확인하기보다 회복시간, 안정화 여부, 누적 오차, 정상범위 체류율, 조작변수의 움직임을 함께 확인할 필요가 있습니다.