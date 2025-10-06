---
title: "JMeter를 활용한 API 성능 테스트 및 개선 시도1"
categories: [개발]
tags: [Spring, cache, jpa, mysql, jmeter, 성능최적화, api, 백엔드]
---


# 서론

[🏥 CareBridge: AI 기반 간호간병통합서비스 플랫폼](https://github.com/2024-Capstone-Team/Carebridge-backend) 캡스톤 프로젝트를 진행하며 백엔드 서버를 구축했다. 개발 과정에서 특정 API의 성능 테스트를 진행했는데, 예상보다 성능이 좋지 않아 개선이 필요했다. 테스트를 위해 프로시저를 활용하여 더미데이터를 생성했다.

# 시스템 구조

## 데이터베이스 구조
Patient 테이블은 다음과 같은 스키마로 구성되어 있다.

<img width="300" alt="Image" src="/assets/images/20250522_1.webp" />

## API 구현 코드

### Controller 코드
<img width="600" alt="Image" src="/assets/images/20250522_2.webp" />

### Service 코드
<img width="700" alt="Image" src="/assets/images/20250522_3.webp" />

### 주요 쿼리
Hospital ID와 Department로 환자를 조회하는 쿼리:
<img width="600" alt="Image" src="/assets/images/20250522_4.webp" />

의료진 ID로 의료진을 조회하는 쿼리:
<img width="600" alt="Image" src="/assets/images/20250522_5.webp" />

# 성능 테스트 진행

CareBridge는 입원 환자를 대상으로 하는 서비스다. 국내 입원이 가능한 종합병원은 약 347개이며, 보건복지부에 따르면 상급종합병원은 47개가 있다. 병원 가동률과 병상 수를 바탕으로 추정해보면 실제 입원 환자는 45,000~55,000명 정도로 예상된다. 

테스트를 위해 실제 입원 환자와 분과별 비율에 맞춰 약 1/10~1/13 비율로 더미데이터를 생성했으며, 약 4,000개의 데이터를 사용했다. 그럼에도 불구하고 쿼리 성능이 좋지 않았고 병목 현상도 발생했다.

## JMeter를 활용한 API 응답시간 체크

JMeter를 활용해 API의 응답속도를 측정했다. 테스트 설정은 다음과 같다:
- Number of threads: 47
- Ramp-up period: 10
- Loop Count: 5

<img width="600" alt="Image" src="/assets/images/20250522_6.webp" />

테스트 결과, 평균 응답시간은 91ms, 최대 응답시간은 351ms로 측정되었다. 데이터 양이 실제 환경보다 적음에도 불구하고 중간중간 병목이 발생하는 지점이 존재했다.

# API 성능 개선 방안

## 고려한 사항

1. **인덱스 최적화**
   Patient 테이블에는 이미 5개의 인덱스가 존재했다:
   - patient_id (PK 인덱스)
   - phone_num (Unique 인덱스)
   - user_id, guardian_contact, chatroom_id (일반 인덱스 3개)
   
   Hospital ID와 Department로 환자를 조회하는 쿼리를 최적화하기 위해 복합 인덱스 생성을 고려했으나, 병원 수가 많지 않아 오히려 성능 하락 리스크가 있었다.

2. **서비스 특성 분석**
   서비스 특성상 CREATE 작업보다 UPDATE와 SELECT 작업이 대부분을 차지했다. 이러한 서비스 특성을 고려했을 때 캐싱이 가장 효과적인 방법이라고 판단했다.

3. **캐싱 방법 선택**
   Redis를 활용한 캐싱 방법을 검토했으나, 현재 상황에서 Redis 도입 시 여러 변경사항과 추가 비용이 발생하는 문제가 있었다. 따라서 Spring Cache를 활용한 Lazy-Loading 방식으로 캐싱을 구현하기로 결정했다.

   이 방법은 Spring Boot 애플리케이션 종료 시 캐시가 사라지는 문제와 캐시된 데이터와 DB 데이터 간 정합성 이슈가 있지만, 데이터 업데이트 시 캐시를 갱신하는 방법으로 정합성 문제를 해결했다. Patient, Hospital, MedicalStaff 엔티티는 업데이트가 거의 발생하지 않기 때문에 이 방법이 효율적이라고 판단했다.

## 캐싱 적용 코드

Hospital ID와 Department로 환자 조회 캐싱 적용 코드:
<img width="600" alt="Image" src="/assets/images/20250522_7.webp" />

의료진 조회 캐싱 적용 코드:
<img width="600" alt="Image" src="/assets/images/20250522_8.webp" />


# 개선 결과

캐싱 적용 후 API 응답속도가 크게 개선되었다:
<img width="600" alt="Image" src="/assets/images/20250522_9.webp" />

캐싱 적용 후 평균 응답시간은 47ms로, 기존 91ms 대비 약 48% 향상되었다. 최고 응답시간이 300ms대로 기록된 경우는 캐시를 처음 저장하는 과정에서 발생했다. 캐시가 한 번 생성된 이후에는 DB 접근 시간이 거의 없어져 대부분의 요청이 매우 빠르게 처리되었다.
