---
layout: post
title: (terraform) 모듈을 사용해서 코드 재작성
subtitle: 모듈을 사용하면 예전 방식으로 돌아가지 않을 겁니다.
banner:
  image: https://i.pinimg.com/736x/c9/cb/70/c9cb70d00a543a9e7b752602b00df888.jpg
  opacity: 0.618
  background: "#000"
  heading_style: "font-size: 4.25em; font-weight: bold; "
author: 오재문
categories: [DevOps]
tags: [terraform,IaC,module]
comments: true

---

## 모듈은 왜 필요 한가?

|Environment	|환경|	Description|
|--|--|--|
|Production	|운영|사용자에게 제공하는 환경|
|Staging|	검증|	운영서버와 비슷한 환경에서에 올리기 전 검증을 위한 단계|
|Development|개발|비용생각,형상은 같지만 규모는 작음|
|QA	|품질|	성능에 관한 검증|
|Local|	로컬|	개발자의 컴퓨터 및 작업환경|

비지니스 규모가 커짐에 따라 관리해야 할 환경이 많아지는 만큼 코드량도 많아지게 됩니다. 때문에 코드 관리를 위한 효율적인 방법을 생각해야 합니다. 단순한 코드 복사붙여넣기보다 코드 덩어리를 의미있는 묶음을 만들어 재사용가능하게 만드는 것이 효율적일 것입니다. 그래서 `모듈`을 통해 재사용한 코드를 작성해보겠습니다.

>A Terraform module is a set of Terraform configuration files in a single directory.

테라폼의 모듈은 폴더에 있는 모든 테라폼 구성 파일입니다. 이전의 모듈을 사용하지 않고 작성했던 코드도 모듈이라고 볼 수 있습니다. 그래서 모듈을 작성하는 것 자체는 전과 같은 방식이기 때문에 어려울 것이 없습니다. 다만 재사용이 가능하도록 몇가지를 수정해야 합니다.

```bash
.
├── live      <----모듈을 사용하는 repo
│   ├── global   <----모든 환경에서 사용되는 리소스를 관리(ex IAM)
│   ├── prod     <----사용자에게 제공하는 환경
│   └── stage    <----프로덕션 검증 환경
└── modules   <----모듈을 정의하는 repo
    ├── IAM
    └── vpc
        ├── main.tf
        ├── outputs.tf
        └── variables.tf

```
우선 공간을 나눌 필요가 있습니다. 크게 모듈을 사용하는 공간과 모듈을 정의하는 공간을 나눕니다. 코드를 저장하는 원격저장소 또한 효율적인 관리를 위해 따로 두어야합니다.

## 모듈 정의

```
└── modules
      └── vpc
          ├── main.tf
          ├── outputs.tf
          └── variables.tf
```

모듈은 프로그래밍 언어에서의 함수와 같습니다. 함수는 외부로부터 값을 받을 수 있는 파라미터, 원하는 값을 도출하기 위한 로직, 도출한 값을 외부로 내보낼 리턴값이 존재합니다. 마찬가지로 모듈을 만들기 위해서 이 세가지가 구성되어야 합니다. 

물론 반드시 그래야 되는 것은 아닙니다. 하지만 코드는 항상 다른 사용자를 염두해 두고 작성하는 것이 좋습니다. 때문에 코드에 대한 정보를 자세히할 필요가 있고 확장가능해야 합니다. 그것을 위해 세가지 구조로 구성하는 것이 다른 사용자와 나 자신에 대한 배려라고 생각합니다.

variables.tf는 모듈을 호출하는 쪽에서 값을 정하도록 파라미터를 만들수 있도록 합니다. 이 방식은 모듈의 재사용성을 높일 수 있습니다. 어떤 값을 변수로 정해야 하는지 한번에 정하는 것이 어렵다면 모듈을 어려 프로젝트에 적욯해 보면서 수정해 나가는 것이 좋은 방법이 될 것입니다.

우선 VPC이름과 VPC 및 서브넷들의 cidr 값을 변수로 만들어서 호출하는 쪽에서 정의하도록 변수로 만들어 보겠습니다.

```
#variables.tf

variable "name"{
  description ="set vpc name"
  type = string
  default = "vpc"
}
variable "vpc_cidr"{
  description ="vpc cidr block "
  type = string
  default = "10.0.0.0/16"
}

variable "private"{
  description ="private subnet"
  type = string
  default = "10.0.0.0/24"
}

variable "public"{
  description ="public subnet"
  type = string
  default = "10.0.1.0/24"
}
```
그런다음 main.tf를 통해 하드코딩된 이름 대신 `var.<변수명>`을 사용합니다.(예를 들어 var.vpc_cidr)이렇게 하면 모듈을 호출하는 곳에서 변수의 값을 지정할 수 있습니다.

---

main.tf는 원하는 동작을 하도록 테라폼 리소스들을 모아놓은 곳입니다. 모듈 밖에서 설정해야 되는 데이터들은 변수를 통해 주입받고 나머지 부분은 로컬변수를 통해 쉽게 읽을 수 있고 관리가 용이하도록 해주는 것을 권장합니다.

