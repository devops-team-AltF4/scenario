# scenario

# devops-01-Final-TeamF
### 프로젝트 기간 : 2022.05/17 ~ 2022.06.07
## 시나리오

개발 팀에 2명의 개발자로 시작하여, 2년 정도 비지니스를 운영하며 현재는 4명의 개발팀을 운영하고 있는 스타트업 A가 있습니다.

- 백엔드 (경력 7년, 팀장)
- 프론트엔드 (경력 3년)
- iOS (경력 5년)
- Android (경력 2년)
백엔드 개발자인 개발 팀장이 간단한 AWS 설정 등을 할 수 있어서 EC2 기반으로 서비스를 운영하다가, 25억 규모의 투자를 받게 되면서 빠르게 개발팀 인원을 확충하여 개발 속도를 높이려고 합니다. 이에 아래와 12명으로 개발팀 인원이 늘어나게 되었습니다.

- 새로운 CTO 1명
- 백엔드 2명
- 프론트엔드 4명
- iOS 2명
- Android 2명
- Devops 1명
빠른 기능 개발을 위하여 프론트엔드 등 iOS, Android 등 클라이언트 개발자가 대규모 확충되었지만, 비용 상의 이유로 백엔드와 DevOps 인원은 최소한의 인원을 꾸리도록 예산을 받을 수 있었습니다. 이때 새로운 CTO는 배포 과정 자동화가 이뤄져야 더 빠르게 기능 개발이 이뤄질 것이라고, 기존에 배포를 담당하며 개발 팀장 역할을 수행하던 백엔드 개발자가 데브옵스 역할을 겸임하면서 새로운 신입 DevOps 인원을 채용하여 관련 역할을 맡기도록 결정하였습니다.

기존의 DevOps 파이프라인을 개선하기 보다는, 새롭게 전반적으로 배포 프로세스를 자동화하는 업무가 필요할 것으로 예상됩니다.
   - 현재는 로컬에서 수동으로 docker build & push 를 하고 있는데, 이것을 자동화해야 합니다.
   - 현재 ssh를 통해 접속해서 수동으로 docker-compose up/down 을 통해 배포하는 방식을 사용하는데, 이를 자동화해야 합니다. (dev 환경)
   - production 환경은 container orchestration 을 통해 구성해야 합니다. 이때 production 환경과 동일한 staging 환경도 추가로 구성해야 합니다.
   - staging 및 production 환경에 대한 배포 파이프라인을 구성해야 합니다.
## 요구사항 (Dev)
Dev환경에서 EC2를 이용하여 ssh 접속 후, docker-compose up 명령어를 통해 직접 올리고자함. (디버깅을 하기위해 EC2를 씀)

