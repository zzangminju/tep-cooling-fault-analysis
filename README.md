# TEP Cooling-System Fault Analysis

**Dynamic Response, Process Resilience, and Control Performance Analysis of Fault 4 and Fault 14**

Tennessee Eastman Process(TEP)의 냉각계 관련 고장인 **Fault 4**와 **Fault 14**를 대상으로 고장 유형에 따른 반응기 동적 응답, 공정 회복력, 온도 제어성능 및 제어부담을 비교한 시계열 분석 프로젝트입니다.

단순히 Fault 발생 전후의 그래프를 비교하는 데 그치지 않고, 정상 운전 데이터를 기준으로 여러 성능지표를 직접 정의하여 두 고장의 영향을 정량적으로 비교했습니다.

---

## 1. Project Overview

본 프로젝트에서는 TEP의 두 냉각계 관련 Fault를 분석했습니다.

- **Fault 4:** Reactor Cooling Water Inlet Temperature Step
- **Fault 14:** Reactor Cooling Water Valve Sticking

두 Fault는 모두 반응기 냉각계와 관련되어 있지만 고장의 성격이 다릅니다.

Fault 4는 냉각수 입구 조건 자체가 변하는 **외란(disturbance)**에 해당합니다.

반면 Fault 14는 냉각수 밸브가 원활하게 움직이지 않는 **밸브 동작 이상(valve sticking)**에 해당합니다.

따라서 본 프로젝트에서는 같은 냉각계 관련 이상이라도 고장 원인에 따라 반응기 온도와 제어계에서 서로 다른 시간적 패턴이 나타나는지를 분석했습니다.

---

## 2. Research Questions

본 프로젝트에서는 다음 다섯 가지 질문을 설정했습니다.

1. Fault 4와 Fault 14는 반응기 온도에서 어떤 서로 다른 동적 반응을 보이는가?
2. 두 고장은 정상범위로 얼마나 빠르게 복귀하며, 실제로 안정적인 상태에 정착하는가?
3. 고장 이후 반응기 온도 제어성능은 정상 운전에 비해 얼마나 악화되는가?
4. 냉각수 조작변수에 요구되는 제어부담은 고장 유형에 따라 어떻게 달라지는가?
5. Peak 시점 차이와 반복진동 특성은 추가적인 시계열 분석기법으로도 확인되는가?

---

## 3. Data

Tennessee Eastman Process reference test data를 사용했습니다.

### Data Source

Original repository:

https://github.com/jkitchin/tennessee-eastman-profbraatz

사용 파일:

- `d04_te.dat`
- `d14_te.dat`

각 파일은 하나의 Fault test run을 나타냅니다.

### Data Structure

- 960 time points
- 52 process variables
- 41 measured variables (`XMEAS`)
- 11 manipulated variables (`XMV`)
- Sampling interval: 3 minutes
- Total duration: approximately 48 hours
- Fault onset: 8 hours

### Main Variables

| Variable | Description |
|---|---|
| `XMEAS(7)` | Reactor Pressure |
| `XMEAS(9)` | Reactor Temperature |
| `XMEAS(21)` | Reactor Cooling Water Outlet Temperature |
| `XMV(10)` | Reactor Cooling Water manipulated variable |

특히 `XMEAS(9)`는 반응기의 온도 회복성과 제어성능을 평가하는 주요 출력변수로 사용했습니다.

`XMV(10)`은 냉각계 제어신호가 정상상태와 비교해 얼마나 크게 움직이는지를 평가하는 데 사용했습니다.

---

## 4. Baseline Definition

각 Fault 파일은 서로 다른 simulation run이므로 각 데이터의 Fault 발생 이전 구간을 이용해 별도의 정상상태 baseline을 설정했습니다.

정상구간:

```text
0 h ≤ time < 8 h
```

반응기 온도의 정상 통계범위는 다음과 같이 정의했습니다.

```text
Normal Range = Normal Mean ± 2 × Normal Standard Deviation
```

### Fault 4

- Normal mean: `120.3993 °C`
- Normal standard deviation: `0.01917 °C`
- Normal range: `120.3610 ~ 120.4377 °C`

### Fault 14

