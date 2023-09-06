# nova-provisioner

Cloud-Formation 을 이용하여 AWS Cloud 서비스를 프로비저닝할 수 있는 EC2 컴퓨팅을 프로비저닝 합니다.

<br>

## nova-provisioner-1.0.yaml 템플릿 다운로드

```
curl -O https://raw.githubusercontent.com/chiwooiac/cloudformation-stacks/main/src/ec2/nova-provisioner/nova-provisioner-1.0.yaml \
  /tmp/nova-provisioner-1.0.yaml
```

<br>

## CloudFormation 스택 생성

[AWS 관리 콘솔](https://console.aws.amazon.com/console/home) 에 로그인 및 `CloudFormation` 서비스에서 `Create Stack` 버튼을 클릭하여 Stack 을
생성 합니다.

![img.png](..%2F..%2F..%2Fimg%2Fimg.png)

- 다운로드 받은 `nova-provisioner-1.0.yaml` 파일을 `Upload a template file` 버튼을 통해 업로드 하고 Next 를 클릭 합니다.

![img_1.png](..%2F..%2F..%2Fimg%2Fimg_1.png)

- 주요 설정 정보 입력

![img_2.png](..%2F..%2F..%2Fimg%2Fimg_2.png)

| Name             | Example          | Desc.                                           |
|------------------|------------------|-------------------------------------------------|
| Stack name       | myterra          | CF 스택 이름입니다.                                    |
| Ownership        | admin@myterra.io | CF 스택을 운영 및 관리하는 책임자입니다.                        |
| Team             | DevOps           | CF 스택을 운영 및 관리하는 조직입니다.                         |
| Resource name    | provisioner      | 애플리케이션 리소스 이름 입니다.                              |
| Environment      | Production       | 스테이지 또는 런타임 환경 입니다.                             |
| OS type          | amazon           | Linux OS 번들입니다.  amazon 또는 ubuntu 를 선택할 수 있습니다. |
| Instance type    | t3.medium        | EC2 인스턴스 타입을 선택할 수 있습니다.                        |
| VPC              | vpc-ee44df7      | 현재 가용할 수 있는 VPC ID를 선택할 수 있습니다.                |
| Public subnet    | ssubnet-1e9b3d49 | 현재 가용할 수 있는 Public Subnet ID를 선택할 수 있습니다.      |

- 모든 설정 정보를 리뷰하고 `Submit` 버튼을 클릭하여 CF 스택을 생성 합니다. 

![img_3.png](..%2F..%2F..%2Fimg%2Fimg_3.png)