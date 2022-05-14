# Install

```sh
# terraform
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform
```

```sh
# aws
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

# Config

```sh
terraform -install-autocomplete
```

# References
- [hashicorp/terraform](https://github.com/hashicorp/terraform) 
- [Terraform](https://www.terraform.io/) 
- [Terraform Docs](https://www.terraform.io/docs/) 
- [Terraform Use Cases](https://www.terraform.io/intro/use-cases.html) 
- [Terraform Registry hashicorp/aws](https://registry.terraform.io/providers/hashicorp/aws/latest/docs) 
- [Terraform - HashiCorp Learn](https://learn.hashicorp.com/tutorials/terraform/aws-variables?in=terraform/aws-get-started) 
- [Terraform 101](https://mooyoul.github.io/2016/12/19/Terraform-101/) 
- [Terraform Guide from GS](https://great-stone.github.io/categories/#terraform) 
- [Great-Stone/terraform-selfservice-app: Terraform selfservice template](https://github.com/Great-Stone/terraform-selfservice-app) 
- [테라폼(Terraform) 기초 튜토리얼: AWS로 시작하는 Infrastructure as Code | 44BITS](https://www.44bits.io/ko/post/terraform_introduction_infrastrucute_as_code) 
- [tfenv로 테라폼(Terraform) 버전 관리하기 | 44BITS](https://www.44bits.io/ko/post/managing-teraform-versions-with-tfenv) 
- [Terraform Course - Automate your AWS cloud infrastructure](https://www.youtube.com/watch?v=SLB_c_ayRMo)

