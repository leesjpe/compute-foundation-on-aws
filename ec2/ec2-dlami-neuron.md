# Launching AWS Neuron XC(Accelerate Compute) Instance with Capacity Block

이 가이드는 예약된 **Capacity Block(CB)**을 사용하여 **XC 인스턴스**(본 예는Trainium2 (`trn2.48xlarge`)) 를 생성하는 절차를 설명합니다.

> **사전 조건:**
> * [Capacity Block](../capacity-reservations/CapacityBlock.md/)이 `Active` 상태여야 합니다. 
> * [VPC 및 Subnet 구성](../networking/vpc.md)이 완료되어야 합니다. (CB AZ와 일치하는 서브넷 필수)

---

## 🚀 Step 1: AMI 선택 (Deep Learning AMI Neuron)

Trainium 인스턴스를 사용하기 위해서는 Neuron SDK가 포함된 전용 이미지를 사용해야 합니다.

1.  EC2 콘솔에서 **Launch Instances**를 클릭합니다.
2.  **Name:** `trn2-dlami` (원하는 이름)
3.  **Application and OS Images (AMI):**
    * 검색창에 `neuron` 입력
    * **Quick Start AMIs** 탭 또는 **Community AMIs**에서 아래 이미지를 선택합니다.
    * **추천 이미지:** `Deep Learning AMI Neuron (Ubuntu 22.04)`

<img width="968" height="788" alt="Screenshot 2025-12-05 at 5 15 42 PM" src="https://github.com/user-attachments/assets/bc9ee24b-ad63-42b6-a6d0-90e4e3ce7bce" />
<img width="1451" height="825" alt="Screenshot 2025-12-05 at 5 16 04 PM" src="https://github.com/user-attachments/assets/6b11fff4-5175-4c48-be7b-3f9d65454d84" />
<img width="927" height="673" alt="Screenshot 2025-12-05 at 5 16 31 PM" src="https://github.com/user-attachments/assets/8c4a0768-5025-41fe-a86a-41f236f94e9c" />


---

## 💻 Step 2: 인스턴스 유형 선택

Capacity Block을 통해 예약한 인스턴스 타입을 선택합니다.

1.  **Instance type:** `trn2.48xlarge` 선택
2.  **Advanced details** 섹션을 펼치거나, 인스턴스 유형 선택 하단의 **Capacity Reservation** 옵션을 확인합니다.
3.  **Capacity Reservation:** `Target by ID` 선택 (또는 그룹 선택)
4.  **Capacity Reservation ID:** 구매한 CB의 ID (예: `cr-0a9f30bc...`)를 선택
5.  **Key pair:** 기존 키페어 선택 또는 `Create new key pair` (유형: `ED25519` 권장)

<img width="951" height="641" alt="Screenshot 2025-12-05 at 5 17 01 PM" src="https://github.com/user-attachments/assets/dfb181ea-7602-4bd0-ac3e-943edcb7aa35" />
<img width="964" height="219" alt="Screenshot 2025-12-05 at 5 17 46 PM" src="https://github.com/user-attachments/assets/9cdf4716-3278-4917-b336-e305294b8d37" />
<img width="599" height="566" alt="Screenshot 2025-12-05 at 5 19 04 PM" src="https://github.com/user-attachments/assets/34c93512-6bce-493f-bec0-ad659c53d3c7" />

---

## 🌐 Step 3: 네트워크 설정 (AZ 일치 필수)

앞서 생성한 **CB 전용 VPC 및 서브넷**을 선택합니다.

1.  **Network settings** > **Edit** 클릭
2.  **VPC:** 앞서 생성한 `XC-Capacity-VPC` 선택
3.  **Subnet:** `XC-Public-Subnet-CB` (반드시 **CB와 동일한 AZ**여야 함)
4.  **Auto-assign public IP:** `Enable` (접속을 위해 필요)
5.  **Security group:** `Create security group`
    * **Allow SSH traffic from:** `My IP` (보안 권장)
  
<img width="941" height="625" alt="Screenshot 2025-12-05 at 5 19 33 PM" src="https://github.com/user-attachments/assets/584ab6e5-3f2f-4991-a191-6a44b3f03e56" />
<img width="954" height="1301" alt="Screenshot 2025-12-05 at 5 22 05 PM" src="https://github.com/user-attachments/assets/f5526606-2075-4ebe-97ab-6653df66ad89" />


---

## 💾 Step 5: 스토리지 구성 (EBS)

모델 가중치(Weight)와 데이터셋을 저장하기 위한 넉넉한 공간을 확보합니다.

1.  **Configure storage** 섹션 확인
2.  **Root volume (Volume 1):** 기본값을 `1024 GiB` (1TB) 이상으로 변경
    * *Qwen 3-32B 같은 대형 모델을 다루려면 최소 500GB 이상 권장*

<img width="953" height="518" alt="Screenshot 2025-12-05 at 5 23 13 PM" src="https://github.com/user-attachments/assets/c4750661-c128-4b46-b1aa-171af98f4604" />
<img width="947" height="486" alt="Screenshot 2025-12-05 at 5 44 56 PM" src="https://github.com/user-attachments/assets/78338c26-1307-438d-b4b7-4a76b633ce5f" />

<img width="1465" height="230" alt="Screenshot 2025-12-05 at 5 45 36 PM" src="https://github.com/user-attachments/assets/3fcf7405-c9ae-4a7b-8f32-c787c39d36a0" />
