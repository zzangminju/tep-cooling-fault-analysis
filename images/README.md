# TEP Fault 4 Time-Series Analysis

Tennessee Eastman Process(TEP)의 Fault 4 데이터를 이용해 **반응기 냉각수 이상 발생 전후의 공정 변화를 시계열 관점에서 분석한 프로젝트**입니다.

단순히 이상 발생 여부를 확인하는 것이 아니라, 반응기 온도와 압력, 냉각수 조작변수가 시간에 따라 어떻게 반응하는지 살펴보고 공정제어 관점에서 의미를 해석하는 것을 목표로 했습니다.

---

## 1. 분석 주제

**냉각수 입구 온도 이상 발생 시 반응기 상태와 제어계의 동적 반응 분석**

TEP의 Fault 4는 **Reactor Cooling Water Inlet Temperature Step Change**에 해당합니다.

이번 분석에서는 Fault 발생 전후의 시계열을 비교해 다음 질문에 답하고자 했습니다.

1. Fault 발생 후 반응기 온도는 어떻게 변하는가?
2. 반응기 냉각수 유량 조작변수는 어떻게 대응하는가?
3. Fault 전후 공정의 변동성은 달라지는가?
4. 반응기 온도가 회복된 이후에도 다른 공정 변수에 영향이 남는가?

---

## 2. 데이터

* 데이터: Tennessee Eastman Process reference test data
* 사용 파일: `d04_te.dat`
* Fault: IDV(4)
* 데이터 포인트: 960개
* 공정 변수: 52개

  * XMEAS 41개
  * XMV 11개
* 샘플링 간격: 3분
* 전체 분석 시간: 48시간
* Fault 발생 시점: 8시간

### 주요 분석 변수

| 변수        | 의미                         |
| --------- | -------------------------- |
| `xmeas_7` | Reactor Pressure           |
| `xmeas_9` | Reactor Temperature        |
| `xmv_10`  | Reactor Cooling Water Flow |

### 데이터 출처

Tennessee Eastman Process 공개 reference dataset

* Repository: `jkitchin/tennessee-eastman-profbraatz`
* Data file: `data/d04_te.dat`

원본 데이터의 라이선스 및 저작권 조건을 확인한 뒤 사용해야 합니다.

---

## 3. 프로젝트 구조

```text
tep-timeseries/
│
├─ data/
│  └─ d04_te.dat
│
├─ images/
│  ├─ 01_reactor_temperature.png
│  ├─ 02_moving_average.png
│  ├─ 03_temperature_change.png
│  ├─ 04_control_response.png
│  ├─ 05_temperature_volatility.png
│  └─ 06_pressure_volatility.png
│
├─ analysis.ipynb
├─ REPORT.md
├─ README.md
└─ requirements.txt
```

---

## 4. 분석 방법

이번 프로젝트에서는 다음과 같은 기본 시계열 분석 기법을 사용했습니다.

### 30분 이동평균

3분 간격 데이터 10개를 이용해 이동평균을 계산했습니다.

센서의 단기적인 변동을 완화하고 반응기 온도의 전체적인 흐름을 확인하기 위해 사용했습니다.

### 변화량

현재 온도와 직전 시점의 온도 차이를 계산했습니다.

이를 통해 Fault 발생 순간의 급격한 온도 변화를 확인했습니다.

### 60분 Rolling Standard Deviation

20개 시점, 즉 최근 60분의 표준편차를 계산했습니다.

평균값뿐 아니라 공정의 변동성이 Fault 전후로 어떻게 달라지는지 확인하기 위해 사용했습니다.

### 구간별 통계 비교

Fault 발생 전후 데이터를 나누어 평균과 표준편차를 비교했습니다.

---

## 5. 주요 결과

### 1) Fault 순간 반응기 온도의 급격한 변화

Fault 직전 반응기 온도는 **120.37°C**였으며, 8시간 시점에서 **120.60°C**까지 상승했습니다.

3분 동안의 변화량은 **+0.23°C**로 전체 960개 시점 중 가장 큰 상승폭이었습니다.

그러나 다음 시점에는 **120.40°C**로 다시 기존 수준에 가까워졌습니다.

### 2) 냉각수 유량의 제어 대응

반응기 냉각수 유량 조작변수 XMV10은 Fault 직전 **40.694**에서 Fault 발생 순간 **47.248**로 증가했습니다.

이는 직전 시점 대비 약 **16.1% 증가**한 값입니다.

또한 XMV10의 평균은

* Fault 이전: **41.14**
* Fault 이후: **44.89**

로 Fault 이후 약 **9.1% 높은 수준**을 유지했습니다.

따라서 반응기 온도 자체는 빠르게 회복했지만, 이를 유지하기 위한 냉각수 제어 수준은 이전과 달라졌음을 확인할 수 있었습니다.

### 3) 온도와 압력의 반응 시간 차이

반응기 온도는 **8.00 h**에 최대값을 기록했지만, 반응기 압력은 **8.10 h**에 최대값 **2716.6 kPa**를 기록했습니다.

두 변수의 peak 사이에는 약 **6분의 시간차**가 있었습니다.

또한 반응기 압력의 표준편차는

* Fault 이전: **5.13 kPa**
* Fault 이후: **7.81 kPa**

로 약 **52.1% 증가**했습니다.

---

## 6. 실행 환경

* Python 3.10 이상
* Jupyter Notebook

주요 라이브러리:

```text
numpy
pandas
matplotlib
jupyter
```

---

## 7. 실행 방법

### 1. 저장소 다운로드

```bash
git clone <repository-url>
cd tep-timeseries
```

### 2. 가상환경 생성

Windows 기준:

```bash
python -m venv .venv
```

가상환경 활성화:

```bash
.venv\Scripts\activate
```

### 3. 라이브러리 설치

```bash
pip install -r requirements.txt
```

### 4. Jupyter Notebook 실행

VSCode에서 `analysis.ipynb`를 열고 위에서부터 순서대로 실행합니다.

또는:

```bash
jupyter notebook
```

을 실행한 뒤 `analysis.ipynb`를 열 수 있습니다.

---

## 8. 결과물

자세한 분석 과정과 시각화, 인사이트는 [`REPORT.md`](REPORT.md)에서 확인할 수 있습니다.

분석 코드는 [`analysis.ipynb`](analysis.ipynb)에 포함되어 있습니다.

---

## 9. 한계

* TEP는 실제 플랜트가 아닌 화학공정 시뮬레이션 데이터입니다.
* 이번 분석은 Fault 4의 단일 test run을 중심으로 수행했습니다.
* 변수 사이의 시간적 관계만으로 직접적인 인과관계를 확정할 수는 없습니다.
* Fault 이후 압력 변동성이 증가했지만, 이후 모든 변동이 Fault 하나만의 영향이라고 단정할 수는 없습니다.

---

## 10. AI 활용

분석 과정에서 AI를 활용해 코드 작성 방향, 시각화 방법, 해석 후보를 검토했습니다.

생성된 결과를 그대로 사용하지 않고 실제 데이터를 직접 실행하여 값과 그래프를 확인했으며, 최종 해석은 계산 결과와 시각화를 근거로 판단했습니다.

구체적인 AI 사용 내용과 검증 방법은 `REPORT.md`의 **AI 사용 로그**에 정리했습니다.
