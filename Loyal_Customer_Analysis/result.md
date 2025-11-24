# 항공사 승객 만족도 분석: 충성 고객의 역설과 서비스 투자 우선순위 탐구

> 본 프로젝트는 103,904명의 항공 승객 데이터를 활용하여 **항공사가 어떤 서비스에 투자해야 승객 만족도를 최대화할 수 있는가**를 규명한다.

특히 단순히 전체 고객을 분석하는 것에서 나아가, **Customer Type(충성 고객 / 비충성 고객)** 이 어떤 요인에 따라 만족도가 달라지는지를 집중적으로 탐구하였다.

분석은 다음과 같은 세 단계로 구성된다.
**1. t-test** : 그룹 간 서비스 평가 점수 차이(평균 차이) 분석
**2. Logistic Regression** : 실제 만족(label)을 결정하는 핵심 요인 분석
**3. Segmentation Deep-dive** : 성별·연령·클래스·거리별 세분화로 투자 우선순위 정교화

## 1. 문제 정의

**"충성 고객의 역설: 충성 고객은 정말 만족하는가, 아니면 더 크게 실망하는가?"**

항공사들은 다양한 Loyalty Program(라운지, 마일리지, 등급 혜택)을 운영하며 “자주 이용하는 고객일수록 만족도가 높고 재구매율이 높을 것이다”라고 가정한다.

그러나 실제 데이터를 보면:

- **Loyal Customer의 만족 비율: 약 47.7% (절반 미만)**
- **Disloyal Customer의 만족 비율: 약 23.7%**

즉,
충성 고객조차 절반 이상이 불만족이며, 비충성 고객은 압도적으로 불만족이다.

따라서 우리는 다음 질문에 답해야 했다.

> “충성 고객과 비충성 고객에게 각각 어떤 서비스를 개선하면 만족도가 올라갈까?”
> “두 그룹은 전혀 다른 요인을 중요하게 생각하는가?”
> “세부 고객군(성별·연령·클래스·거리)에 따라 우선순위가 달라질까?”

## 2. 데이터 전처리

- 만족도(Satisfaction)를 이진(label)로 변환
- 서비스 품질 컬럼을 수치형으로 변환
- AgeGroup, DistanceGroup 생성
- Customer Type을 Loyal / Disloyal로 분리

## 3. t-test 결과 해석 (Loyal vs Disloyal 서비스 점수 비교)

**목적**: 단순히 회귀 계수로 “만족도에 기여하는 중요도”를 본 것이 아니라, **서비스 점수 자체가 충성 고객과 비충성 고객 사이에서 실제로 차이가 나는지**를 검증하는 단계.

이 분석을 통해
-> “비충성 고객이 만족도가 낮은 이유가 정말 서비스 ‘품질이 낮아서’인가?”
-> “아니면 같은 점수여도 충성 고객보다 기준이 높아서 만족도가 떨어지는 것인가?”
를 구분할 수 있음.

### 3-1. 거의 모든 서비스 항목에서 유의미한 차이 존재

p-value가 **모든 서비스에서 매우 작음(0 ~ 10^-14 수준)** → **충성 고객과 비충성 고객의 서비스 평가 점수 평균이 통계적으로 유의하게 다름.**

즉, **충성 고객 ≠ 비충성 고객**
서비스에 대한 인식 자체가 다르게 형성되어 있음을 의미한다.

### 3-2. 충성 고객의 평균 점수가 대부분 더 높음 (Mean Diff > 0)

| 서비스                 | Loyal Mean | Disloyal Mean | 차이(L-D) |
| ---------------------- | ---------- | ------------- | --------- |
| Online boarding        | 3.434      | 2.854         | +0.580    |
| Seat comfort           | 3.539      | 2.994         | +0.544    |
| Inflight entertainment | 3.428      | 3.048         | +0.380    |
| Cleanliness            | 3.339      | 3.054         | +0.284    |
| On-board service       | 3.417      | 3.228         | +0.189    |
| Leg room               | 3.400      | 3.328         | +0.072    |
| Gate location          | 2.973      | 2.993         | -0.020    |
| Inflight wifi service  | 2.814      | 2.811         | +0.003    |

**핵심 흐름**

- 대다수 항목에서 충성 고객이 더 높은 점수를 줌
- 즉, 서비스 품질 자체가 나쁘지 않다는 뜻
- 그럼에도 불구하고 충성 고객의 만족도(라벨)는 50%를 넘지 못함
  -> **“기대치 역설(Higher expectation paradox)”** 가능성이 매우 큼