```terraform

locals{
  any_port = 0
  any_protocol = "-1"
  all_ips = ["0.0.0.0/0"]
}

resource "aws_security_group" "vpc_allow_all" {
  name = "vpc_allow_all"
  vpc_id = aws_vpc.vpc.id

  ingress {
    from_port = local.any_port
    to_port = local.any_port
    protocol = local.any_protocol
    cidr_blocks = local.all_ips
  }

  egress {
    from_port = local.any_port
    to_port = local.any_port
    protocol = local.any_protocol
    cidr_blocks = local.all_ips
  }

  tags {
    Name = "${var.test_vpc_tag_name} security_group"
  }
}
```

로컬 변수는 현재 실행 파일에서 사용되는 지역 변수 입니다.파일 외의 다른 모듈에는 영향을 주지않으며 모듈 외부에서 이 값을 재정의할 수 없습니다.  주로 특정 값들을 연산하여 하나의 변수로 만들어야 할 때 사용 됩니다. merge, concat, max 와 같은 다양한 함수를 사용하여 변수를 만들 수도 있습니다. 위의 경우는 여러곳에서 하드코딩되고 있는 값들을 로컬변수로 관리하는 예로 볼 수 있습니다.

---

프로그래밍 언어에서 함수를 사용할 경우 함수내에 계산된 값을 함수 바깥에서 필요로 하는 경우가 있습니다. 그럴때 리턴을 이용해서 함수 밖으로 값을 반환할수 있습니다. 모듈도 마찬가지입니다. outputs.tf를 통해 모듈 내에 정의된 값들을 반환할 수 있습니다. 

outputs 파일에 출력값을 정의 했을 경우 모듈을 호출할 쪽에서 다음과 같은 문구로 출력 변수에 접근할 수 있습니다.

```
module.<MODULE_NAME>.<OUTPUT_NAME>
```
---
## 모듈 호출

```bash
└── live         
    ├── global
    ├── prod
    └── stage
```

작성한 코드가 실행되는지 확인을 해 보겠습니다. 모듈은 다른 디렉토리에서 호출되어 쓸수 있습니다. 구문은 다음과 같습니다.

```tf
module "<NAME>" {
	source = "<SOURCE>"
	[CONFIG...]
}
``` 
```terraform
# stage/main.tf

module vpc {
  source = "../../modules/vpc"
  name  = stage_vpc
  vpc_cidr = "1.0.0.0/16"
  private = "1.0.0.0/24"
  public = "1.0.1.0/24"
}


# prod/main.tf

module prod_vpc {
  source = "../../modules/vpc"
  vpc_name  = "prod_vpc"
}
```

주의할 점은 모듈을 추가하거나 변경이 있을 때마다 init명령을 해야 올바르게 작동합니다. apply 명령을 한 후 올바르게 동작하는지 확인하고 삭제합니다. 

![image](https://user-images.githubusercontent.com/51963264/226698457-6ea28742-faa0-41b9-811d-a26312ed7ace.png)


---

## 버전 관리

문제점이 있습니다. 스테이징환경과 운영환경이 동일한 모듈을 가리킬 경우 해당 모듈을 변경하여 배포할 때 두 환경 모두에게 영향을 주게 됩니다. 의도한대로라면 스테이징환경에 검증을 거친 후 운영환경에 적용하는 것이 맞습니다.

지금까지의 모듈을 사용할 때마다 로컬 파일 경로를 이용했습니다. 사실 모듈은 파일 경로 외에도 HTTP URL을 이용하여 모듈을 불러올 수 있습니다. 버전을 만드는 쉬운 방법은 바로 깃을 이용하는 것입니다. 그래서 깃리포지토리를 통해 모듈폴더를 관리해보겠습니다.

그러기 위해서는 구조를 변경할 필요가 있습니다.기존에서  modules을 정의하는 구역과 정의한 모듈을 호출해서 사용하는 구역을 나눌 예정입니다. 각 구역은 레포지토리를 따로 두어 관리됩니다. 

마지막으로 변경된 코드는 깃허브 리포지토리에 저장되야 합니다. 깃허브 원격 계정과 연결한 후 깃 명령어를 통해 원격저장소로 푸시합니다. 또한 버전관리를 위해 깃허브 UI를 사용하여 릴리스를 생성하면 태그가 만들어 집니다.

깃허브의 모듈 경로로 바꾸면 의도한 대로 모듈을 환경별로 따로 만들 수 있습니다. 경로는 다음과 같습니다.

```
# stage/main.tf

module vpc {
  source = "git@github.com:<OWNER>/<REPO>.git//<PATH>?ref=v0.0.2"
  name  = stage_vpc
  vpc_cidr = "1.0.0.0/16"
  private = "1.0.0.0/24"
  public = "1.0.1.0/24"
}

# prod/main.tf

module prod_vpc {
  source = "git@github.com:<OWNER>/<REPO>.git//<PATH>?ref=v0.0.1"
  vpc_name  = "prod_vpc"
}
```
---

## 마치며

스테이징환경과 프로덕션환경에서 재사용가능하도록 모듈을 작성했습니다 하지만 사용자의 요구사항에 따라 프라이빗 서브넷이 필요하지 않을수 있고 좀 더 복잡한 구조의 인프라를 요구할지도 모릅니다.지금은 모듈로는 무리가 있지만 HCL에서 제공하는 조건문과 반복문을 사용하는 것으로 코드를 한번 작성하는 것으로 모든 환경에 적용이 가능한 조금 더 유연한 모듈을 만들 수 있습니다.

