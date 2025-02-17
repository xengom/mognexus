---
layout: post
title:  롤 내전 전적 검색서비스 만들기  
date:   2024-05-02
last_modified_at: 2025-02-17
category: [Toy Project]
tags: [Javascript, Node.js, Express.js, Raspberry Pi]
---

# Overview

## Purpose
리그 오브 레전드 클랜 내의 내전 기록을 남기고, 이를 기반으로 각 유저의 내전 전적을 검색할 수 있도록 하여 원활한 내전 진행을 촉진하기 위함.

<br/>

## Key Features
1. Replay 파일을 Discord 채널을 통해 업로드 시, 해당 게임의 결과를 DB에 저장
2. `!전적`, `!통계` 등의 명령어를 통해 저장된 Data를 가공해서 조회
3. `!탈퇴`, `!복귀`, `!닉변` 등의 관리자용 명령어를 통해 클랜원 관리

<br/>
<br/>
## Technology Stack
1. figma (design)
2. javascript & express.js (backend)
3. next.js (frontend)
4. jira & confluence & git (project managing)
5. AWS Lightsail (Deploy)

<br/><br/>
# Trial and Error Log
## Cloud 도입을 위한 과정
초기 개발 진행 단계 당시, 개발자 데스크탑 Local로 서비스를 제공하고 있었기 때문에 서비스를 이용하려면 반드시 개발자 데스크탑이 구동중이였어야 했다.
<br/>
다만 이 서비스가 어느 정도의 리소스를 필요로하고 어느 정도의 비용이 청구될지 알 수 없었기 때문에 우선 Raspberry Pi에 올려서 24시간 구동하되, 가능한 범위 안에서 서비스를 경량화하여 Cloud로 올리고자 했다.
<br/><br/>
### 1. Docker 도입
우선 추후 진행할 Cloud 도입를 용이하게 하기위해 Docker를 이용하여 배포하는 방식으로 변경하였다. <br/>
배포 순서는 다음과 같다.
1. Docker Image로 Build
2. Docker Hub로 Push
3. Raspberry Pi에서 Docker Hub의 Image 를 Pull
4. 해당 Image를 Container로 실행


<br/><br/>

### 2. 1차 경량화
Raspberry Pi 상에서 서비스를 제공해본 결과, `Spring Boot`와 `PostgreSQL`를 구동했을 때, Idle상태에서도 메모리를 약 1.2GB(캐시포함 2GB)정도 사용하였다. 따라서 `Spring Boot`에 비해 비교적 가벼운 `FastAPI(Python)`로 서비스를 Porting하였다.

| -      | Spring Boot | FastAPI     |
| ------ | ----------- | ----------- |
| CPU사용량 | 5 - 10%     | 1 - 5%      |
| RAM사용량 | 1 - 2GB     | 300 - 500MB |


<br/><br/>

### 3. Docker Image Build 시간 Issue
`Spring Boot`를 사용할 때는 크게 문제가 없었으나 `FastAPI`로 Porting 후, Docker Image Build시 `pip install`단계에서 상당한 시간이 소요되었다. (약 1~10분) 따라서 Docker 를 배제하고, 수정한 소스를 `tar.gz` 파일로 압축하여 ssh를 통해 Raspberry Pi 서버로 밀어넣고, 서버에서 압축 해제하여 직접 실행하는 방식으로 변경하였다.

<br/><br/>

### 4. Front 개발 & JS기반 환경으로의 Porting
추후 OP.GG와 같은 전적 검색 Web Site를 만들자는 의견이 나와서 `React`기반으로 개발이 진행되었다. 이 경우 Front는 `Javascript`, Back은 `Python`으로 개발되어 신규 개발인원은 두 언어를 모두 학습해야하기 때문에 Backend Service도 `Javascript`기반의 `Express.js`로 변경하기로 합의하였다.
<br/>
변경에 따른 리소스 사용량은 다음과 같다.

| -      | Spring Boot | FastAPI     | Express.js    |
| ------ | ----------- | ----------- | ------------- |
| CPU사용량 | 5 - 10%     | 1 - 5%      | 1-3%          |
| RAM사용량 | 1 - 2GB     | 300 - 500MB | 300MB - 400MB |

<br/><br/>
### 5. 비용 Issue 와 Cloud Service 선택
최종적으로 서비스를 직접 사용중인 클랜 운영진과 합의하여 Cloud 사용 비용을 후원받기로 하였다. 다만 어디까지나 운영진의 사비 또는 클랜원의 후원으로 운영될 예정인 만큼, 최대한 Cloud 비용을 절감할 방안이 필요했다. 모든 것을 AWS에서만 사용하면 편하게 환경설정할 수 있고 일괄 관리가 가능하지만 최대한 Cost cut하기 위하여 다음 서비스들을 검토했었다.

| No. | Frontend         | Backend            | Database      | Discord Bot        | Cost |
| --- | ---------------- | ------------------ | ------------- | ------------------ | ---- |
| 1   | AWS CloudFront   | AWS EC2            | AWS RDS       | AWS EC2            | $50↑ |
| 2   | vercel(next.js)  | AWS Lightsail      | Cloudflare D1 | AWS Lightsail      | $10↑ |
| 3   | Cloudflare Pages | Cloudflare workers | Cloudflare D1 | Cloudflare workers | $0↑  |

<br/>

우선 `AWS RDS`는 가장 레퍼런스도 많고 사용하기도 쉬웠으나 가장 비싼 요금을 자랑했다. 사용한만큼만 낸다고는 하지만 기본요금도 무시할 수 없기 때문에 1번안은 제외하였다.
<br/><br/>
2번안의 경우 Backend와 Discord Bot을 저렴한 `AWS Lightsail`로 옮기고 나머지를 무료 서비스로 제공하는 안이였으나 관리포인트가 너무 많아졌기 때문에 제외하였고, 3번안을 진지하게 진행하였으나, `Cloudflare D1`은 특성상 SQLite만 지원하기 때문에 SQLite로의 DB Porting이 필요했으며, 무엇보다 `Cloudflare Workers` 무료 요금제의 경우, 요청은 하루 10만건으로 매우 넉넉했으나, 요청당 CPU 처리 시간이 10ms로 정말 단순한 요청만 처리가 가능했다.
<br/><br/>
실제로 개인 서비스를 `Cloudflare Workers`로 Porting하여 2 주정도 사용해본 결과 단순 요청들은 문제없이 사용 가능했으나 Promise를 사용한 무거운(?) 요청의 경우 간간히 Fail했었고, 무엇보다도 `Cloudflare Workers`의 런타임은 Node.js가 아닌, Browser와 동일한 V8 엔진 (libuv나 각종 내장 모듈이 없다!)기반이기 때문에 원하는 package를 마음대로 가져와 쓰는 것도 불가능 했다.
<br/>
따라서 어느정도 지출을 감수하고 AWS Lightsail로 전부 통일하여 한 달 $30 정도로 비용을 조정하였다.

<br/><br/>