### 3-3. 예외: Gate location/Inflight service 일부 항목은 비충성 고객이 더 높음

- Gate location: Disloyal Mean이 미세하게 더 높음
- Inflight service: Disloyal Mean이 +0.069 더 높음
  이건 흥미로운 포인트로,
  -> **비충성 고객은 기본 기대가 낮아 기본적인 편의만 제공되어도 점수를 후하게 줄 가능성**이 있다.
  즉, 기대치가 낮으면 실망도 적다는 구조.

### 3-4. 총평: 이 t-test가 주는 중요한 Insight

**1) 충성 고객의 만족도가 50% 이하인 핵심 이유는 ‘기대치가 매우 높기 때문’임**
서비스 품질 자체는 충성 고객에게 더 좋게 평가되는데도 만족도가 낮음.
-> 충성 고객 불만족은 **‘품질 문제’가 아니라 ‘기대-경험 불일치(Expectation Gap)’ 문제**

**2) 비충성 고객의 낮은 만족도는 ‘서비스 품질 자체가 낮기 때문’임**
대부분 항목에서 점수가 낮음.
-> 비충성 고객의 케이스는 **실제 제공되는 서비스 품질 개선을 통해 만족도를 올릴 수 있음**

**3) 서비스 개선 전략이 두 고객군에서 완전히 다르다는 근거 확보**
| 고객군 | 불만족 원인 | 전략 방향 |
| ----- | ---------- | --------- |
| 충성 고객 | 기대 수준이 너무 높음 / 작은 문제에도 민감 | 기대 관리, 프리미엄 서비스 보장, 개인화 |
| 비충성 고객 | 실제 서비스 품질이 낮아서 불만족 | 핵심 서비스 품질 개선(좌석, 탑승, 엔터테인먼트 등) |

## 4. Logistic Regression 분석 (핵심 분석)

**역할: “어떤 서비스가 실제 만족(label=1)을 결정하는가?”**
= 항공사가 실제로 투자해야 하는 서비스 우선순위를 찾는 분석이다.

### 4-1. 충성 고객(Loyal)의 만족 결정 요인 (Top 3)

| 순위 | 서비스          | 회귀 계수 | 해석                             |
| ---- | --------------- | --------- | -------------------------------- |
| 1    | Online boarding | 1.110     | 탑승 과정의 효율성이 가장 중요   |
| 2    | Inflight Wi-Fi  | 0.603     | 업무·생산성·디지털 편의성 중심   |
| 3    | Leg room        | 0.582     | 반복 여행자 → 공간적 편안함 중시 |

**충성 고객 결론**

- 효율성 + 디지털 편의 + 공간적 편안함
- 즉, “자주 타본 사람”에게 중요한 건 좌석 자체가 아니라 전체 탑승 경험의 매끄러움

### 4-2. 비충성 고객(Disloyal)의 만족 결정 요인 (Top 3)

| 순위 | 서비스          | 회귀 계수 | 해석                          |
| ---- | --------------- | --------- | ----------------------------- |
| 1    | Inflight Wi-Fi  | 2.733     | 절대적 영향력                 |
| 2    | Food and drink  | 2.212     | 즉각적·직관적 만족 요소       |
| 3    | Online boarding | 1.858     | 탑승 시 처음 받는 인상이 중요 |

**비충성 고객 결론**

- 즉각적으로 느낄 수 있는 편의 요소가 만족도를 밀어올림
- “생각보다 괜찮네?”라는 경험이 중요
- Wi-Fi가 모든 분석에서 1위로 나타남

### 4-3. Regression 최종 결론

- 충성 고객은 **효율성 중심(Online boarding)**
- 비충성 고객은 **체감 편의 중심(Wi-Fi, Food)**
- 두 그룹 모두 Wi-Fi는 공통적으로 중요하지만, 우선순위의 강도는 매우 다르다

## 5. Deep-Dive 세그먼트 분석

> Deep-Dive 분석은 단순히 “충성/비충성 고객 전체의 서비스 우선순위”를 구하는 단계를 넘어,
> 고객을 실제로 나누는 주요 변수(Gender, AgeGroup, Class, DistanceGroup, Type of Travel)에 따라 투자 우선순위가 어떻게 달라지는지 확인하기 위한 세부 분석이다.
> 이는 항공사가 균일한 서비스 투자 전략이 아닌, 세그먼트별 맞춤 전략을 수립할 수 있게 한다.

