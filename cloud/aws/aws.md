# S3

- bucket Policy aws:referer를 사용하면 특정 도메인 Allow/Deny 제어 가능



# ELB

## ALB

- SG에서 1차 필터링 후 고급 라우팅 설정으로 HTTP Header값 기반 특정 도메인 Allow/Deny 제어 가능

## NLB

- NLB는 ALB와 달리 SG가 존재하지 않는다. 그러므로 NLB => EC2로의 SG Inbound Rule 등록시 NLB의 IP CIDR를 넣는다. 



# ECS

```sh
# install aws-cli

aws configure

# install ecs-cli

# aws console에서 ecs cluster 생성

ecs-cli configure --region ap-northeast-2 --cluster public-cluster

# Create ECS Task
ecs-cli compose --project-name test create --launch-type EC2

# aws console에서 target group, alb 생성 후

# aws console에서 ecs service 생성
# or ecs-cli compose --project-name keycloak service up
```



## Logging

AWS FireLens의 경우, 컨테이너 로그 통합 관리 기능을 의미합니다. 기존 FireLens (19년 말 출시)가 없었을 때에는 직접 기능을 코드로 구현하고 배포 스크립트를 작성해서 로그 라우터 기능을 구현 했어야 했는데요. 방금말씀 드린 버든을 줄이고, 간편하게 사용할 수 있도록  만들어진 기능이라고 보시면 됩니다. FireLens의 경우, 실습 때처럼 몇 가지 configuration 값(라우팅 대상, 필터)을 정의하면 필요한 위치로 컨테이너 로그를 전송하도록 작성할 수 있습니다. 질문 주셨던 내용 중 일부를 복기해 보자면

- AWS에서 제공하는 Fluent Bit 이미지 외에 직접 만든 Fluent Bit 혹은 Fluentd 이미지로 구성할 수 있나?
    - 네 가능합니다. 또한, 기존에 구축해 놓으신 EFK 스택(Amazon OS(구 ElasticSearch), Fluentd, Kibana)이 있다고 말씀 주셨는데 위에 말씀드린 대로 구성 값을 정의하시면 해당 스택으로도 로그를 라우팅 할 수 있습니다. 또한, EC2로 ECS 클러스터를 구축하는 경우, side car 방식(실습에서 구성했던 방식)도 사용 가능하지만 각 worker node에 에이전트처럼 심는 daemonset 형태로도 배포하실 수 있습니다(Fargate의 경우, side car 방식만 지원).
    - 참고 자료:
        - FireLens를 통해 로그를 라우팅할 때 작성하는 ECS task 정의 샘플 파일:https://docs.aws.amazon.com/AmazonECS/latest/developerguide/firelens-example-taskdefs.html#firelens-example-forward
        - ECS custom log routing: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_firelens.html
        - ECS FireLens 구성 파일 예시: https://github.com/aws-samples/amazon-ecs-firelens-examples#fluent-bit-examples
        - Fluent Bit vs Fluentd 성능표(블로그 하단): https://aws.amazon.com/blogs/opensource/centralized-container-logging-fluent-bit/
- 추가적으로 container insight의 경우,  애플리케이션 단에서 발생하는 로그 및 메트릭(CPU, memory disk 그리고 network) 값을 확인하실 수 있고 컨테이너 재시작, 메트릭 값에 의해 트리거되는 CloudWatch alarm 등의 기능도 함께 사용하실 수 있습니다. 아래의 workshop 자료를 참고하세요.
    - 참고 자료: https://catalog.us-east-1.prod.workshops.aws/v2/workshops/31676d37-bbe9-4992-9cd1-ceae13c5116c/ko-KR/containerinsights/ecs



# EKS