- Normal mean: `120.3999 °C`
- Normal standard deviation: `0.01863 °C`
- Normal range: `120.3626 ~ 120.4371 °C`

본 프로젝트에서 사용한 `mean ± 2σ` 범위는 실제 산업 안전기준이 아니라 두 Fault의 회복성을 비교하기 위해 정의한 **통계적 정상범위**입니다.

---

## 5. Analysis Methods

본 프로젝트에서는 다음 시계열 및 공정제어 분석 방법을 사용했습니다.

### Basic Time-Series Analysis

- Moving Average
- First Difference
- Rolling Standard Deviation

### Process Recovery Analysis

- Statistical Normal Range
- Recovery Time
- Settling Time
- In-range Ratio

### Control Performance Analysis

- Integral Absolute Error (IAE)
- Integral Squared Error (ISE)

### Control Burden Analysis

- XMV(10) deviation from normal baseline
- Cumulative absolute control deviation
- Normalized Control Burden

### Additional Time-Series Analysis

- Peak Lag Analysis
- Cross-correlation
- Differenced Cross-correlation
- Autocorrelation
- Fast Fourier Transform (FFT)

---

## 6. Key Metrics

### Recovery Time

Fault 발생 후 반응기 온도가 처음으로 정상범위에 다시 진입하는 데 걸린 시간입니다.

### Settling Time

단순히 정상범위에 한 번 진입하는 것만으로 회복되었다고 판단하지 않고,

```text
정상범위 안에서 최소 30분 연속 유지
```

하는 시점을 안정적인 settling으로 정의했습니다.

데이터가 3분 간격이므로 연속 10개 관측값이 정상범위에 포함되는 경우입니다.

### In-range Ratio

Fault 발생 직후 동일한 2시간 분석구간

```text
8.00 h ≤ time < 10.00 h
```

에서 전체 관측값 중 정상범위 안에 존재한 비율입니다.

### IAE

IAE는 **Integral Absolute Error**의 약자입니다.

정상 평균에서 벗어난 온도 오차의 절댓값을 시간에 따라 누적합니다.

```text
IAE = ∫ |Temperature - Normal Mean| dt
```

온도가 정상값에서 얼마나 많이, 얼마나 오래 벗어나 있었는지를 평가합니다.

### ISE

ISE는 **Integral Squared Error**의 약자입니다.

```text
ISE = ∫ (Temperature - Normal Mean)² dt
```

오차를 제곱하기 때문에 큰 순간적 온도 이탈에 더 큰 가중치를 부여합니다.

### Control Burden

XMV(10)의 정상 평균에서 벗어난 절대편차를 시간에 따라 누적한 값을 본 프로젝트에서는 Control Burden으로 정의했습니다.

이는 실제 냉각수 사용량이나 에너지 소비량이 아니라,

> 냉각계 제어신호가 정상 운전 수준에서 얼마나 크게 벗어나 움직였는가

를 나타내는 상대적인 분석 지표입니다.

---

## 7. Key Results

| Metric | Fault 4 | Fault 14 |
|---|---:|---:|
| Recovery Time | 3 min | 9 min |
| Settling Time | 57 min | Not settled |
| In-range Ratio | 77.5% | 2.5% |
| IAE | 0.0494 | 0.4415 |
| ISE | 0.00333 | 0.12130 |
| Control Burden | 7.364 | 12.679 |
| IAE / Normal | 1.66× | 15.08× |
| ISE / Normal | 4.56× | 175.81× |
| Control Burden / Normal | 8.34× | 15.60× |

---

## 8. Fault 4: Short and Strong Disturbance

Fault 4 발생 직전인 7.95 h에서 반응기 온도는 `120.37 °C`였습니다.

Fault 발생 시점인 8.00 h에서는 `120.60 °C`까지 상승했습니다.

3분 동안의 온도 변화는:

```text
+0.23 °C
```

였으며 전체 960개 시점에서 가장 큰 양의 1-step temperature change였습니다.

다음 시점인 8.05 h에서는 다시 `120.40 °C`까지 감소하여:

```text
-0.20 °C
```

의 변화를 보였습니다.