분석 대상 세그먼트:

- Gender
- Age
- Class
- Flight Distance
- Type of Travel

### 5-1. Loyal Customer 세그먼트별 Top 3 중요 서비스

#### 1) Gender

| Subgroup   | Top1                          | Top2                           | Top3                          |
| ---------- | ----------------------------- | ------------------------------ | ----------------------------- |
| **Male**   | Online boarding (1.438)       | Inflight entertainment (0.545) | Inflight wifi service (0.532) |
| **Female** | Inflight wifi service (0.777) | Online boarding (0.762)        | Leg room service (0.663)      |

-> 남성은 "온라인 탑승"이 절대적으로 중요하며, 여성은 "기내 Wi-Fi"가 최우선 투자 요인

#### 2) Age

| Age       | Top1                          | Top2                           | Top3                           |
| --------- | ----------------------------- | ------------------------------ | ------------------------------ |
| **10대**  | Online boarding (2.352)       | Food and drink (2.172)         | Inflight wifi service (0.938)  |
| **20대**  | Online boarding (2.228)       | Food and drink (2.100)         | Inflight entertainment (0.435) |
| **30대**  | Online boarding (1.185)       | Inflight entertainment (0.688) | Inflight wifi service (0.467)  |
| **40대**  | Leg room service (0.817)      | Online boarding (0.738)        | Inflight wifi service (0.659)  |
| **50대**  | Leg room service (0.882)      | Inflight wifi service (0.776)  | Online boarding (0.765)        |
| **60대**  | Inflight wifi service (0.923) | Leg room service (0.851)       | Online boarding (0.597)        |
| **70대+** | Leg room service (0.708)      | Inflight entertainment (0.656) | Inflight wifi service (0.611)  |

-> 10~30대는 “온라인 탑승 + 음식 + 엔터테인먼트”
-> 40대 이상에서는 “다리공간 + Wi-Fi” 요구가 강하게 증가

#### 3) Class

| Class        | Top1                          | Top2                           | Top3                    |
| ------------ | ----------------------------- | ------------------------------ | ----------------------- |
| **Eco Plus** | Inflight wifi service (2.267) | Inflight entertainment (0.504) | Online boarding (0.307) |
| **Business** | Leg room service (0.634)      | Checkin service (0.598)        | Online boarding (0.588) |
| **Eco**      | Inflight wifi service (2.508) | Inflight entertainment (0.348) | Online boarding (0.183) |

-> Economy 계층일수록 Wi-Fi 영향이 폭발적으로 크다.
-> Business 클래스는 “여유 공간 + 체크인 경험”이 핵심

#### 4) Flight Distance

| Distance   | Top1                    | Top2                          | Top3                           |
| ---------- | ----------------------- | ----------------------------- | ------------------------------ |
| **Short**  | Online boarding (1.001) | Inflight wifi service (0.914) | Inflight entertainment (0.443) |
| **Medium** | Online boarding (0.909) | Leg room service (0.564)      | On-board service (0.554)       |
| **Long**   | Checkin service (0.681) | Cleanliness (0.588)           | Online boarding (0.571)        |

-> 단거리는 "온라인 탑승/디지털 서비스" 중심,
-> 장거리는 “체크인 경험, 청결, 공간”처럼 “기본 품질” 쪽에 무게가 있다.

#### 5) Type of Travel

| Type                | Top1                          | Top2                     | Top3                   |
| ------------------- | ----------------------------- | ------------------------ | ---------------------- |
| **Personal Travel** | Inflight wifi service (2.649) | Online boarding (0.306)  | Food and drink (0.033) |
| **Business travel** | Checkin service (0.733)       | On-board service (0.683) | Cleanliness (0.625)    |

-> 개인 여행객은 "기내 Wi-Fi → 온라인 편의" 중심
-> 비즈니스 여행객은 “체크인 → 기내 서비스 품질 → 청결”이 핵심 우선순위

### 5-2. Disloyal Customer 세그먼트별 Top 3 중요 서비스

#### 1) Gender

| Gender     | Top1                          | Top2                   | Top3                    |
| ---------- | ----------------------------- | ---------------------- | ----------------------- |
| **Male**   | Inflight wifi service (2.702) | Food and drink (2.049) | Online boarding (1.842) |
| **Female** | Inflight wifi service (2.675) | Food and drink (1.974) | Online boarding (1.687) |

-> 남녀 모두 “Wi-Fi → 음식 → 온라인 탑승” 구조로 거의 동일