## 구성한 DEV 아키텍처
![스크린샷, 2022-06-04 16-34-40](https://user-images.githubusercontent.com/50416571/172083815-ae6b97b5-b862-46fe-aab1-9c412c2d9cc0.png)


### 판단 
EC2에 compose 명령어를 쓰는 것보다 ecs에 클러스터를 ec2를 사용함으로써 개발자들의 요구사항을 충족시켜주고 좀 더 편리한 환경 

### 이유
      1. EC2가 갑자기 down되면 깡통 EC2에 docker를 다시 설치해야해서 실행 시간이 걸린다.

      2. 컨테이너 오케스트레이션을 할 수 없다,
      
      3. start.sh, stop.sh 스크립트를 따로 작성해 실행 시켜야 한다.
      
      
### ECS를 선택한 이유

      1. AWS 서비스를 잘 결합해서 컨테이너 배포에 맞도록 서비스를 만들었기 때문에 손쉽게 배포 및 운영이 가능하다.
      
      2. Amazon ECS에서 기계 학습 도구를 쓰고 싶다면 Amazon SageMaker를 쉽게 연동 가능하며 배치 작업을 위해서는 AWS Batch 등과도 쉽게 연동 가능하다.
      
      3. Git Action으로 ECR, ECS에 배포하므로 자동화가 편리해진다.
      
      4. 지금은 서비스가 많이 없지만 새로운 서비스가 더 많이 생겼을때 docker로만 관리하는것보다 더 편하다.
      
### 초기 구축 과정
    
      1. ECS 클러스터를 콘솔을 통해 구성
      
      2. ECS 작업정의를 콘솔을 통해 구성 (API Server 및 Auth Server, Redis)
      
      3. ALB를 콘솔을 통해 생성
      
      4. 작업정의를 바탕으로 ECS Service 생성
      
      5. ALB의 타겟그룹 설정
      
      6. Auth-Server의 Healthy Check는 /auth일때 200으로 설정
      
      7. Git Action으로 API-Server 및 Auth-Server을 ECR 및 ECS로 푸쉬
      
### 초기 구축과정에서 자동화로의 변화

      1. Terraform을 이용하여 VPC, Subnets, ALB 등 리소스 생성
      
      2. Terraform에서 만든 리소스를 바탕으로 Action-Secrete에 VPC, TargetGroup, Subnetes을 정의
      
      3. Git Action에서 ecs-cli 명령어를 통해 클러스터를 생성하고 API-Server와 Auth-Server, Redis에 대한 서비스,작업정의 생성 및 ECR에 업로드
      
      4. 이후 code push가 일어날 때 ecs-cli 명령어를 통해 작업정의 및 서비스만 바뀌어 ECS상에 업로드
      
      
## 요구사항 (staging, proudction)

      1. staging, proudction 환경은 쿠버네티스로 구성할 것. 
      2. staging에서 proudction으로 갈 때는 release를 통해 이루어질것.

## 구성한 Staging/Production 아키텍처
![스크린샷, 2022-06-04 16-34-37](https://user-images.githubusercontent.com/50416571/172083825-fc260538-9d58-4812-82d1-2e5c37cef433.png)

### EKS 사용이유

      1. kubernetes master node를 AWS애서 직접 고가용성 관리를 하므로 관리 운영 부담이 현저히 줄어듬
      
      2. AWS IAM 기반 인증 사용
      
      3. EKS는 기본적으로 amazon-vpc-cni-k8s 을 지원. amazon-vpc-cni-k8s 플러그인은 VPC 상에서 유효한 실제 IP를 Pod에 할당
      
      4. EKS를 사용시 다른 kubenetes를 지원하는 클라우드서비스로 마이그레이션이 용이하기 때문에 aws에 의존도가 낮아짐
      
      5. kubectl을 사용 할 수 있고 ecs에 비해 다수의 컨테이너를 관리하기에 좀 더 유리한면이 있음.
### EKS를 구성하면서 고려해야 했던 점

      1. RBAC(역할 기반 액세스 제어) 설정
          - 역할(Role) 및 역할 바인딩(RoleBinding) 생성
      
      2. ALB 설정
          - Ingress 및 Ingress Controller 설치
          - 서브넷 태그 설정
          - IAM 권한
          
      3. 노드안에서 Auth-Server와 Redis 연결
      
      
###참조문서
- eks vs ec2 사용비교
automatically.https://ikcoo.tistory.com/147
https://aws.amazon.com/ko/eks/faqs/#:~:text=Q%3A%20Amazon%20EKS%20%EC%B6%94%EA%B0%80%20%EA%B8%B0%EB%8A%A5%EC%9D%84%20%EC%82%AC%EC%9A%A9%ED%95%B4%EC%95%BC%20%ED%95%98%EB%8A%94%20%EC%9D%B4%EC%9C%A0%EB%8A%94,%ED%95%98%EA%B3%A0%20%EA%B4%80%EB%A6%AC%ED%95%A0%20%EC%88%98%20%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4.

- GitOps
https://www.samsungsds.com/kr/insights/gitops.html