따라서 Fault 4는 장기간 큰 온도편차가 지속되는 형태라기보다 **짧고 강한 순간적 온도 충격**의 특징을 보였습니다.

### Recovery

- Recovery Time: **3 min**
- Settling Time: **57 min**
- First 2-hour In-range Ratio: **77.5%**

반응기 온도는 3분 만에 처음 정상범위에 재진입했지만 이후 다시 정상범위를 이탈했습니다.

30분 연속 정상범위 유지 조건을 적용하면 Fault 발생 후 57분이 지나서야 안정적인 settling 구간이 시작되었습니다.

따라서 최초 정상범위 복귀와 실제 안정적인 회복은 동일하지 않았습니다.

---

## 9. Fault 14: Persistent Oscillation

Fault 14에서는 Fault 발생 이후 반응기 온도가 정상 평균 주변에서 큰 폭으로 상승과 하강을 반복했습니다.

Fault 발생 9분 후 처음 정상범위에 재진입했지만 이후 다시 반복적으로 정상범위를 벗어났습니다.

### Recovery

- Recovery Time: **9 min**
- Settling Time: **Not settled**
- First 2-hour In-range Ratio: **2.5%**

30분 연속 정상범위 유지 조건을 적용했을 때 Fault 이후 관측기간에서 settling이 확인되지 않았습니다.

첫 2시간 동안 정상범위에 존재한 관측값도 전체의 2.5%에 불과했습니다.

따라서 Fault 14는 일시적으로 정상범위에 진입하는 순간은 존재하지만 지속적인 정상 운전 상태를 회복하지 못하는 특성을 보였습니다.

---

## 10. Control Performance

Fault 발생 후 첫 2시간 IAE는 다음과 같습니다.

- Fault 4: `0.0494 °C·h`
- Fault 14: `0.4415 °C·h`

Fault 14의 절대 IAE는 Fault 4보다 약 8.94배 높았습니다.

각 Fault의 정상 운전 수준과 비교하면:

- Fault 4 IAE / Normal: **1.66×**
- Fault 14 IAE / Normal: **15.08×**

따라서 Fault 14에서는 정상 온도에서 벗어난 상태가 훨씬 지속적으로 나타났습니다.

Fault 발생 후 첫 2시간 ISE는:

- Fault 4: `0.00333 °C²·h`
- Fault 14: `0.12130 °C²·h`

Fault 14의 절대 ISE는 Fault 4보다 약 36.4배 높았습니다.

정상 운전 수준과 비교하면:

- Fault 4 ISE / Normal: **4.56×**
- Fault 14 ISE / Normal: **175.81×**

ISE는 큰 온도오차에 더 큰 가중치를 부여하기 때문에 Fault 14에서 반복적으로 발생한 큰 온도 진동이 매우 강하게 반영되었습니다.

`175.81×`라는 값은 온도가 175배 증가했다는 뜻이 아니라 **ISE라는 제곱오차 기반 지표가 정상상태보다 그만큼 크게 증가했다는 의미**입니다.

---

## 11. Control Burden

Fault 4 발생 순간 XMV(10)은:

```text
40.694 → 47.248
```

로 증가했습니다.

약 **16.1% 증가**한 값입니다.

Fault 4의 정상 XMV(10) 평균은 약 `41.14`, Fault 이후 평균은 약 `44.89`로 Fault 이후 약 **9.1% 높은 수준**이 유지되었습니다.

Fault 후 첫 2시간 누적 Control Burden은:

- Fault 4: **7.364**
- Fault 14: **12.679**

정상 2시간 Control Burden과 비교하면:

- Fault 4: **8.34×**
- Fault 14: **15.60×**

정상 수준을 제외한 Excess Control Burden은:

- Fault 4: **6.481**
- Fault 14: **11.866**

Fault 4에서는 XMV(10)이 정상보다 높은 방향으로 이동한 뒤 높은 수준을 유지하는 형태가 나타났습니다.

반면 Fault 14에서는 XMV(10)이 정상 기준을 중심으로 높은 값과 낮은 값을 반복하면서 크게 왕복했습니다.

따라서 두 고장은 온도뿐 아니라 제어신호에서도 서로 다른 동적 signature를 나타냈습니다.