#### 2) Age

| Age       | Top1                          | Top2                          | Top3                                      |
| --------- | ----------------------------- | ----------------------------- | ----------------------------------------- |
| **10대**  | Inflight wifi service (2.828) | Online boarding (1.536)       | Inflight entertainment (0.855)            |
| **20대**  | Food and drink (3.135)        | Inflight wifi service (3.078) | Online boarding (2.670)                   |
| **30대**  | Inflight wifi service (2.429) | Online boarding (0.677)       | Food and drink (0.634)                    |
| **40대**  | Inflight wifi service (2.691) | Food and drink (0.426)        | Departure/Arrival time convenient (0.327) |
| **50대**  | Inflight wifi service (2.003) | Food and drink (0.763)        | Online boarding (0.733)                   |
| **60대**  | Inflight wifi service (1.192) | Food and drink (1.036)        | Online boarding (0.881)                   |
| **70대+** | Online boarding (1.607)       | Cleanliness (1.072)           | Inflight service (0.718)                  |

-> 20대 비충성 고객은 “음식 + Wi-Fi”가 절대적
-> 40대는 시간 편의성(지연 관리)이 Top3에 등장하는 것이 특징

#### 3) Class

| Class        | Top1                          | Top2                   | Top3                    |
| ------------ | ----------------------------- | ---------------------- | ----------------------- |
| **Eco Plus** | Inflight wifi service (1.329) | Food and drink (0.540) | Checkin service (0.384) |
| **Business** | Inflight wifi service (2.666) | Food and drink (2.206) | Online boarding (1.918) |
| **Eco**      | Inflight wifi service (2.778) | Food and drink (1.747) | Online boarding (1.609) |

-> 비충성 고객은 클래스와 무관하게 Wi-Fi가 절대적 영향력 1위
-> Business/Eco 모두 음식 중요도가 매우 높다.

#### 4) Flight Distance

| Distance   | Top1                          | Top2                   | Top3                    |
| ---------- | ----------------------------- | ---------------------- | ----------------------- |
| **Short**  | Inflight wifi service (2.698) | Food and drink (2.258) | Online boarding (1.972) |
| **Medium** | Inflight wifi service (2.933) | Food and drink (0.634) | Checkin service (0.305) |

-> 거리와 무관하게 Wi-Fi 영향력이 극도로 높음
-> 단거리에서는 음식 영향력도 매우 높다.

#### 5) Type of Travel

| Type                | Top1                          | Top2                    | Top3                    |
| ------------------- | ----------------------------- | ----------------------- | ----------------------- |
| **Personal Travel** | Inflight wifi service (2.280) | Online boarding (0.773) | Seat comfort (0.634)    |
| **Business Travel** | Inflight wifi service (2.707) | Food and drink (2.150)  | Online boarding (1.843) |

-> 업무/개인 모두 "Wi-Fi → 온라인 → 음식/좌석" 패턴 유지.

### 5-3. Deep-Dive 결과 핵심 요약

**Loyal Customer (충성 고객)**

- 디지털 서비스(특히 Online boarding)는 거의 모든 그룹에서 Top3 유지
- 40대 이상은 Leg room + Wi-Fi의 영향력이 커짐
- Business 고객은 공간(leg room)·체크인 경험이 핵심

**Disloyal Customer (비충성 고객)**

- 모든 세그먼트에서 기내 Wi-Fi가 절대적 1위
- 음식(Food and drink) 영향력도 굉장히 큼
- 비즈니스여행, 장거리, 계층과 무관하게 동일 패턴

## 6. Customer Type 별 서비스 투자 우선순위 종합 분석

### 6-1. Loyal Customer - 투자 우선순위

| 순위    | 서비스명                  | 근거 요약                                                                                     |
| ------- | ------------------------- | --------------------------------------------------------------------------------------------- |
| **1위** | **Online boarding**       | 상관계수·회귀·Deep-dive 모두 안정적으로 상위권. 디지털 경험이 충성 고객에게 강력한 만족 요인. |
| **2위** | **Leg room service**      | 좌석 관련 서비스 중 만족도 영향이 가장 큼. 여러 세그먼트에서 반복적으로 Top3에 등장.          |
| **3위** | **Inflight wifi service** | 기내 Wi-Fi가 특히 여성, Eco Plus·Eco 클래스에서 중요. 회귀/Deep-dive에서 consistently 상위.   |
| **4위** | Cleanliness               | 장거리·고령층 세그먼트에서 중요 영향.                                                         |
| **5위** | Inflight entertainment    | 많은 세그먼트에서 Top2~Top3에 반복 등장.                                                      |

