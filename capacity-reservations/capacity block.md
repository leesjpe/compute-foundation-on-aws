# EC2 Capacity Blocks for ML

AWS Trainium, P 시리즈와 같은 고성능 인스턴스는 가용한 온디맨드 수량이 없다면 **Capacity Block(CB)**을 통해 미리 예약해야 사용 할 수 있습니다. 이 가이드는 CB를 구매하고 활성화하는 전체 과정을 설명합니다.
본 가이드는 trn2.48xlarge 를 기준으로 하지만 전체 과정은 P 인스턴스 타입도 동일하게 적용 됩니다.

## 1. Capacity Block 검색 및 구매

콘솔의 **EC2 > Capacity Reservations > Purchase Capacity Blocks for ML** 메뉴로 진입합니다.
<img width="1330" height="574" alt="cb1" src="https://github.com/user-attachments/assets/d0bdefb2-c8c0-4fe5-90e3-ecf8757db33d" />

### Step 1: 인스턴스 및 기간 선택
원하는 인스턴스 타입(`trn2.48xlarge`)과 기간, 시작 날짜를 선택합니다.
<img width="1092" height="1064" alt="cb2" src="https://github.com/user-attachments/assets/3a939d7c-24ac-47f7-9041-a5947f439f07" />

> **Tip:** Trainium2는 2025-12 기준 특정 리전(예: us-east-2)에서만 제공될 수 있으므로 리전을 잘 확인하세요.

### Step 2: 가능한 블록 확인 및 선택
가능한 시간대와 가격을 확인합니다. CB는 **특정 Availability Zone(AZ)**에 고정되어 제공됩니다.
> **중요:** 여기서 배정받은 AZ (예: `us-east-2b`)를 반드시 기억해야 합니다. 추후 이 AZ에 서브넷을 만들어야 인스턴스를 띄울 수 있습니다.>

> **중요:** Capacity Block 은 시작시간은 기본적으로 한국시간 20:30 이지만 조회시점에 바로 가용한 인스턴스가 있다면 즉시 시작하는 옵션을 선택하고 1일 단위가 아닌 추가 시간 및 금액이 표기된 CB 를 선택하여 바로 작업을 시작 할 수 있습니다.

> **중요:** Capacity Block 종료시간은 시작시간과 동일하게 20:30 이며 인스턴스는 20:00 부터 종료 됩니다. 작업 중인 내용은 20:00 전에 완료 해야 합니다. 

<img width="792" height="648" alt="cb3" src="https://github.com/user-attachments/assets/6f952c62-25a5-43bf-ae20-45ce5e30c5f7" />


<img width="1092" height="364" alt="cb4" src="https://github.com/user-attachments/assets/ecfe375e-bcf3-4a69-980e-adf1e9e9b33e" />
<img width="1099" height="1113" alt="cb5" src="https://github.com/user-attachments/assets/dff2ee39-e70e-4297-ae2c-2074fe27962d" />

### Step 3: 구매 확정 (Confirm)
가격과 시간을 최종 확인하고, 텍스트 입력창에 `confirm`을 입력하여 구매를 확정합니다.
<img width="593" height="401" alt="cb6" src="https://github.com/user-attachments/assets/4860c686-ff5d-4268-98d8-532d9f0c02ed" />


---

## 2. 예약 상태 확인 (Scheduled vs Active)

구매가 완료되면 상태가 변경됩니다.
<img width="1227" height="291" alt="cb7" src="https://github.com/user-attachments/assets/06f7c297-a389-4487-89ea-d2adfb7149d8" />
<img width="1224" height="276" alt="cb8" src="https://github.com/user-attachments/assets/cd5dc29e-318e-4cd6-a8de-08db56ef1c94" />


* **Scheduled (예정됨):** 구매는 성공했으나, 아직 시작 시간이 되지 않은 상태입니다.
<img width="1189" height="1347" alt="cb10" src="https://github.com/user-attachments/assets/dda30429-b842-4029-842d-8d9a4e88f7e3" />


* **Active (활성):** 예약 시간이 되어 인스턴스를 실행할 수 있는 상태입니다.
<img width="1220" height="1191" alt="cb9" src="https://github.com/user-attachments/assets/a517bee9-636d-4823-a9ba-c05aa98a3f3b" />

---

## 3. (중요) 네트워크 구성 주의사항

Capacity Block은 구매 시점에 **AZ(가용 영역)가 지정**됩니다. (예: `us-east-2b`)
따라서 인스턴스를 띄울 때 **반드시 해당 AZ에 있는 서브넷**을 사용해야 합니다.

> 스크린샷의 예시에서는 **`us-east-2b`** 가 할당되었으므로, VPC 생성 시 해당 구역에 서브넷을 생성해야 합니다.