---

## 12. Normalized Performance Degradation

각 지표를 자체 정상상태 대비 비율로 비교하면 다음과 같습니다.

| Metric | Fault 4 | Fault 14 |
|---|---:|---:|
| IAE / Normal | 1.66× | 15.08× |
| ISE / Normal | 4.56× | 175.81× |
| Control Burden / Normal | 8.34× | 15.60× |

Fault 14는 세 지표 모두에서 Fault 4보다 큰 성능 저하를 보였습니다.

특히 ISE의 큰 증가는 Fault 14에서 큰 온도편차가 반복적으로 발생했다는 사실을 반영합니다.

IAE, ISE, Control Burden은 계산식과 단위가 서로 다르므로 서로의 절대적인 숫자 크기를 직접 비교하지 않고 **각 지표 안에서 Fault 4와 Fault 14를 비교**했습니다.

---

## 13. Lag Analysis

Fault 4에서 반응기 온도의 최대값은 `8.00 h`, 반응기 압력의 최대값은 `8.10 h`에서 나타났습니다.

두 peak 사이에는 약 **6분**의 차이가 있었습니다.

처음에는 이를 압력이 온도보다 약 6분 늦게 반응하는 현상으로 가정했습니다.

그러나 peak 두 개만으로 일정한 전달지연을 결론내리는 것은 충분하지 않다고 판단하여 cross-correlation을 추가 수행했습니다.

### Raw Cross-correlation

7.5~10 h 구간에서 온도와 압력을 -30~+30분 범위로 이동하면서 correlation을 계산했습니다.

- 0 min lag correlation: `0.241`
- 6 min lag correlation: `0.277`
- Maximum correlation: `0.360`
- Maximum-correlation lag: `24 min`

상관 자체가 강하지 않았습니다.

### Differenced Cross-correlation

장기적인 수준의 영향을 줄이기 위해 온도 변화량과 압력 변화량을 이용해 다시 분석했습니다.

- Strongest lag: `0 min`
- Correlation: `0.274`

음의 correlation까지 절댓값으로 확인했지만 이보다 강한 관계는 발견되지 않았습니다.

따라서 온도 peak와 압력 peak 사이에 6분 차이는 관찰되었지만 **두 변수 사이에 일정한 6분의 고정 전달지연이 존재한다는 근거는 충분하지 않았습니다.**

초기 가설은 추가 분석 결과에 따라 수정했습니다.

---

## 14. Fault 14 Oscillation Analysis

Fault 14의 반복적인 온도 진동 특성을 추가 분석했습니다.

### Autocorrelation

Fault 발생 후 8~20 h의 반응기 온도를 이용하여 autocorrelation을 계산했습니다.

주요 positive peak가 반복적으로 관찰되었습니다.

예를 들어:

- 33 min: `0.936`
- 60 min: `0.872`
- 66 min: `0.846`
- 93 min: `0.880`
- 126 min: `0.885`
- 159 min: `0.852`
- 192 min: `0.802`
- 219 min: `0.814`

이는 Fault 14의 온도 변화가 단순한 random noise가 아니라 반복 구조를 포함하고 있음을 보여줍니다.

### FFT

FFT 분석에서 가장 큰 frequency component는:

- Dominant frequency: **9.08 cycles/hour**
- Corresponding period: **6.61 min**

으로 나타났습니다.

그러나 데이터의 sampling interval은 3분입니다.

따라서 sampling frequency는 시간당 20회이며 Nyquist frequency는:

```text
10 cycles/hour
```

입니다.

FFT에서 확인된 `9.08 cycles/hour`는 Nyquist limit에 매우 가깝습니다.

또한 6.61분 주기당 실제 관측값은 약:

```text
6.61 / 3 ≈ 2.2 samples
```

에 불과합니다.

따라서 정확한 물리적 진동주기가 6.61분이라고 단정하지 않았습니다.

대신 Fault 14에서는 **빠르고 반복적인 온도 oscillation이 강하게 나타나지만 현재 3분 sampling interval로 정확한 주기를 추정하는 데에는 한계가 있다**고 해석했습니다.

---

## 15. Main Findings

### 1. Recovery와 Settling은 서로 다르다.