### 6-2. Disloyal Customer - 투자 우선순위

| 순위    | 서비스명                  | 근거 요약                                                                      |
| ------- | ------------------------- | ------------------------------------------------------------------------------ |
| **1위** | **Inflight wifi service** | 압도적으로 가장 중요한 요인. 거의 모든 세그먼트의 Top1. 회귀계수도 압도적 1위. |
| **2위** | **Food and drink**        | 비충성 고객은 기본적 편의 요소의 만족도가 특히 낮음. 여러 세그먼트에서 Top2.   |
| **3위** | **Online boarding**       | 기초 수준의 디지털 경험 개선 필요. 만족도와 상관·회귀 모두 높은 영향.          |
| **4위** | Checkin service           | 평균 점수 차이 및 회귀계수 기준 투자 가치 높음.                                |
| **5위** | Cleanliness               | 고령층·중거리 세그먼트에서 중요하게 나타남.                                    |

## 7. 핵심 발견 요약

### 7-1. 충성 고객(Loyal)도 디지털 서비스 품질에 크게 영향을 받는다.

- 충성 고객 만족도의 핵심은 Online boarding(온라인 탑승 절차의 편의성)
- 기대치가 높은 고객군일수록 “디지털 프로세스의 매끄러움”을 굉장히 강조함
- 이는 비행 경험이 많을수록 기초적 서비스보다 운영 효율성·디지털 UX를 더 평가한다는 의미
  -> **충성 고객 유지 전략 = “디지털 경험(Online boarding / Wi-Fi) + 좌석”이 핵심**

### 7-2. 비충성 고객(Disloyal)은 “기본 서비스 불만족”으로 이탈한다.

- 가장 낮은 평가를 받은 항목들: **Food & drink, Check-in, Inflight service, Cleanliness**
- 회귀·상관·점수·세그먼트 모두에서 **Food & drink**는 비충성 고객에서 최상위 중요도로 등장
- 특히 **Inflight Wi-Fi는 거의 모든 비충성 하위 세그먼트의 Top1**
  -> **비충성 고객 전환 전략 = “기본적 체감 서비스 개선 + 기내 Wi-Fi 제공”**

### 7-3. Loyal vs Disloyal 고객의 구조적 차이

| 구분        | Loyal                                 | Disloyal                                |
| ----------- | ------------------------------------- | --------------------------------------- |
| 주요 관심사 | 디지털 효율성, 좌석 편안함, 기내 경험 | 기본 서비스 퀄리티, 음식, 체크인, Wi-Fi |
| 특징        | 기대치 높고 반복 이용 경험 多         | 단순 경험 기반 평가, 기본 품질에 민감   |
| 전략        | Loyalty 유지 중심                     | 이탈 방지 & 초보 이용자 만족도 상승     |

### 7-4. Deep-dive에서 확인된 “세그먼트별 숨은 패턴”

**Loyal 고객 세그먼트 특징**

- **여성**: Inflight Wi-Fi, Online boarding 비중 높음
- **고령층(60대 이상)**: Leg room service(좌석) 영향 가장 강함
- **Eco/Eco Plus**: Wi-Fi → Entertainment → Boarding 순
- **Personal Travel 고객**: Wi-Fi 중요도 매우 높음

**Disloyal 고객 세그먼트 특징**

- **모든 연령대에서 Wi-Fi가 Top1** (예외 없음)
- **20대·30대**: Food & drink가 강력한 영향
- **40대 이상**: Food & drink + Basic service(체크인, 기내서비스) 문제 반복
- **Business travel**: Wi-Fi + Food & drink 두 요소가 핵심

## 결론: 항공사는 어디에 투자해야 하는가?

#### Loyal Customer 전략 (고객 유지)

**1. Online boarding UX 최적화**
**2. Leg room / 좌석 업그레이드 제공**
**3. 기내 Wi-Fi 고도화**
-> 충성 고객은 “시간 절약 / 효율성 / 편안함” 중심 경험을 원함.

#### Disloyal Customer 전략 (이탈 방지 & 신규 고객 만족도 개선)

**1. Inflight Wi-Fi 품질 개선 = 최우선**
**2. Food & drink 품질 개선**
**3. 체크인 및 기본 서비스 개선**
-> 비충성 고객은 “기본 서비스 품질”이 부족해 만족도가 낮음.
