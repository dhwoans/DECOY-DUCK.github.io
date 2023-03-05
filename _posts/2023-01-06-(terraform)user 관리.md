---
layout: post
title: (terraform) user 관리
subtitle: 코드로 인프라를 관리해 봅시다
banner:
  image: https://geekylane.com/wp-content/uploads/2019/05/How-to-set-up-IAM-on-AWS-account-Complete-Step-by-Step-Guide.png
  opacity: 0.618
  background: "#000"
  heading_style: "font-size: 4.25em; font-weight: bold; "
author: 오재문
categories: [DevOps]
tags: [aws,terraform,IAM]
comments: true

---

## IAM

AWS Identity and Access Management(IAM)을 사용하여 리소스를 사용하도록 인증(로그인) 및 권한 부여(권한 있음)된 대상을 제어합니다.

IAM은 크게 **사용자(Users)**, **그룹(Groups)**, **역할(Roles)**, **정책(Policies)**으로 구성되어 있습니다. 잠시 살펴보겠습니다.

`IAM 정책`은 사용자, 그룹 및 역할에 대한 권한을 정의하는 문서입니다. 정책은 JSON 형식으로 작성되며 사용자, 그룹 및 역할에 정책을 연결하여 AWS 리소스에 대한 액세스를 제어할 수 있습니다.


![image](https://user-images.githubusercontent.com/51963264/222964099-b8ac8a7a-16c1-4830-a678-af37f8f51dfe.png)

`IAM 사용자`는 실제 사용자를 의미합니다.관리자, 개발자 또는 AWS 리소스에 액세스해야 하는 사람의 개별 AWS 계정입니다. 사용자를 이용해서 로그인이 가능합니다.

여기서 **IAM 사용자는 AWS 루트 계정(email)과 다르다**는 것을 유의하여야합니다. 루트 계정은 모든 권한을 갖고 있고, IAM 사용자는 단지 루트 계정에 의해 권한을 부여 받은 대상입니다. 루트 계정을 사용할 경우 위험에 노출될 수 있기 때문에  AWS에서는 루트 계정 대신 관리자 권한(Administrator Access)을 부여한 IAM 사용자를 따로 만들어서 사용하는 것을 권장하고 있습니다.

![image](https://user-images.githubusercontent.com/51963264/222964627-000f5b80-931f-4875-9fae-22f3c1a5432b.png)

`IAM 그룹`은 사용자 집합입니다. 그룹을 생성하고 그룹에 권한을 할당함으로써 관리자는 여러 사용자의 AWS 리소스에 대한 액세스를 한 번에 관리할 수 있습니다.

![image](https://user-images.githubusercontent.com/51963264/222966194-bf2ea5cf-91da-42a4-89dd-084b094ba91c.png)

`IAM 역할`은 특정 작업이나 기능에 대한 권한 집합을 정의하는 엔터티입니다. 역할을 EC2 인스턴스와 같은 AWS 서비스에 할당할 수 있습니다.
[공식문서](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/id.html)에 따르면 애플리케이션이 s3에 접근해야 하는 경우 역할을 부여하여 접근을 허용할수 있습니다. 

IAM에 관해서는 다음에 자세히 다뤄보겠습니다.여기서는 Terraform 으로 관리자 권한의 IAM user를 생성하는 것과 생성된 user를 IAM group 에 등록시키는 것을 해보겠습니다. 

## terraform을 이용한 IAM 관리

### IAM User 기본 생성

```bash
#iam.ft
resource "aws_iam_user" "iam" {
  name = var.iam_name
}
```

이것만으로도 AWS 콘솔창에 IAMUser가 정상적으로 생성된 것을 확인할 수 있습니다. 

하지만 IAM 유저로 로그인하기위해서는 IAM Username과 별도의 비밀번호가 필요합니다.  

IAM User로 로그인하기 위한 처리는 잠시 뒤에 다뤄보겠습니다.

### IAM group 기본 생성

```bash
#iam.ft
resource "aws_iam_group" "iam_group" {
  name = var.iam_group_name
}
```

마찬가지로  group 또한 name만 필수 값으로 가집니다. 하지만 유의미한 group을 생성하기 위해서는 그룹에 정책을 등록해야 합니다.

앞서 말했다시피 AWS에서는 루트 계정 대신 Administrator Access을 부여한 IAM 사용자를 따로 만들어서 사용하는 것을 권장하므로 damin 그룹을 만들어 보겠습니다.

**IAM group 에 정책 등록**

```
resource "aws_iam_group_policy" "admin" {
  name   = "iam_access_policy_for_admin"
  group  = aws_iam_group.iam_group.id
  policy = data.aws_iam_policy_document.devs.json
}
```

`policy`에 정책을 바로 적어도 좋지만 `data`로 따로 빼서 적어 보도록 하겠습니다.

```
data "aws_iam_policy_document" "admin" {
  # AdministratorAccess
	statement {
		actions   = ["*"]
		effect    = "Allow"
		resources = ["*"]
	}
}
```

### IAM group 에 IAM user 등록

```bash
resource "aws_iam_group_membership" "iam_group" {
  name = aws_iam_group.iam_group.name

  users = [
    aws_iam_user.iam_user.name
  ]

  group = aws_iam_group.iam_group.name
}
```

### 비밀번호 설정

IAM user를 통해 aws 콘솔에 로그인하기 위한 Password를 설정하는 방법에 대해 알아보도록 하겠습니다. 우선 비밀번호를 스크립트 상에 남기지 않아야 합니다. 하지만 terraform을 정상적으로 실행시키기 위해서는 비밀번호를 작성할 수 밖에 없습니다.이런 역설적인 상황을 해결하기 위해서는 암호화된 암호를 만들 수 밖에 없습니다.

우선 `aws_iam_user_login_profile` 리소스를 살펴 보겠습니다.



[pgp_key rgument](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_user_login_profile#argument-reference)를 통해 암호화 된 암호를 생성하여 암호노출을 최소화할 수 있습니다.

전체적인 흐름은 다음과 같습니다

1. Keybase를 설치합니다.
2. Keybase를 이용해 key를 생성합니다.
3.  terraform 코드를 작성합니다. 
4. terraform apply 실행합니다.
5. 출력된 비밀번호를 복호화 합니다.

 먼저 [https://keybase.io/](https://keybase.io/)에서 계정을 생성한후 키를 생성합니다.

```bash
brew install keybase
```

또는 mac경우 위의 명령어를 입력합니다. 그런 다음 키를 생성합니다.

```bash
curl https://keybase.io/{유저네임}/key.asc
```

성공적으로 키가 만들어졌다면 위의 명령어를 통해 확인할 수 있습니다.

사전 작업은 끝났고  테라폼으로 돌아와 리소스를 작성합니다.

```bash
resource "aws_iam_user_login_profile" "admin" {
  user    = aws_iam_user.iam_user.name
  pgp_key = "keybase:{유저네임}"
}

output "password" {
  value = aws_iam_user_login_profile.admin.encrypted_password
}
```

pgp_key에 `keybase:{유저네임}` 을 넣고 terraform apply를 실행했을 때 출력되는 인코딩된 암호를 확인합니다.

```bash
terraform output -raw password | base64 --decode | keybase pgp decrypt
```

그런 다음 출력된 암호를 복호화하는 명령어를 입력합니다. 그럼 user로 로그인하기 위한 준비가 다 되었습니다.

![Untitled](https://user-images.githubusercontent.com/51963264/222953247-d1c50a6e-0cd8-4916-88eb-ef96096a67f2.png)

로그인 되는지 확인합니다.