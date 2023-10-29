# CloudFormation

[AWS CloudFormation](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/Welcome.html)은 AWS 리소스를 모델링하고 설정하여 리소스 관리 시간을 줄이고 AWS에서 실행되는 애플리케이션에 
더 많은 시간을 사용하도록 해 주는 서비스입니다. AWS 리소스를 설명하는 템플릿을 생성하면 CloudFormation이 해당 리소스의 프로비저닝과 구성을 담당합니다.

<br>

## Checkout
```
git clone https://github.com/chiwooiac/cloud-formation.git
```

## CloudFormation 템플릿

템플릿이란 스택을 구성하는 AWS 리소스를 선언한 것입니다. 템플릿은 YAML 표준을 준수하는 형식의 텍스트 파일로 정의합니다.

필요한 최상위 객체는 Resources 객체뿐이며, 하나 이상의 리소스를 선언해야 합니다. Resources 객체만을 포함하는 가장 기본적인 수준의 템플릿으로 시작하겠습니다. 이 템플릿에는 단일 리소스 선언만 들어 있습니다.

```yaml
Resources:
```


## S3 버킷 리소스 샘플 

S3 버킷 리소스를 생성 하며 S3 버킷 객체의 이름은 HelloBucket 입니다. CloudFormation 스택을 생성하면 HelloBucket 이 생성 됩니다.

`Type` 속성은 AWS 클라우드 리소스를 정의 합니다.

```yaml
Resources:
  HelloBucket:
    Type: 'AWS::S3::Bucket'
```

<br>

`Properties` 속성은 리소스 'AWS::S3::Bucket' 가 가지는 고유의 속성을 정의 합니다. 

다음은 S3 버킷 프로비저닝을 위한 `이름`, `속성`, `태그 정보`를 정의하고 있습니다. 

```yaml
Resources:
  HelloBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-hello-s3 
      AccessControl: Private
      Tags:
        - Key: Name
          Value: my-hello-s3
        - Key: Environment
          Value: Production
```


<br>


### EC2 리소스 선언으로 보는 연관 리소스 구성

`!Ref` 키워드는 템플릿으로 선언된 다른 리소스를 참조할 수 있습니다. 현재 리소스가 구성되기 위해선 `!Ref`가 가리키는 리소스가 사전에 만들어진 이후 입니다.

아래 샘플은 EC2 인스턴스를 구성합니다. 연관되는 리소스로 SecurityGroup, KeyPair, AMI-ID 를 채울수 있습니다.



```yaml
Resources:
  Ec2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      SecurityGroups:
        - !Ref InstanceSecurityGroup
      KeyName: mykey
      ImageId: ''
  InstanceSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
```

<br>

### 입력 파라미터를 사용하여 사용자 맞춤형 프로비저닝 구성


`Parameters` 속성으로 사용자 입력 파라미터를 넘겨받을 수 있습니다.  

```yaml
Parameters:
  WordPressUser:
    Type: String
    Default: admin
    NoEcho: true
    Description: The WordPress database admin account user name
    MinLength: 1
    MaxLength: 16
    AllowedPattern: "[a-zA-Z][a-zA-Z0-9]*"
  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access into the WordPress web server
    Type: AWS::EC2::KeyPair::KeyName
  WebServerPort:
    Type: Number
    Default: 8888
    Description: TCP/IP port for the WordPress web server
    MinValue: 1
    MaxValue: 65535
  OSName:
    Type: String
    AllowedValues:
      - amazon
      - ubuntu
```

<br>


## 매핑을 사용하여 조건 값 지정

`Mappings` 속성은 Key-Value 페어로 매핑됩니다. 아래 RegionMap 매핑의 AMI는 레이블이며 AMI ID는 값입니다. 

```
Parameters:
  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instance
    Type: String
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-76f0061f
    us-west-1:
      AMI: ami-655a0a20
    eu-west-1:
      AMI: ami-7fd4e10b
    ap-southeast-1:
      AMI: ami-72621c20
    ap-northeast-1:
      AMI: ami-8e08a38f
Resources:
  Ec2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      KeyName: !Ref KeyName
      ImageId: !FindInMap 
        - RegionMap
        - !Ref 'AWS::Region'
        - AMI
      UserData: !Base64 '80'
```

- Ec2Instance 인스턴스를 프로비저닝 함에 있어서 `ImageId` 의 참조는 `RegionMap` 매핑에서 현재 세션의 AWS 리전 `!Ref 'AWS::Region'` 에 해당하는 AMI 값을 가져오라는 의미 입니다. 

```yaml
ImageId: !FindInMap
  - RegionMap
  - !Ref 'AWS::Region'
  - AMI
```

<br>

## [AWS 리소스 및 속성 유형 참조](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html) 

[AWS 리소스 및 속성 유형 참조](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)를 참조하여 원하는 리소스를 원하는 형태로 프로비저닝 할 수 있습니다.


<br>

## [내장 함수 참조](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)

CloudFormation 의 [내장 함수](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)를 사용하여 문자열, 컬렉션, 리소스의 속성, Loop 반복, Subset 등을 제어할 수 있습니다.


## Bastion EC2
- [Bastion EC2](./src/ec2/bastion/bastion-1.0.yaml) 관리 목적의 EC2 베스천을 생성하는 CloudFormation 템플릿 입니다.
- [nova-provisioner](./src/ec2/nova-provisioner/HELP.md) AWS 리소스를 Terraform 으로 프로비저닝 할 수 있는 EC2 컴퓨팅을 생성하는 CloudFormation 템플릿 입니다.
- [github-actions-role](./src/iam-role/github-actions-role/HELP.md) Github Actions이 빌드된 Artifact를 AWS 클라우드에 배포하기위한 AssumedRole을 생성하는 CloudFormation 템플릿 입니다.



