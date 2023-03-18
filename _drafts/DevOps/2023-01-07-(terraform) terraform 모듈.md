---
layout: post
title: (terraform) 모듈을 사용해서 재작성
subtitle: 모듈을 사용하면 예전 방식으로 돌아가지 않을 겁니다.
banner:
  image: https://i.pinimg.com/736x/c9/cb/70/c9cb70d00a543a9e7b752602b00df888.jpg
  opacity: 0.618
  background: "#000"
  heading_style: "font-size: 4.25em; font-weight: bold; "
author: 오재문
categories: [DevOps]
tags: [terraform,IaC]
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

비지니스 규모가 커짐에 따라 관리해야 할 환경도 많아지게 됩니다. 그만큼 코드량도 많아지게 됩니다. 때문에 코드 관리를 위한 효율적인 방법을 생각해야 합니다. 단순한 코드 복사보다 코드 덩어리를 의미있는 묶음을 만들어 재사용가능하게 만드는 것이 효율적일 것입니다. 그래서 모듈을 통해 재사용한 코드를 작성해보겠습니다.

## 모듈

>A Terraform module is a set of Terraform configuration files in a single directory.

테라폼의 모듈은 폴더에 있는 모든 테라폼 구성 파일입니다. 

```tf
module "<NAME>" {
	source = "<SOURCE>"
	[CONFIG...]
}
```
## 버전관리

성공적으로 모듈을 실행했지만 문제점이 있습니다. 스테이징환경과 운영환경이 동일한 모듈을 가르킬 경우 해당 모듈을 변경하여 배포할 때 두환경 모두에게 영향을 주게 됩니다. 의도한대로라면 스테이징환경에 검증을 거친 후 운영환경에 적용하는 것이 맞습니다.

지금까지의 모듈을 사용할 때마다 로컬 파일 경로를 이용했습니다. 사실 모듈은 파일 경로 외에도 HTTP URL을 이용하여 모듈을 불러올 수 있습니다. 버전을 만드는 쉬운 방법은 바로 깃을 이용하는 것입니다. 그래서 깃리포지토리를 통해 모듈폴더를 관리해보겠습니다.

## 마치며