Fault 4는 3분 만에 처음 정상범위에 재진입했지만 안정적인 settling까지는 57분이 걸렸습니다.

Fault 14도 9분 만에 정상범위에 한 번 진입했지만 이후 지속적으로 정상범위를 이탈하여 settling이 확인되지 않았습니다.

따라서 한 시점에서 정상범위로 돌아오는 것만으로 공정 전체의 회복을 판단하기 어렵습니다.

### 2. 정상적인 출력값이 정상적인 제어상태를 의미하지 않는다.

Fault 4에서 반응기 온도는 빠르게 정상 수준으로 복귀했지만 첫 2시간 Control Burden은 정상 운전의 약 8.34배였습니다.

따라서 process output이 정상처럼 보이더라도 manipulated variable에서는 추가적인 제어부담이 계속 존재할 수 있습니다.

### 3. 같은 냉각계 Fault라도 서로 다른 signature를 나타낸다.

Fault 4는 순간적으로 강한 온도 spike 이후 회복되는 특성을 보였습니다.

Fault 14는 지속적인 온도 oscillation과 큰 제어신호 변동을 보였습니다.

따라서 같은 냉각계에서 발생하는 고장이라도 원인에 따라 서로 다른 동적 패턴을 보일 수 있습니다.

### 4. Peak 시점만으로 동적 관계를 판단하기 어렵다.

온도와 압력 peak 사이에는 6분의 차이가 있었지만 cross-correlation 분석에서는 일정한 6분 lag가 확인되지 않았습니다.

따라서 단일 peak 관찰과 전체 시계열의 동적 관계를 구분해야 합니다.

---

## 16. Chemical Engineering Relevance

본 프로젝트의 분석 접근은 실제 화학공정의 여러 문제와 연결될 수 있습니다.

### Process Monitoring

주요 측정변수가 정상범위에 있더라도 manipulated variable이 평소보다 크게 움직이고 있다면 공정이 정상상태를 유지하기 위해 추가적인 제어노력을 사용하고 있을 가능성이 있습니다.

### Fault Diagnosis

고장 유형별 dynamic signature를 정의하면 향후 이상상태를 분류하거나 고장 원인을 추정하는 데 활용할 수 있습니다.

### Predictive Maintenance

밸브나 냉각계 등의 조작변수가 정상상태보다 지속적으로 크게 움직이는 패턴은 설비 이상이나 성능 저하를 탐색하는 지표로 확장할 수 있습니다.

### Process Control

Recovery Time, Settling Time, IAE 및 ISE를 이용하면 단순히 현재 온도가 정상인지 확인하는 것보다 제어시스템의 동적 성능을 더 자세히 평가할 수 있습니다.

### Process Safety

반응기 냉각계 이상은 온도제어와 직접 연결되므로 공정변수가 잠시 정상으로 돌아왔는지만 보는 것이 아니라 실제로 안정적인 상태를 유지하는지 확인하는 것이 중요합니다.

---

## 17. Limitations

본 프로젝트에는 다음과 같은 한계가 있습니다.

### Single Run Analysis

각 Fault에 대해 하나의 test simulation run을 분석했습니다.

따라서 현재 결과를 모든 Fault 4 및 Fault 14 상황에 그대로 일반화할 수 없습니다.

향후 여러 독립 run에 동일한 KPI를 적용하여 평균과 분산을 비교할 필요가 있습니다.

### Statistical Normal Range

정상범위를 `mean ± 2σ`로 정의했습니다.

이는 실제 산업 안전기준이나 공식 TEP control specification이 아니라 본 분석을 위한 통계적 기준입니다.

### Settling Criterion

Settling을 `30분 연속 정상범위 유지`로 정의했습니다.

이 역시 실제 산업 표준이 아니라 두 Fault를 동일한 기준으로 비교하기 위해 설정한 분석 기준입니다.

### Control Burden

Control Burden은 실제 냉각수 소비량, 에너지 사용량 또는 비용을 의미하지 않습니다.

XMV(10)의 정상 평균 대비 누적 절대편차로 정의한 상대적 분석 지표입니다.

### Correlation and Causality

