# EC2 Capacity Blocks for ML (Trainium2)

AWS Trainium, P 시리즈와 같은 고성능 인스턴스는 가용한 온디맨드 수량이 없다면 **Capacity Block(CB)**을 통해 미리 예약해야 사용 할 수 있습니다. 이 가이드는 CB를 구매하고 활성화하는 전체 과정을 설명합니다.
본 가이드는 trn2.48xlarge 를 기준으로 하지만 전체 과정은 P 인스턴스 타입도 동일하게 적용 됩니다.

## 1. Capacity Block 검색 및 구매

콘솔의 **EC2 > Capacity Reservations > Purchase Capacity Blocks for ML** 메뉴로 진입합니다.




### Step 1: 인스턴스 및 기간 선택
원하는 인스턴스 타입(`trn2.48xlarge`)과 기간, 시작 날짜를 선택합니다.
*(스크린샷: Page 2 - Find Capacity Block 화면)*
> **Tip:** Trainium2는 2025-12 기준 특정 리전(예: us-east-2)에서만 제공될 수 있으므로 리전을 잘 확인하세요.

### Step 2: 가능한 블록 확인 및 선택
가능한 시간대와 가격을 확인합니다. CB는 **특정 Availability Zone(AZ)**에 고정되어 제공됩니다.
*(스크린샷: Page 3 - 검색 결과 화면)*
> **중요:** 여기서 배정받은 AZ (예: `us-east-2b`)를 반드시 기억해야 합니다. 추후 이 AZ에 서브넷을 만들어야 인스턴스를 띄울 수 있습니다.

### Step 3: 구매 확정 (Confirm)
가격과 시간을 최종 확인하고, 텍스트 입력창에 `confirm`을 입력하여 구매를 확정합니다.
*(스크린샷: Page 5 - confirm 입력 화면)*

---

## 2. 예약 상태 확인 (Scheduled vs Active)

구매가 완료되면 상태가 변경됩니다.

* **Scheduled (예정됨):** 구매는 성공했으나, 아직 시작 시간이 되지 않은 상태입니다.
    *(스크린샷: Page 6 - Scheduled 상태)*
* **Active (활성):** 예약 시간이 되어 인스턴스를 실행할 수 있는 상태입니다.
    *(스크린샷: Page 7 - Active 상태)*

---

## 3. (중요) 네트워크 구성 주의사항

Capacity Block은 구매 시점에 **AZ(가용 영역)가 지정**됩니다. (예: `us-east-2b`)
따라서 인스턴스를 띄울 때 **반드시 해당 AZ에 있는 서브넷**을 사용해야 합니다.

> 스크린샷의 예시에서는 **`us-east-2b`** 가 할당되었으므로, VPC 생성 시 해당 구역에 서브넷을 생성해야 합니다.
