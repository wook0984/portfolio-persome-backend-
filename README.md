# portfolio-persome-backend-
# [팀 프로젝트] 맞춤형 뷰티 쇼핑몰 '퍼솜(PERSOME)' 백엔드 포트폴리오

![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring](https://img.shields.io/badge/spring-%236DB33F.svg?style=for-the-badge&logo=spring&logoColor=white)
![JPA](https://img.shields.io/badge/JPA-A46A41?style=for-the-badge&logo=Hibernate&logoColor=white)
![MySQL](https://img.shields.io/badge/mysql-%2300f.svg?style=for-the-badge&logo=mysql&logoColor=white)

---

## 1. 프로젝트 소개

'퍼솜(PERSOME)'은 사용자의 관심사(피부 타입, 선호 성분 등)를 파악하여 최적의 뷰티 제품을 추천하는 맞춤형 쇼핑몰입니다. 이 프로젝트는 5인 팀으로 진행되었으며, 저는 백엔드 시스템 설계 및 핵심 기능 개발을 담당했습니다.

본 레포지토리는 팀 프로젝트의 전체 소스 코드가 아닌, 저의 기여도를 명확히 보여주기 위한 포트폴리오용으로 제작되었습니다.

> 팀 프로젝트 원본 레포지토리는 Private으로 관리되고 있습니다.
> (링크: `https://github.com/C3L2-Project`)
> 
- 개발 기간: 2025.09.08 ~ (진행 중)
- 주요 기능: 회원 관리, 상품 관리, 주문 및 결제, 고객센터 게시판 등

---

## 2. 저의 역할 및 주요 기여

###  동시 주문 처리 성능 최적화 (API 응답 시간 5초 → 1초 미만 단축)

[문제 상황]
타임세일 이벤트를 가정하고 JMeter로 부하 테스트를 진행한 결과, 동시 주문 요청 시 심각한 성능 저하와 재고 데이터 불일치 문제가 발생했습니다.

[원인 분석]
VisualVM으로 스레드 덤프를 직접 분석하여, 재고 차감 로직의 `synchronized` 블록에서 심각한 락 경합(Lock Contention)이 발생하고 있음을 스택 트레이스를 통해 규명했습니다. 또한, DB 커넥션 풀 부족 현상도 함께 발견했습니다.

[해결 과정]
DB 커넥션 풀 사이즈를 최적화하고, 애플리케이션 레벨의 `synchronized` 대신 데이터의 정합성을 보장하며 동시성 제어에 더 적합한 JPA의 비관적 락(Pessimistic Lock)을 도입하여 아키텍처를 개선했습니다.

*결과]
동일한 부하 테스트 환경에서 평균 API 응답 시간을 5초에서 1초 미만으로 단축**시켰으며, 데이터 불일치 문제를 해결하여 시스템의 안정성과 신뢰도를 확보했습니다.

###  API 설계 및 구현
- Spring Boot와 JPA를 사용하여 회원, 상품, 주문 관련 RESTful API 30여 개를 설계 및 구현했습니다.
- 역할과 책임을 분리하는 객체지향 설계 원칙(SOLID)을 적용하여 코드의 유지보수성과 확장성을 고려했습니다.

### 데이터베이스 모델링
- `핵심 데이터 모델링` 지식을 기반으로 사용자, 상품, 주문, 리뷰, 관심사 간의 관계를 정의하여 38개 테이블로 구성된 데이터베이스 스키마를 설계했습니다.
- 정규화를 통해 데이터 중복을 최소화하고, 주요 조회 기능에 인덱스를 적용하여 성능을 최적화했습니다.

---

## 3. 기술 스택

- Backend: Java 21, Spring Boot, Spring MVC, JPA
- Database: MySQL, Oracle 11g
- Infra & Tools: Git, Github, IntelliJ IDEA, VisualVM, Docker
- *ollaboration: Slack, Notion

---

## 4. 핵심 코드 (성능 최적화 부분)

아래는 동시성 문제를 해결하기 위해 JPA의 비관적 락(Pessimistic Lock)을 적용한 핵심 재고 차감 메서드입니다.

```java
// ProductService.java
@Transactional
public void decreaseStock(Long productId, int quantity) {
    // 1. SELECT FOR UPDATE 구문을 통해 데이터베이스 레코드에 쓰기 락(Write Lock)을 겁니다.
    //    이를 통해 다른 트랜잭션이 이 레코드에 접근하는 것을 막아 데이터 정합성을 보장합니다.
    Product product = productRepository.findByIdForUpdate(productId)
            .orElseThrow(() -> new EntityNotFoundException("상품을 찾을 수 없습니다."));

    // 2. 재고를 확인하고 차감하는 비즈니스 로직을 수행합니다.
    //    이 로직이 실행되는 동안 해당 상품 레코드는 다른 트랜잭션으로부터 보호됩니다.
    if (product.getStock() < quantity) {
        throw new IllegalStateException("재고가 부족합니다.");
    }

    product.decreaseStock(quantity);

    // 3. 트랜잭션이 커밋되면 락이 해제됩니다.
}
