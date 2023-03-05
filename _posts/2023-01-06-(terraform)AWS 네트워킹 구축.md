---
layout: post
title: (terraform)AWS 네트워킹 구축
subtitle: 코드로 인프라를 관리해 봅시다
banner:
  image: https://i.pinimg.com/564x/e8/81/1a/e8811a9886ee7d0960530b92783fd18e.jpg
  opacity: 0.618
  background: "#000"
  heading_style: "font-size: 4.25em; font-weight: bold; "
author: 오재문
categories: [DevOps]
tags: [terraform,aws]
comments: true

---



[AWS 공식문서에서 제공하는 가이드](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/VPC_Scenario2.html)에 따라  웹 서버는 퍼블릭 서브넷에 두고 데이터베이스 서버는 프라이빗 서브넷에 두는 다중 계층 웹 사이트를 위한 인프라를 생성해 보도록하겠습니다. 

![Untitled](https://user-images.githubusercontent.com/51963264/222951484-c5be7b48-9c92-4791-8bc9-f3a3e22de522.png)

main.tf 파일을 만들어 리소스를 하나씩 써내려가도 무방하지만 리소스별로 파일을 나누어 생성해 보도록 하겠습니다.

```bash
.
├── account.tf
├── eip.tf
├── endpoint.tf
├── igw.tf
├── nat.tf
├── output.tf
├── route.tf
├── subnet.tf
├── terraform.tf
├── terraform.tfstate
├── terraform.tfvars
├── variables.tf
├── versions.tf
└── vpc.tf
```

### VPC 생성

```bash
# vpc.tf
resource "aws_vpc" "vpc" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "terraform_vpc"
  }
```

plan은 실제로 리소스를 작성하기 전 리소스가 어떻게 생성되는지를 확인하는 명령어입니다.

오류가 없다면 terraform apply 명령어를 통해 변경사항을 적용할수 있습니다.

![Untitled 1](https://user-images.githubusercontent.com/51963264/222951502-8054d5b0-e473-4aa5-b868-c38ffed3221a.png)

성공적으로 apply가 실행 됐다면 디렉토리에 ****추가적으로 terraform.tfstate 파일이 생성된 것을 확인할수 있는데요, ****해당 파일은 local에서 관리되는 상태 파일입니다.

현재 resources의 상태를 기록하고 있는 것을 확인하실 수 있습니다.

remote로 상태 관리하는 것은 다음에 다뤄보겠습니다.


### **서브넷 생성**

서브넷은 인터넷 접근이 불가능한 private과 접근가능한 public 두가지가 있습니다.

```bash
resource "aws_subnet" "privateSubnet" {
  vpc_id = "${aws_vpc.vpc.id}"
  cidr_block = "10.0.1.0/24"
  availability_zone = "ap-northeast-2c"
  tags = {
    Name = var.vpc_name
  }
}

resource "aws_subnet" "publicSubnet" {
  vpc_id = "${aws_vpc.vpc.id}"
  cidr_block = "10.0.2.0/24"
  availability_zone = "ap-northeast-2a"
  tags = {
    Name = var.vpc_name
  }
}
```

### 인터넷게이트웨이 연결

인터넷 게이트웨이는 VPC가 인터넷과 통신할 수 있도록 해줍니다. 루트 테이블에 연결됩니다. 앞서 말했듯 private subnet은 인터넷과 연결되어 있지 않기 때문에 public만 연결합니다.

```bash
resource "aws_internet_gateway" "IGW" {
  vpc_id = aws_vpc.vpc.id
  tags = {
    Name = var.vpc_name
  }
}
```

### **NAT Gateway(NGW)**

`ngw`는 `vpc` 내부의 `prviate subnet`의 인스턴스들과 인터넷/AWS 서비스에 연결하는 게이트 웨이입니다.

내부적으로 `public subnet`에 위치하고 있으며, `route table`을 통해서 연결됩니다. 

```bash
resource "aws_eip" "eip" {
  vpc = true
  
  lifecycle{
    create_before_destroy=true
  }
}
```

nat 게이트웨이를 만들기 위해 먼저 elastic IP를 생성합니다.

```bash
resource "aws_nat_gateway" "ngw" {
  allocation_id = aws_eip.eip.id
  subnet_id     = aws_subnet.publicSubnet.id
}
```

인터넷과 연결하기 위해서는 public subnet에 위치해야한다는 점을 유의해야합니다.

### Route Table

루트 테이블은 트랙픽이 어디로 가야 할지 알려주는 테이블 입니다.

VPC를 생성시 기본 루트 테이블이 함께 만들어집니다. 하지만 VPC를 보호하기 위해 기본 라우팅 테이블은 초기 설정을 그대로 두고 사용하지 않는 것을 권장합니다.

라우팅 테이블을 작성하고 이를 서브넷과 연결하겠습니다.

```bash
# route table
resource "aws_route_table" "publicRoute" {
  vpc_id = aws_vpc.vpc.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.IGW.id
  }
  tags = {
    Name = var.vpc_name
  }
}
resource "aws_route_table" "privateRoute" {
  vpc_id = aws_vpc.vpc.id
	route{
    gateway_id = aws_nat_gateway.NATgatewaay.id
  }
  tags = {
      Name = var.vpc_name
  }
}

##라우팅 테이블 연결
resource "aws_route_table_association" "publicRTAssociation" {
  subnet_id = aws_subnet.publicSubnet.id
  route_table_id = aws_route_table.publicRoute.id
}
resource "aws_route_table_association" "privateRTAssociation" {
  subnet_id = aws_subnet.privateSubnet.id
  route_table_id = aws_route_table.privateRoute.id
}
```

 

### Endpoint

nat gateway를 통해서 외부의 s3와 연결이 가능하지만 보안상의 위험에 노출될 수 있습니다. 때문에 endpoint를 이용해 내부 트랙픽이 외부로 드러나지 않게 할 수 있습니다.

```bash
resource "aws_vpc_endpoint" "s3" {
  vpc_id       = aws_vpc.vpc.id
  service_name = "com.amazonaws.ap-northeast-2.s3"
}

# Add vpc endpoint to route table of private subnet
resource "aws_vpc_endpoint_route_table_association" "s3_routetable" {
  vpc_endpoint_id = aws_vpc_endpoint.s3.id
  route_table_id = aws_route_table.privateRoute.id
} 
```

이 코드는 완벽하게 동작하지만 재사용하기에는 무리가 있습니다. 다음에는 모듈을 이용해서 재사용가능한 코드로 바꿔보겠습니다.