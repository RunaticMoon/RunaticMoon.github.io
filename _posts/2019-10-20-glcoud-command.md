---
title: "[GCP] gcloud 명령어 정리"
date: 2019-10-20 10:48 +09:00
categories:
  - GCP
tags:
  - Compute Engine
  - Kubernetes
  - Load Balancer
toc: true
---

## [GCP] gcloud 명령어

### [Compute Engine] Create Instance

```bash
gcloud compute instances create [instance_name] --machine-type [machine_type] --zone [your_zone]
```



### [Compute Engine] 기본 Zone/Region 설정

```bash
# 기본 Zone 설정
gcloud config set compute/zone [default_zone]
google-compute-default-zone [default_zone]

# 기본 Region 설정
gcloud config set compute/region [default_region]
google-compute-default-region [default_region]
```



### [Compute Engine] SSH 연결

```bash
gcloud compute ssh [instance_name] --zone [YOUR_ZONE]
```



### [Gcloud] 사용중인 계정 이름 목록

```bash
gcloud auth list
```



### [Gcloud] 프로젝트 ID 목록

```bash
gcloud config list project
```



### [Compute Engine] Windows 시작 상태 테스트

```bash
gcloud compute instances get-serial-port-output [instance_name] --zone [your_zone]
```



### [Compute Engine] 기본 리전 및 영역 확인

```bash
gcloud compute project-info describe --project [your_project_ID]
```



### [Gcloud] 환경 변수 설정

```bash
export PROJECT_ID=[your_project_ID]
```



### [Gcloud] 대화형 모드 (베타)

```bash
# 설치
gcloud components intall beta

# 전환
gcloud beta interactive
```



### [Kubernetes] 클러스터 만들기

```bash
gcloud container clusters create [CLUSTER-NAME]
```



### [Kubernetes] 클러스터의 사용자 인증 정보 얻기

```bash
gcloud container clusters get-credentials [CLUSTER-NAME]
```



### [Kubernetes] 클러스터에 애플리케이션 배포하기

```bash
# image_url은 dockerhub이나 google container registry의 링크
# version 미지정시 최신버전 사용
kubectl run [app_name] --image=[image_url]:[version] --port [port_number]
```



### [Kubernetes] 서비스 실행하기

```bash
kubectl expose deployment [app_name] --type="LoadBalancer"
```



### [Kubernetes] 서비스 실행되는지 검사하기

```bash
kubectl get service [app_name]
```



### [Kubernetes] 클러스터 삭제하기

```bash
gcloud container clusters delete [CLUSTER-NAME]
```



### [Compute Engine] 인스턴스 템플릿 만들기

```bash
gcloud compute instance-templates create [template_name] \
         --metadata-from-file startup-script=[script_name]
```



### [Compute Engine] 대상풀 만들기

```bash
gcloud compute target-pools create [pool_name]
```



### [Compute Engine] 인스턴스 목록 출력하기

```bash
gcloud compute instances list
```



### [Compute Engine] 방화벽 구성 (포트 열기)

```bash
gcloud compute firewall-rules create [firewall_name] --allow tcp:[port_number]
```



### [Compute Engine] 인스턴스 템플릿을 사용하는 관리형 인스턴스 그룹 만들기

```bash
gcloud compute instance-groups managed create nginx-group \
         --base-instance-name [instance_name] \
         --size [instance_size] \
         --template [template_name] \
         --target-pool [pool_name]
```



### [Load Balancer] L3 네트워크 부하 분산기 생성

```bash
gcloud compute forwarding-rules create nginx-lb \
         --region [your_region] \
         --ports=[port_number] \
         --target-pool [pool_name]
```



### [Compute Engine] 모든 인스턴스의 전달 규칙목록 출력

```bash
gcloud compute forwarding-rules list
```



### [Compute Engine] HTTP 트래픽 상태확인 생성

```bash
gcloud compute http-health-checks create http-basic-check
```



### [Compute Engine] 인스턴스 그룹에 관한 HTTP 서비스 정의

```bash
gcloud compute instance-groups managed \
       set-named-ports nginx-group \
       --named-ports http:80
```



### [Compute Engine] 백엔드 서비스 생성

```bash
gcloud compute backend-services create nginx-backend \
      --protocol HTTP --http-health-checks http-basic-check --global
```



### [Compute Engine] 백엔드 서비스에 인스턴스 그룹 추가

```bash
gcloud compute backend-services add-backend nginx-backend \
    --instance-group nginx-group \
    --instance-group-zone us-central1-a \
    --global
```



### [Compute Engine] 기본 URL 맵 생성

```bash
gcloud compute url-maps create web-map \
    --default-service nginx-backend
```



### [Compute Engine] HTTP 프록시 생성하여 URL맵에 라우팅

```bash
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map
```



### [Compute Engine] 요청을 라우팅하는 글로벌 전달규칙 생성

```bash
gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80
```



### [Compute Engine] 전달규칙 확인하기

```bash
gcloud compute forwarding-rules list
```