# High-Performance Storage Setup (Local NVMe RAID 0)

## 📖 Background: Why do we need this?

대규모 언어 모델(LLM) 학습 및 추론 환경에서는 **데이터 입출력(I/O) 속도**가 전체 성능의 병목이 될 수 있습니다.
일반적인 AWS 스토리지인 **EBS(Elastic Block Store)**는 네트워크 기반이므로, 수백 GB에 달하는 모델 가중치(Weights)를 로딩하거나 체크포인트를 저장할 때 속도 한계가 있습니다.

이에 AWS Accelerated Computing 인스턴스(Trn1, Trn2, P4, P5 등)는 물리 서버에 직접 연결된 초고속 **Instance Store (NVMe SSD)**를 제공합니다.

### 🎯 Objective
* **Speed:** 흩어져 있는 여러 개의 물리적 NVMe SSD를 하나로 묶어(RAID 0), **수십 GB/s의 압도적인 대역폭**을 확보합니다.
* **Capacity:** `trn2.48xlarge` 기준 4개의 SSD를 합쳐 약 **7~8TB**의 넉넉한 단일 작업 공간(`/data`)을 만듭니다.

> **ℹ️ Note:** 이 가이드는 **Trn2.48xlarge (4x NVMe)**를 기준으로 작성되었으나, Trn1, Inf2, P4d 등 Local NVMe를 제공하는 모든 고성능 인스턴스에 동일한 개념으로 적용됩니다.

---

Trainium2 (`trn2.48xlarge`) 인스턴스는 매우 빠른 속도의 **Local NVMe SSD (Instance Store)**를 제공합니다.
대규모 LLM 모델의 가중치(Weights) 로딩 속도를 극대화하기 위해, 여러 개의 NVMe 디스크를 **RAID 0**로 묶어 `/data` 디렉토리에 마운트하여 사용합니다.

> **⚠️ 주의 (Warning):**
> Instance Store는 **Ephemeral(임시) 스토리지**입니다. 인스턴스를 **Stop(중지)**하거나 **Terminate(종료)**하면 데이터가 **모두 삭제**됩니다. (Reboot 시에는 유지됨)
> 중요한 데이터(모델 체크포인트 등)는 반드시 S3나 EFS로 백업해야 합니다.

---

## 🛠️ Setup Script (Recommended)

아래 스크립트를 터미널에서 순차적으로 실행하여 구성 합니다. 

[마운트 되지 않은 초기 상태]

<img width="592" height="195" alt="Screenshot 2025-12-06 at 10 33 54 AM" src="https://github.com/user-attachments/assets/5a4bb1a3-276c-492d-b7b6-819fcc242b17" />



# 1. RAID 0 생성 (nvme 1~4번을 하나로 묶음)

mdadm 도구를 사용하여 4개의 디스크를 하나의 논리적 장치(/dev/md0)로 묶습니다.

```bash
sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=4 /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1 /dev/nvme4n1
```

[실행결과]
<img width="1269" height="87" alt="Screenshot 2025-12-06 at 10 36 53 AM" src="https://github.com/user-attachments/assets/16cb7011-10db-4cad-bb4c-dd2a6ecf02ed" />

이 때 해당 장치는 인스턴스에 연결된 nvme 를 아래 명령어로 확인 한 뒤에 인스턴스 상황에 맞게 기재 
```bash
lsblk
```
<img width="736" height="336" alt="Screenshot 2026-01-08 at 11 51 46 AM" src="https://github.com/user-attachments/assets/bb5d0be8-a2b5-4bc2-8126-6121f087cc1f" />

* 위 이미지의 경우 대상 장치는 nvme 는 nvme1n1, nvme4n1, nvme0n1, nvme3n1

# 2. 파일 시스템 포맷
```bash
sudo mkfs.ext4 /dev/md0
```

[실행결과]

<img width="759" height="294" alt="Screenshot 2025-12-06 at 10 37 30 AM" src="https://github.com/user-attachments/assets/33be543a-37d3-4a09-b7c6-fdffe80fb7a9" />

# 3. 마운트
```bash
sudo mkdir -p /data
sudo mount /dev/md0 /data
sudo chmod 777 /data
```

# 4. 마운트 확인 (약 7TB가 보여야 함)
```bash
df -h /data
```
<img width="595" height="210" alt="Screenshot 2025-12-06 at 10 38 08 AM" src="https://github.com/user-attachments/assets/be731bc3-9c54-4166-97b9-8507f643d837" />

---