> [EKS Best Practices Guides](https://aws.github.io/aws-eks-best-practices/) 

- IP 주소 인벤토리를 확인하기 위해, CNI Metrics Helper와 같은 툴을 사용하면 Cloud Watch Alarms를 설정하여 서브넷의 IP 주소가 부족할 경우 알림을 받을 수 있다. Cluster Auto Scaling은 Karpenter(AWS에서만 사용가능, 추후 범용으로 확장 예정)를 사용하면 좀 더 용이하다.



# CDK

> [CDK Labs](https://github.com/cdklabs) 
>
> [AWS ECS DevOps using AWS CDK](https://github.com/aws-samples/aws-ecs-devops-using-aws-cdk) 



# References

- [AWS Console](https://ap-northeast-2.console.aws.amazon.com/console/) 
- [AWS Console dgdsingen](https://dgdsingen.signin.aws.amazon.com/console/) 
- [AWS Service Health Dashboard](https://status.aws.amazon.com/) 
- [AWS](https://aws.amazon.com/ko/) 
- [AWS FAQ](https://aws.amazon.com/ko/faqs/) 
- [AWS Docs](https://docs.aws.amazon.com/) 
- [AWS Blogs](https://aws.amazon.com/blogs/) 
- [AWS Blog ko](https://aws.amazon.com/ko/blogs/korea/) 
- [AWS Whitepapers](https://aws.amazon.com/ko/whitepapers/) 
- [AWS Whitepapers ko](https://aws.amazon.com/ko/blogs/korea/ko-whitepapers/) 
- [AWS Knowledge Center](https://aws.amazon.com/ko/premiumsupport/knowledge-center/) 
- [AWS Certification](https://aws.amazon.com/ko/certification/) 
- [AWS Training](https://www.aws.training/Account/Transcript/Current) 
- [AWS Skill Builder](https://explore.skillbuilder.aws/learn/signin) 
- [AWS Architecture](https://aws.amazon.com/ko/architecture/) 
- [AWS Well-Architected](https://aws.amazon.com/ko/architecture/well-architected/) 
- [Amazon Builders Library](https://aws.amazon.com/ko/builders-library/) 
- [AWS Web Hosting](https://aws.amazon.com/ko/websites/) 
- [AWS Economics](https://aws.amazon.com/ko/economics/) 
- [AWS Pricing](https://aws.amazon.com/ko/pricing/) 
- [AWS Support](https://aws.amazon.com/ko/premiumsupport/?nc2=h_ql_ce) 
- [AWS Solutions](https://aws.amazon.com/ko/solutions/) 
- [AWS Pricing Calculator](https://calculator.aws/#/) 
- [AWS Quick Starts](https://aws.amazon.com/ko/quickstart/) 
- [Amazon Science](https://www.amazon.science/) 
- [AWS Workshops](https://workshops.aws/) 
- [AWS Samples](https://github.com/aws-samples) 
- [AWS Network Firewall](https://aws.amazon.com/ko/blogs/korea/aws-network-firewall-new-managed-firewall-service-in-vpc/) 
- [AWS WAF](https://aws.amazon.com/ko/waf/) 
- [Amazon EC2 인스턴스 유형](https://aws.amazon.com/ko/ec2/instance-types/) 
- [Linux AMI 가상화 유형](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/virtualization_types.html) 
- [Linux의 RAID 구성](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/raid-config.html) 
- [Amazon EBS 볼륨 유형](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/EBSVolumeTypes.html#EBSVolumeTypes_gp2) 
- [EBS 비용 최적화](https://www.popit.kr/aws-ebselastic-block-storage%EC%9D%98-%EB%B9%84%EC%9A%A9-%EC%B5%9C%EC%A0%81%ED%99%94/)
- [API GW Private API](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/apigateway-private-apis.html) 
- [국내 파트너 - Security Information Center](http://security.aws-korea.com/local_partners/) 
- [IAM 모범 사례 - AWS Identity and Access Management](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/best-practices.html) 
- [Amazon Elasticsearch Service – Amazon Web Services(AWS)](https://aws.amazon.com/ko/elasticsearch-service/) 
- [AWS CloudFormation – 코드형 인프라 및 AWS 리소스 프로비저닝](https://aws.amazon.com/ko/cloudformation/) 
- [AWS Systems Manager Parameter Store - AWS Systems Manager](https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/systems-manager-paramstore.html) 
- [Shuffle Sharding : Massive & Magical Fault Isolation](https://aws.amazon.com/ko/blogs/architecture/shuffle-sharding-massive-and-magical-fault-isolation/) 
- [Amazon Aurora: Design Considerations for High Throughput Cloud-Native Relational Databases](https://www.allthingsdistributed.com/2017/05/amazon-aurora-design-considerations.html) 
- [AWS NetworkACL & SecurityGroup 개념](https://cleverdj.tistory.com/122) 
- [Cloud 환경에서의 방화벽 Network ACL 와 Security Group 에 대하여](https://jins-dev.tistory.com/entry/Cloud-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EB%B0%A9%ED%99%94%EB%B2%BD-Network-ACL-%EC%99%80-Security-Group-%EC%97%90-%EB%8C%80%ED%95%98%EC%97%AC) 
- [RDS with SSL/TLS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.SSL.html)

## Tutorial
- [ECS & CI/CD Workshop](https://ecs.containerhero.io/) 
- [Amazon EKS Workshop](https://www.eksworkshop.com/) 
- [AWS VPC 피어링: 서로 다른 VPC 간에 통신하기](https://www.44bits.io/ko/post/communication_between_vpcs__aws_vpc) 
- [AWS Builders 클라우드 비용, 어떻게 줄일 수 있을까?](https://www.slideshare.net/awskorea/aws-builders-134642233) 
- [Hystrix! API Gateway를 도와줘! - 우아한형제들 기술 블로그](http://woowabros.github.io/experience/2017/08/21/hystrix-tunning.html) 
- [AWS Beanstalk 성능 튜닝 - 1. DB Connection Pool](https://jojoldu.tistory.com/318?category=777282) 
- [AWS Beanstalk 성능 튜닝 - 2. 커널 파라미터 튜닝](https://jojoldu.tistory.com/319?category=777282) 
- [AWS Beanstalk 성능 튜닝 - 3. Nginx 튜닝](https://jojoldu.tistory.com/322?category=777282) 
- [AWS Beanstalk 성능 튜닝 - 4. 기타](https://jojoldu.tistory.com/323?category=777282) 
- [AWS Deploy - 1. Code Deploy 사용하기](https://jojoldu.tistory.com/281?category=777282) 
- [AWS Deploy - 2. AWS Code Build로 빌드하기](https://jojoldu.tistory.com/282?category=777282) 
- [AWS Deploy - 3. AWS Code Pipeline으로 배포하기](https://jojoldu.tistory.com/283?category=777282) 
- [AWS Code Deploy + Jenkins - 1. 기본 환경 구성](https://jojoldu.tistory.com/313) 
- [AWS Code Deploy + Jenkins - 2. Code Deploy 연동](https://jojoldu.tistory.com/314) 
- [AWS Code Deploy + Jenkins - 3. Jenkins 연동](https://jojoldu.tistory.com/315) 
- [SpringBoot로 웹 서비스 출시 - 6. TravisCI & AWS CodeDeploy로 배포 자동화](https://jojoldu.tistory.com/265) 
- [SpringBoot로 웹 서비스 출시 - 7. Nginx를 활용한 무중단 배포 구축하기](https://jojoldu.tistory.com/267?category=635883) 
- [SpringBoot로 웹 서비스 출시 - 8. 운영 환경 설정](https://jojoldu.tistory.com/269?category=635883) 
- [SpringBoot로 웹 서비스 출시 - 9. 도메인구매, HTTPS 연결, 타임존 수정](https://jojoldu.tistory.com/270?category=635883) 
- [SpringBoot로 웹 서비스 출시 - 10. 1인 개발 시작시 알았으면 좋았을 것들](https://jojoldu.tistory.com/272?category=635883) 
- [Nginx를 ELB Reverse Proxy로 사용할때 주의점](http://tech.kkung.net/blog/nginx-with-elb/) 
- [Nginx upstream에 AWS ELB DNS를 걸 때 주의사항](https://perfectacle.github.io/2019/04/28/nginx-proxy-aws-elb/) 

