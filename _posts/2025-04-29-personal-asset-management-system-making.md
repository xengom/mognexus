---
layout: post
title:  개인 자산관리 시스템 만들기
date:   2025-04-29
last_modified_at: 2025-08-13
category: [Toy Project]
tags: [Typescript, React, Vite, Radix, Cloudflare, Drizzle]
---

# Overview
## Purpose
월급 등 개인 자산을 관리하고 포트폴리오 수익률을 시각화하는 스프레드시트를 대체하기 위함
<br/>
<br/>


## Key Features
1. 복식부기 가계부
2. 계정관리
3. 자산관리(투자추적)


<br/>
<br/>


## Technology Stack
- **Front**: React + TypeScript + Vite + Radix UI
- **Back**: Cloudflare Pages Functions
- **DB**: Cloudflare D1 + Drizzle ORM  
- **Validation**: Zod
- **Hosting**: Cloudflare Pages
- **Architect**: Functional Programming을 기반으로 설계할 것.

<br/>
<br/>

## 아키텍처 구성도
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   프론트엔드     │    │  CF Workers     │    │   외부 API      │
│   React + TS    │◄──►│   API Layer     │◄──►│  Yahoo Finance  │
│   Shadcn UI     │    │   Drizzle ORM   │    │   기타 소스     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
                       ┌─────────────────┐
                       │  Cloudflare D1  │
                       │   SQLite DB     │
                       └─────────────────┘
```



<br/>
<br/>

<br/>
<br/>
# Trial and Error Log
* 250813
개발 시작