Cross-correlation은 변수 간 관계를 탐색하는 방법이며 인과관계를 증명하지 않습니다.

TEP는 여러 공정변수와 제어루프가 연결된 복잡한 시스템이므로 온도와 압력 두 변수만으로 직접적인 인과관계를 확정할 수 없습니다.

### FFT Resolution

Fault 14 FFT에서 약 6.6분의 dominant period가 계산되었지만 3분 sampling interval로 인해 Nyquist limit에 매우 가까웠습니다.

따라서 정확한 물리적 진동주기로 단정하지 않고 보조적인 분석 결과로 사용했습니다.

---

## 18. AI Usage

AI는 분석 결과를 자동 생성하는 도구가 아니라 **학습 및 구현 보조 도구**로 사용했습니다.

주요 활용 내용은 다음과 같습니다.

- 시계열 분석 개념 설명
- Python 코드 구조 작성 보조
- Recovery Time 및 Settling Time 구현
- IAE 및 ISE 구현
- Control Burden 정의 및 구현
- Cross-correlation 구현
- Autocorrelation 구현
- FFT 구현
- 결과 해석 검토

모든 코드는 로컬 Python/Jupyter 환경에서 직접 실행했습니다.

AI가 제안한 해석 역시 그대로 사용하지 않고 실제 데이터 출력과 그래프를 확인한 뒤 검증했습니다.

특히 온도와 압력 peak 사이의 6분 차이를 처음에는 고정된 lag일 가능성이 있다고 판단했지만, cross-correlation을 추가 수행한 결과 강한 지연관계가 확인되지 않아 초기 해석을 수정했습니다.

또한 Fault 14 FFT에서 약 6.6분의 dominant period가 계산되었으나 sampling interval과 Nyquist limit을 확인한 뒤 정확한 물리적 주기로 단정하지 않았습니다.

---

## 19. Project Structure

```text
tep-cooling-fault-analysis/
├── analysis.ipynb
├── REPORT.md
├── README.md
├── requirements.txt
├── data/
│   ├── d04_te.dat
│   └── d14_te.dat
└── images/
```

---

## 20. Environment

- Python 3.10+
- NumPy
- pandas
- Matplotlib
- Jupyter

Dependencies are listed in `requirements.txt`.

---

## 21. Installation

Clone the repository:

```bash
git clone https://github.com/zzangminju/tep-cooling-fault-analysis.git
cd tep-cooling-fault-analysis
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate the virtual environment on Windows:

```bash
.venv\Scripts\activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Run `analysis.ipynb` from the first cell to reproduce the analysis.

---

## 22. Data Source and License

The Tennessee Eastman Process reference data used in this project were obtained from:

```text
https://github.com/jkitchin/tennessee-eastman-profbraatz
```

The original repository distributes the material under the **BSD-3-Clause License**.

This project retains the original data source and license information for reproducibility and attribution.

---

## 23. Conclusion

Fault 4와 Fault 14는 모두 반응기 냉각계와 관련된 이상이지만 공정에 남기는 시계열 signature는 크게 달랐습니다.

### Fault 4

- Short and strong temperature spike
- Recovery Time: **3 min**
- Settling Time: **57 min**
- In-range Ratio: **77.5%**
- IAE / Normal: **1.66×**
- ISE / Normal: **4.56×**
- Control Burden / Normal: **8.34×**

### Fault 14

- Persistent temperature oscillation
- Recovery Time: **9 min**
- Settling Time: **Not settled**
- In-range Ratio: **2.5%**
- IAE / Normal: **15.08×**
- ISE / Normal: **175.81×**
- Control Burden / Normal: **15.60×**

본 프로젝트의 핵심 결론은 다음과 같습니다.

> **공정 출력값이 순간적으로 정상범위에 복귀하는 것과 실제 공정이 안정적인 운전상태를 회복하는 것은 서로 다르다. 따라서 공정 이상을 평가할 때는 측정변수의 현재값뿐 아니라 Recovery, Settling, 누적 오차 및 제어부담과 같은 동적 특성을 함께 평가해야 한다.**

자세한 분석 과정과 결과 해석은 [`REPORT.md`](REPORT.md)를 참고하세요.