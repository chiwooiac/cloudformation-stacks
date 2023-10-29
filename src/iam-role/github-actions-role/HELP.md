# github-actions-role

Cloud Formation을 Github Actions 워크플로우가 빌드한 애플리케이션 Artifact를 AWS 클라우드 환경에 배포하기 위한 IAM 역할을 구성 합니다. 

<br>

## CloudFormation 템플릿 다운로드

```
curl -O https://raw.githubusercontent.com/chiwooiac/cloudformation-stacks/main/src/iam-role/github-actions-role/github-actions-cf-1.0.yaml
```

<br>

## CloudFormation 템플릿 검증
```
aws cloudformation validate-template --template-body file://./github-actions-cf-1.0.yaml
```

## CloudFormation 스택 생성

[AWS 관리 콘솔](https://console.aws.amazon.com/console/home) 에 로그인 및 `CloudFormation` 서비스에서 `Create Stack` 버튼을 클릭하여 Stack 을
생성 합니다.

![img.png](..%2F..%2F..%2Fimg%2Fimg.png)

- 다운로드 받은 `github-actions-cf-1.0.yaml` 파일을 `Upload a template file` 버튼을 통해 업로드 하고 Next 를 클릭 합니다.

![img_1.png](..%2F..%2F..%2Fimg%2Fimg_1.png)

- 주요 설정 정보 입력

![img_1.png](..%2F..%2F..%2Fimg%2Fimg_4.png)

<br>

| Lable Group       | Name                   | Example                   | Desc.                                                                                           |
|-------------------|------------------------|---------------------------|-------------------------------------------------------------------------------------------------|
| StackName         | StackName              | simplydemo-github-actions | CF 스택 이름입니다.                                                                                    |
| ResourceGroupName | Name                   | simpledemo                | 리소스 그룸 이름입니다. 프로젝트 코드를 입력할 수 있습니다.                                                              |
| OwnerShip         | Owner                  | admin@myterra.io          | CF 스택을 운영 및 관리하는 책임자입니다.                                                                        |
| -                 | Team                   | DevOps                    | CF 스택을 운영 및 관리하는 조직입니다.                                                                         |
| OIDC Settings     | GitHubOrg              | simplydemo                | Github Actions 가 실행되는 Github 조직 또는 계정 이름입니다.                                                    |
| -                 | AWSRegion              | ap-northeast-2            | Artifact를 배포 할 AWS 리전 코드를 선택합니다.                                                                |
| -                 | RepositoryName         | vertx-lotto-api           | AWS ECR 저장소 이름 입니다.                                                                             |
| -                 | ArtifactS3Bucket       | simplydemo-artifact-s3    | 빌드한 Artifact를 업로드 할 AWS S3 버킷 이름입니다. Spark Job jar 또는 lambda 코드를 압축한 zip 파일 등을 S3 버킷으로 업로드 합니다. |
| -                 | RoleMaxSessionDuration | 21600                     | Github Actions 가 액세스하는 STS 임시 토큰의 유효 시간입니다. (단위: 초)                                            |
 
<br>

- 모든 설정 정보를 리뷰하고 `Submit` 버튼을 클릭하여 CF 스택을 생성 합니다. 


<br>

## 자격 증명 공급자 및 페더레이션
[자격 증명 공급자 및 페더레이션](https://docs.aws.amazon.com/ko_kr/IAM/latest/UserGuide/id_roles_providers.html) AWS 외부에서 사용자 자격 증명을 이미 관리하고 있다면, IAM 페더레이션 자격 증명 공급자를 사용할 수 있습니다. 

Github, Google, Facebook 등에서 제공하는 OIDC 기반 외부 자격 증명 공급자(IdP)를 이용하여 사용자의 신원을 확인하고 로그인 할 수 있습니다. 

외부 자격 증명 공급자(IdP)를 사용하려면 IAM 자격 증명 공급자 엔터티를 생성하여 AWS 계정과 IdP 간에 페더레이션 신뢰 관계를 설정합니다.

- OIDC 공급자의 경우 다음 예시와 같이 하위 콘텍스트 키가 있는 OIDC 공급자의 정규화된 URL을 사용합니다.
```
server.example.com:sub
```

- `WebIdentity` 페더레이션은 보통 JWT의 일부로 토큰에 포함된 사용자를 고유하게 식별하고, 특정 리소스에 대한 권한 부여를 관리하기 위해 사용됩니다.


### github-actions OIDC Provider

`token.actions.githubusercontent.com`는 github actions 을 위한 OIDC 공급자 URL 입니다.

이 정보를 IAM 계정에 페더레이션 신뢰 관계를 통합하고 STS 임시 토큰을 발급하기위한 샘플은 아래와 같습니다.  

```
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::111122223333:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
                },
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": "repo:simplydemo/*:*"
                }
            }
        }
    ]
}
```

여기서 `sts:AssumeRoleWithWebIdentity`수행은 생성된 IAM 역할을 OIDC(인증된 웹 ID 공급자)로부터 받은 ID 토큰을 사용하여 AWS 리소스를 액세스 할 수 있도록 임시 토큰을 발급해 줍니다.  

여기서 aud와 sub는 JWT(JSON Web Token)의 페이로드에서 사용되는 속성입니다.  
- aud (audience: 대상)   
역할에 연결된 신원을 식별하는데 사용합니다. AWS 리소스를 액세스 하기위한 임시 토큰이 발급된 대상으로 AWS의 경우, 일반적으로 'sts.amazonaws.com'이 될 수 있습니다.

- sub (subject: 주체)  
토큰에 포함된 액세스 주체를 식별하는 데 사용됩니다. 일반적으로 IdP 공급자가 제공한 사용자/클라이언트 앱에 대한 고유한 식별자입니다. 

