# VPC Setup for AWS Trainium (Capacity Blocks)

이 가이드는 AWS GPU/Trainium/Inferentia 인스턴스, 특히 **Capacity Block(CB)**을 사용하는 환경을 위한 VPC 구성 방법을 안내합니다.

Capacity Block은 **특정 Availability Zone(AZ)**에 고정되어 예약됩니다. 따라서 인스턴스를 실행하기 위해서는 **반드시 해당 AZ와 일치하는 서브넷**이 존재해야 합니다. 이 가이드에서는 **AWS CloudShell**을 사용하여 이 과정을 자동화하고 실수를 방지하는 방법을 다룹니다.

---

## 📋 Prerequisites (사전 준비)

작업을 시작하기 전에 다음 사항들이 준비되어야 합니다.

1.  **Capacity Block 구매 완료:** [EC2 Capacity Reservations] 메뉴에서 상태가 `Scheduled` 또는 `Active`인지 확인합니다.
2.  **AWS CloudShell 접근 권한:** AWS 콘솔에서 CloudShell을 실행할 수 있어야 합니다.

---

## 📍 Step 1: Capacity Block AZ 정보 확인

가장 먼저 내 Capacity Block이 어느 가용 영역(AZ)에 할당되었는지 확인해야 합니다.

1.  **AWS CloudShell**을 실행합니다.
2.  아래 명령어를 입력하여 내 계정의 Capacity Reservation 정보를 조회합니다.

```bash
aws ec2 describe-capacity-reservations \
    --query 'CapacityReservations[*].{ID:CapacityReservationId, AZ:AvailabilityZone, Type:CapacityReservationType, Status:State}' \
    --output table
```

<img width="1289" height="242" alt="Screenshot 2025-12-05 at 5 08 20 PM" src="https://github.com/user-attachments/assets/7edb8276-eb86-49b0-9b15-31400d2dc1cc" />

3. 조회된 CB 의 AZ 정보를 아래 TARGET_AZ 에 넣고 스크립트 실행

```bash
# ==========================================
# [중요] 여기에 Capacity Block이 할당된 AZ를 정확히 적어주세요!
TARGET_AZ="us-east-2b"  
# ==========================================

# 1. 나머지 변수 설정
VPC_NAME="Dev-Capacity-VPC"
SUBNET_NAME="Dev-Public-Subnet-CB"
IGW_NAME="Dev-IGW"
RT_NAME="Dev-Public-RT"
VPC_CIDR="10.0.0.0/16"
SUBNET_CIDR="10.0.1.0/24"

echo "=== VPC 생성 시작 (Target AZ: $TARGET_AZ) ==="

# 2. VPC 생성
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block $VPC_CIDR \
  --tag-specifications "ResourceType=vpc,Tags=[{Key=Name,Value=$VPC_NAME}]" \
  --query 'Vpc.VpcId' --output text)
echo "VPC Created: $VPC_ID"

# 3. 서브넷 생성 (AZ 지정 옵션 추가됨)
SUBNET_ID=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block $SUBNET_CIDR \
  --availability-zone $TARGET_AZ \
  --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=$SUBNET_NAME}]" \
  --query 'Subnet.SubnetId' --output text)
echo "Subnet Created in $TARGET_AZ: $SUBNET_ID"

# 4. 인터넷 게이트웨이(IGW) 생성
IGW_ID=$(aws ec2 create-internet-gateway \
  --tag-specifications "ResourceType=internet-gateway,Tags=[{Key=Name,Value=$IGW_NAME}]" \
  --query 'InternetGateway.InternetGatewayId' --output text)
echo "IGW Created: $IGW_ID"

# 5. IGW를 VPC에 연결
aws ec2 attach-internet-gateway \
  --vpc-id $VPC_ID \
  --internet-gateway-id $IGW_ID
echo "IGW Attached to VPC"

# 6. 라우팅 테이블 생성
RT_ID=$(aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --tag-specifications "ResourceType=route-table,Tags=[{Key=Name,Value=$RT_NAME}]" \
  --query 'RouteTable.RouteTableId' --output text)
echo "Route Table Created: $RT_ID"

# 7. 라우팅 경로 추가 (0.0.0.0/0 -> IGW)
aws ec2 create-route \
  --route-table-id $RT_ID \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id $IGW_ID > /dev/null
echo "Public Route Added"

# 8. 서브넷과 라우팅 테이블 연결
aws ec2 associate-route-table \
  --subnet-id $SUBNET_ID \
  --route-table-id $RT_ID > /dev/null
echo "Route Table Associated with Subnet"

# 9. 서브넷의 '퍼블릭 IP 자동 할당' 기능 켜기
aws ec2 modify-subnet-attribute \
  --subnet-id $SUBNET_ID \
  --map-public-ip-on-launch
echo "Auto-assign Public IP Enabled"

echo "=== 모든 작업 완료 ==="
echo "VPC ID: $VPC_ID"
echo "Subnet ID ($TARGET_AZ): $SUBNET_ID"

```

생성이 완료 되면 다음과 같은 메시지를 확인 할 수 있습니다. 

<img width="528" height="143" alt="Screenshot 2025-12-05 at 5 10 55 PM" src="https://github.com/user-attachments/assets/58d97626-1d82-4ff2-a290-e2b59a679bc4" />





