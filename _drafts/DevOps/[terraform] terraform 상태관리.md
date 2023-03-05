# [terraform] terraform 상태관리

이때까지 우리는 terraform의 상태를 local의  **terraform.tfstate파일을 통해** 관리 해왔습니다. local환경 에서의 상태 관리는 팀 단위일 때 문제가 될 수있습니다. 

테라폼 상태관리에 있어서 여러 사용자가 동시에 접근하지 못하는 `잠금` 기능은  필수적 입니다. 때문에 다른 개발 언어와 같이 깃을 이용해 테라폼 상태 파일을 저장하는 것은 부적절 합니다.

대신 테라폼에는 `백엔드` 기능이 있습니다. 테라폼 백엔드는 테라폼이 상태를 로드하고 저장하는 방법을 결정합니다. 기본 백엔드는 로컬 백엔드로써 로컬 디스크에 상태 파일을 저장합니다. 원격 백엔드를 사용하면 상태 파일을 원격 공유 저장소에 저장할 수 있습니다.

아마존 S3,애저 스토리지, 구글 클라우드 스토리지등 다양한 원격 백엔드가 지원됩니다. 그 중 terraform cloud를 이용해 원격상태 관리를 해보겠습니다.

### terraform cloud

먼저 회원가입이 필요합니다. [이곳](https://www.notion.so/terraform-terraform-1bb5d479d2224271894a262818691df4)에서 회원 가입을 한 후 token을 만들어 줍니다. 그런 다음 local환경에 token을 저장해야 합니다. ~/.terraformrc을 열어 다음과 같이 작성합니다.

```bash
plugin_cache_dir = "$HOME/.terraform.d/plugin-cache"

credentials "app.terraform.io"{
 token ="{terraform 에서 발급받은 키를 넣어 주세요}"
}
```

그런 다음 다시 terraform 코드로 돌아와 아래와 같이 코드를 추가합니다.

```bash

terraform {
    backend "remote" {
    hostname      =  "app.terraform.io"
    organization  =   "dhwoans"

    workspaces{
      name = "cloud-backend"
    }
  }
...
}
```

그런 다음  init apply 를 차례로 입력하면 끝납니다.

```bash
Error: configuring Terraform AWS Provider: no valid credential sources for Terraform AWS Provider found.
│ 
│ Please see https://registry.terraform.io/providers/hashicorp/aws
│ for more information about providing credentials.
│ 
│ Error: failed to refresh cached credentials, no EC2 IMDS role found, operation error ec2imds: GetMetadata, request send failed, Get "http://169.254.169.254/latest/meta-data/iam/security-credentials/": dial tcp 169.254.169.254:80: i/o timeout
│ 
│ 
│   with provider["registry.terraform.io/hashicorp/aws"],
│   on provider.tf line 19, in provider "aws":
│   19: provider "aws" {
│ 
╵
Operation failed: failed running terraform plan (exit 1)
```

만약 정상적으로 실행되지 않고 위와 같은 오류가 있다면 아래 글을 참고하면 해결할 수 있습니다.

[Terraform error configuring AWS provider backend issue](https://stackoverflow.com/questions/71906029/terraform-error-configuring-aws-provider-backend-issue)