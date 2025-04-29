---
layout: post
title:  개인 자산관리 시스템 만들기
date:   2025-04-29
last_modified_at: 2025-04-29
category: [Toy Project]
tags: [Typescript, Next.js, GraphQL, Prisma, Redis, RabbitMQ, Docker]
---

# Overview
## Purpose
월급 등 개인 자산을 관리하고 포트폴리오 수익률을 시각화하는 스프레드시트를 대체하기 위함
<br/>
<br/>


## Key Features
1. 월급관리기능
    1. 월급명세서 자동파싱([이전 포스트](https://mognex.com/2025/02/14/parsing-payroll-file)참조)
    2. 고정비 관리(연/월별 정기지출)
    3. 경조사비 관리
    4. 카드관리(전월실적, 카드혜택, 고정비 지출수단)
    5. 저축투자 비중관리(월급 중 얼마나 저축/투자 할지 계획)
2. 포트폴리오관리기능
    1. 자산구성 및 목표비율 관리
    2. 배당내역관리
    3. 자산현황 스냅샷(일별 기록)
    4. 전체자산 시각화

<br/>
<br/>


## Technology Stack
#Next.js #GraphQL #PostgreSQL #Prisma #Redis #RabbitMQ #Docker

<br/>
<br/>

## Domain

|Domain|Content|
|--|--|
|User|Login, Auth, Account, Session|
|UserPlan|월급, 지출계획, 저축투자계획|
|Portfolio|자산구성 및 목표비율관리|
|Dividend|배당내역관리|
|DailyRecord|자산 현황 스냅샷|
|Overview|전체자산 시각화|


# Trial and Error Log
* 250429
project시작

