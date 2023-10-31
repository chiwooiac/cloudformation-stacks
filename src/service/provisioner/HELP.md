# provisioner stack

AWS 리소스를 프로비저닝 하기 위한 VPC 환경 및 EC2 인스턴스를 배포 합니다.

EC2 인스턴스는 프로비저닝 도구인 Terraform 이 설치되어 있습니다.

<br>

## CloudFormation 템플릿 다운로드

```
curl -O https://raw.githubusercontent.com/chiwooiac/cloudformation-stacks/main/src/service/provisioner-service-1.0.yaml
```

<br>

## CloudFormation 템플릿 검증
```
aws cloudformation validate-template --template-body file://./provisioner-service-1.0.yaml
```

## CloudFormation 스택 생성

[AWS 관리 콘솔](https://console.aws.amazon.com/console/home) 에 로그인 및 `CloudFormation` 서비스에서 `Create Stack` 버튼을 클릭하여 Stack 을
생성 합니다.

![img.png](..%2F..%2F..%2Fimg%2Fimg.png)

- `Create stack` 단계 에서 다운로드 받은 `provisioner-service-1.0.yaml` 파일을 `Upload a template file` 버튼을 통해 업로드 하고 Next 를 클릭 합니다.

![img_5.png](..%2F..%2F..%2Fimg%2Fimg_5.png)

- `Specify stack details` 단계 에서 주요 설정 정보를 입력합니다.

![img_6.png](..%2F..%2F..%2Fimg%2Fimg_6.png)

<br>

주요 입력 항목은 아래와 같습니다.

| Label Group            | Name         | Example             | Desc.                                                                     |
|------------------------|--------------|---------------------|---------------------------------------------------------------------------|
| StackName              | StackName    | simply              | Resource Name for CloudFormation Stack                                    |
| OwnerShip              | Owner        | admin@email.address | Enter the email of the Owner who will operate the CloudFormation stack.   |
| -                      | Team         | DevOps              | Team name that will operate the CloudFormation stack.                     |
| Resource Configuration | Name         | provision           | Resource Name for CloudFormation Stack                                    |
| -                      | Environment  | Development         | Choose the Runtime Environment  such as Production, Development, Testbed. |
| -                      | CidrBlock    | 10.33.100.0/24      | Input your VPC CIDR Block.                                                |
| Instance Settings      | InstanceType | t3.small            | Choose the instance type. (eg: t3.micro, t3.small, t3.medium, t3.large)   |
| -                      | OsType       | amazon              | Choose the OS type.                                                       |


<br>


- `Configure stack options` 단계 에서 태그, 추가적인 IAM 권한, 프로비저닝 실패에 대한 Rollback 정책을 설정할 수 있습니다. 그대로 두고 다음 단계로 이동합니다.

![img_7.png](..%2F..%2F..%2Fimg%2Fimg_7.png)

- `Review simply` 구성 내용을 간단하게 리뷰하고 하단의 `I acknowledge that AWS CloudFormation might create IAM resources with custom names.` 를 체크하고 `Submit` 버튼을 클릭하고 스택을 생성 합니다.
 
![img_8.png](..%2F..%2F..%2Fimg%2Fimg_8.png)

## CloudFormation 스택 트러블 슈팅

CloudFormation 템플릿을 실행 하던 중 얼마든지 프로비저닝 오류가 발생할 수있습니다.    

![img_9.png](..%2F..%2F..%2Fimg%2Fimg_9.png)

`Events` 탭에서 수행 이력을 살펴 보면 `ServiceLinkedRoleForAutoScaling` 리소스 생성 중 해당 리소스가 이미 존재하기에 발생된 오류 원인을 식별할 수 있습니다.

![img_10.png](..%2F..%2F..%2Fimg%2Fimg_10.png)

이 경우에는 `Stack aActions` 탭에서 `Import resources into stack` 버튼을 클릭하여 아래와 같이 템플릿에 정의한 이름 항목에 실제 리소스 이름 또는 ARN 을 기입하면 됩니다.

![img_11.png](..%2F..%2F..%2Fimg%2Fimg_11.png)