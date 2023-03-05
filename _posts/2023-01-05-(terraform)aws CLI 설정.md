---
layout: post
title: (terraform) aws cli 
subtitle: 코드로 인프라를 관리해 봅시다
banner:
  image: https://i.pinimg.com/564x/1e/a4/6c/1ea46cb639c6c2594f29c27bde15de42.jpg
  opacity: 0.618
  background: "#000"
  heading_style: "font-size: 4.25em; font-weight: bold; "
author: 오재문
categories: [DevOps]
tags: [aws]
comments: true

---

AWS CLI 를 이용하면 다양한 AWS 서비스를 스크립트로 작성 해서 여러 작업을 한번에 동작 시킬 수 있으며, 각종 관리 자동화를 수행하는데 유리합니다. AWS의 모든 서비스 기능은 API를 호출함으로서 가능하며, asw CLI 역시 간단한 명령줄을 통해 이러한 광범위한 API 기능을 수행할 수 있습니다.

## aws CLI 설치


### Window

[AWS 공식 DOC](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/install-cliv2-windows.html)로 접속하여 AWS CLI 버전 2 를 다운받습니다.

정상적으로 설치가 되었는지 아래 명령을 통해 확인합니다. 

```bash
aws --version

aws-cli/2.9.16 Python/3.9.11 Windows/10 exe/AMD64 prompt/off
```
### 💡정상적으로 작동하지 않는 경우 


![Untitled](https://user-images.githubusercontent.com/51963264/222947970-3c5526cb-29e5-4e20-8a22-94fdc18f5399.png)

AWS CLI 버전 1은 Python용 SDK를 사용하여 빌드되므로 호환되는 버전의 Python을 설치해야 합니다. 버전을 확인하여 2로 다시 설치를 시도하거나 아래 명령을 실행시킵니다.

```bash
pip3 install awscli
```

---

### MacOS

Homebrew를 사용하는 경우 다음 명령어로 설치할 수 있습니다.

```
$ brew install awscli
```

설치 후에는 `which`로 설치 경로를 확인하고, 실행이 되는 지 확인해봅니다.

```bash
which aws
/usr/local/bin/aws

aws --version
aws-cli/2.9.16 Python/3.11.1 Darwin/21.6.0 source/x86_64 prompt/off
```

실행 파일의 경로가 제대로 잡혀있지 않다면 홈브류의 `link` 명령어를 사용해 재설정합니다.

```
brew link awscli
```

다음 명령어로 최신 버전으로 업그레이드 할 수 있습니다.

```
brew upgrade awscli
```

## 자격증명 설정

AWS CLI에서 AWS의 다양한 리소스를 활용하기 위해서는 자격 증명이 필요합니다. 자격증명은 AWS 계정의 엑세스키를 통해 이루어 집니다. 우선 엑세스 키를 발급받아 보겠습니다.

### key 생성

자격증명 설정을 하기 위해 aws 계정의 접근키를 생성해야 합니다.

![Untitled 2](https://user-images.githubusercontent.com/51963264/222948444-e877eae2-d928-4007-8dee-6cd694198f3b.png)

계정 네임을 클릭한후 `Security credentials` 에 들어갑니다.

![Untitled 3](https://user-images.githubusercontent.com/51963264/222948470-c96a3604-ac57-4e26-8f0b-edebd511506e.png)


접근키 생성 카테고리를 클릭한 후 새로운 키 생성을 생성합니다.

AWSAccessKeyId : 자격증명 주체를 가리키는 ID 값
AWSSecretKey : 자격증명 주체가 본인임을 인증하는 Key 값

유의하실 점은 접근키는 누구에게나 공개가능하지만 Secret Access Key키는 아무에게 공개해서는 안됩니다.

AWS CLI에서 자격 증명 우선순위는 다음과 같습니다. 이 중 두가지를 실습해 보겠습니다.

- CLI 명령어 옵션
- 환경 변수
- CLI 자격 증명 파일  (~/.aws/credentials)
- CLI 설정 파일  (~/.aws/config)


### configure 명령어

로컬에 액세스 키를 등록하는 방법은 `configure` 명령어를 사용하는 방법입니다. `aws configure`를 실행하면 4개의 값을 즉석에서 지정할 수 있습니다.

```
$ aws configure
AWS Access Key ID [None]: AKIAJEXHUYCTEHM2D3S2A
AWS Secret Access Key [None]: 3BqwEFsOBd3vx11+TOHhI9LVi2
Default region name [None]:ap-northeast-2
Default output format [None]:
```

각 필드의 의미는 다음과 같습니다.

- **AWS 액세스 키 ID AWS Access Key ID**
    
    액세스 키의 ID 값을 지정합니다.
    
- **AWS 시크릿 액세스 키AWS Secret Access Key**
    
    액세스 키의 ID 값애 대응하는 시크릿 키를 지정합니다. 시크릿 키는 오직 발급하는 시점에만 확인할 수 있습니다.
    
- **기본 리전Default region name**
    
    `aws` 명령어를 사용할 때 API를 호출할 기본 리전을 지정합니다. ap-northeast-2는 서울 리전입니다. 이 값은 명령어를 실행할 때 `--region <REGION>` 옵션으로 덮어쓸 수 있습니다.
    
- **기본 출력 포맷Default output format**
    
    API 호출한 결과를 출력할 포맷을 지정합니다. text, json, table,yaml등 다양한 포맷을 사용할수 있습니다. 이 값은 명령어를 실행할 때 `--output <FORMAT>` 옵션으로 덮어쓸 수 있습니다.
    

![Untitled 4](https://user-images.githubusercontent.com/51963264/222948522-b9ca0730-63dd-46b5-9bbf-23d4e91e7566.png)


### CLI 설정 파일

또다른 방법으로 `/.aws/config` 에서 문서 편집을 통해 설정값을 등록할수 있다.

win

```bash
notepad .\.aws\config
```

mac

```bash
vim ./.aws/config
```

다음 자격증명 정보 확인명령어를 통해 확인 해볼수 있다.

```bash
aws sts get-caller-identity
```

![Untitled 5](https://user-images.githubusercontent.com/51963264/222948535-500b4804-53d4-4942-a436-62c214d01d0f.png)
