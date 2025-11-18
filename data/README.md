# DATA

## Raw Data

데이터 설명: 항공사 승객 만족도 조사 데이터로, 103,904명의 승객을 대상으로 수집되었습니다. 각 승객의 인구통계학적 특성, 여행 정보, 서비스 평가 항목(1-5점 척도), 그리고 최종 만족도(만족/불만족)가 포함되어 있습니다. 이 데이터를 통해 어떤 요인이 승객 만족도에 영향을 미치는지 분석할 수 있습니다.

### 주요 컬럼 분류

**종속 변수**:
- `satisfaction`: satisfied / neutral or dissatisfied

**승객 특성**:
- `Gender`: Male/Female
- `Customer Type`: Loyal/disloyal Customer
- `Age`: 나이
- `Type of Travel`: Personal Travel / Business travel

**비행 정보**:
- `Class`: Eco, Eco Plus, Business
- `Flight Distance`: 비행 거리 (마일 단위)
- `Departure Delay in Minutes`: 출발 지연 시간 (분)
- `Arrival Delay in Minutes`: 도착 지연 시간 (분)

**서비스 평가 (1-5점 척도)**:
- `Inflight wifi service`: 기내 Wi-Fi 서비스
- `Departure/Arrival time convenient`: 출발/도착 시간 편리성
    - 해당 컬럼은 0~5점 척도이니 전처리 필요
- `Ease of Online booking`: 온라인 예약 편의성
- `Gate location`: 탑승구 위치 편리성
- `Food and drink`: 기내 음식 및 음료
- `Online boarding`: 온라인 탑승 수속
- `Seat comfort`: 좌석 편안함
- `Inflight entertainment`: 기내 엔터테인먼트
- `On-board service`: 기내 서비스
- `Leg room service`: 다리 공간
- `Baggage handling`: 수하물 처리
- `Checkin service`: 체크인 서비스
- `Inflight service`: 기내 전반 서비스
- `Cleanliness`: 청결도