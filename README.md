# TEP Cooling-System Fault Analysis

Tennessee Eastman Process(TEP)의 두 냉각계 이상인 **Fault 4**와 **Fault 14**를 비교한 시계열 분석 프로젝트입니다.

같은 냉각계 이상이라도 두 고장이 반응기 온도와 제어시스템에 서로 다른 영향을 주는지 확인했습니다.

- **Fault 4**: 냉각수 입구 온도가 갑자기 변하는 이상
- **Fault 14**: 냉각수 밸브가 원활하게 움직이지 않는 이상

## 한눈에 보는 결과

![Fault 4 vs Fault 14 Temperature](images/10_temperature_fault_comparison.png)

Fault 4는 고장 직후 온도가 크게 한 번 튄 뒤 비교적 빠르게 회복했습니다.

반면 Fault 14는 고장 이후 온도가 계속 위아래로 흔들렸습니다.

| Metric | Fault 4 | Fault 14 |
|---|---:|---:|
| 처음 정상범위로 복귀 | 3 min | 9 min |
| 안정적으로 정착 | 57 min | 확인되지 않음 |
| 첫 2시간 정상범위 체류율 | 77.5% | 2.5% |
| IAE / Normal | 1.66× | 15.08× |
| ISE / Normal | 4.56× | 175.81× |
| Control Burden / Normal | 8.34× | 15.60× |

## 1. 정상범위로 돌아오는 것과 안정화는 다르다

![Temperature Recovery](images/07_temperature_recovery.png)

Fault 4의 온도는 고장 발생 **3분 후 처음 정상범위에 들어왔습니다.**

하지만 이후 다시 정상범위를 벗어났고, 30분 이상 계속 정상범위에 머무르기 시작한 것은 **57분 후**였습니다.

즉,

> **잠깐 정상값으로 돌아오는 것과 공정이 실제로 안정되는 것은 다릅니다.**

## 2. 온도가 정상이어도 제어기는 더 많이 움직일 수 있다

![Control Burden](images/08_control_burden.png)

Fault 4에서는 온도가 빠르게 회복됐지만 냉각수 관련 제어변수인 XMV(10)은 평소보다 크게 움직였습니다.

Fault 이후 첫 2시간의 제어신호 누적편차는 정상 운전보다:

- Fault 4: **8.34배**
- Fault 14: **15.60배**

높았습니다.

여기서 Control Burden은 실제 냉각수 사용량이 아니라, **제어신호가 평소 수준에서 얼마나 크게 벗어나 움직였는지 나타내는 지표**입니다.

## 3. Fault 14의 영향이 훨씬 지속적이었다

![Normalized Performance](images/09_normalized_fault_comparison.png)

Fault 14는 정상범위 체류율이 첫 2시간 동안 **2.5%**에 불과했습니다.

또한 정상 운전과 비교했을 때:

- IAE: **15.08배**
- ISE: **175.81배**
- Control Burden: **15.60배**

로 증가했습니다.

ISE가 특히 큰 이유는 이 지표가 **큰 온도 이탈에 더 큰 점수를 주기 때문**입니다.

따라서 “온도가 175배 나빠졌다”는 의미는 아닙니다.

## 핵심 결론

> **공정 온도가 잠깐 정상으로 돌아왔다고 해서 공정 전체가 안정됐다고 볼 수는 없다.**

Fault 4는 큰 순간 변화 이후 회복되는 형태였고, Fault 14는 반복적인 온도 흔들림이 계속되는 형태였습니다.

따라서 공정 이상을 판단할 때는 현재 온도뿐 아니라 **회복시간, 안정화 여부, 온도오차, 제어신호의 움직임**을 함께 확인할 필요가 있습니다.

## 사용한 분석

- Moving Average / First Difference
- Rolling Standard Deviation
- Recovery Time / Settling Time
- In-range Ratio
- IAE / ISE
- Control Burden
- Cross-correlation
- Autocorrelation
- FFT

## Data

Tennessee Eastman Process reference data를 사용했습니다.

- `d04_te.dat` — Fault 4
- `d14_te.dat` — Fault 14
- 960 samples per run
- 52 process variables
- Sampling interval: 3 minutes
- Fault onset: 8 hours

Data source:  
https://github.com/jkitchin/tennessee-eastman-profbraatz

## Files

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

자세한 분석 방법과 한계는 [`REPORT.md`](REPORT.md)에서 확인할 수 있습니다.