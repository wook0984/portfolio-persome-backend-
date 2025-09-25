# [팀 프로젝트] 맞춤형 뷰티 쇼핑몰 '퍼솜(PERSOME)' 백엔드 포트폴리오

![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=for-the-badge&logo=openjdk&logoColor=white)
![Spring](https://img.shields.io/badge/spring-%236DB33F.svg?style=for-the-badge&logo=spring&logoColor=white)
![JPA](https://img.shields.io/badge/JPA-A46A41?style=for-the-badge&logo=Hibernate&logoColor=white)
![MySQL](https://img.shields.io/badge/mysql-%2300f.svg?style=for-the-badge&logo=mysql&logoColor=white)
![Uploading Persome (1).png…]()

---

##  1. 프로젝트 소개

'퍼솜(PERSOME)'은 사용자의 관심사(피부 타입, 선호 성분 등)를 파악하여 최적의 뷰티 제품을 추천하는 맞춤형 쇼핑몰입니다. 
이 프로젝트는 5인 팀으로 진행되었으며, 저는 백엔드 시스템 설계 및 핵심 기능 개발을 담당했습니다.

본 레포지토리는 팀 프로젝트의 전체 소스 코드가 아닌, 저의 기여도를 명확히 보여주기 위한 포트폴리오용으로 제작되었습니다.

> 팀 프로젝트 원본 레포지토리는 Private으로 관리되고 있습니다.
> (링크: `https://github.com/C3L2-Project`)

- 개발 기간: 2025.09.08 ~ (진행 중)
- 주요 기능: 회원 관리, 상품 관리(검색, 페이징), 주문/결제, 리뷰, 위시리스트 등

---

## 2. 저의 역할 및 주요 기여

저는 이 프로젝트에서 상품(Product) 도메인의 백엔드 시스템 전체를 책임지고 설계 및 개발했습니다. 
단순한 기능 구현을 넘어, 코드의 유지보수성과 확장성을 고려한 설계에 집중했습니다.

### 관심사 분리(SoC) 원칙에 따른 체계적인 API 설계
상품 도메인 내에서도 역할과 책임이 다른 API들을 명확히 분리하여 관리했습니다.
-   `ProductController`: 상품 검색, 전체 목록 조회 등 핵심 CRUD REST API를 담당합니다.
-   `ProductHighlightController`: 메인 페이지 노출을 위한 인기/신규 상품 조회 API를 별도로 분리했습니다.
-   `ProductViewController`: 서버사이드 렌더링을 위한 페이지 이동(View) 로직을 전담합니다.
-   이러한 관심사의 분리(Separation of Concerns)를 통해 각 컨트롤러가 하나의 책임에만 집중하도록 하여, 향후 기능 변경 및 확장이 용이한 구조를 설계했습니다.

### DTO를 활용한 안정적인 데이터 전송 설계**
JPA Entity를 API에 직접 노출할 경우 발생할 수 있는 문제를 방지하기 위해, 요청(Request)과 응답(Response)의 목적에 맞는 DTO(`Data Transfer Object`)를 적극적으로 설계하고 활용했습니다.
-   Request DTO (`OrderSearchDto`):** 검색 및 페이징 조건과 같은 클라이언트의 요청 데이터를 안전하게 캡슐화합니다.
-   Response DTO (`ProductDetailResponse`, `PageProductAllResponse`): API 스펙에 필요한 데이터만 선별하여 전달함으로써, 불필요한 데이터 노출을 막고 Entity의 데이터 무결성을 보호했습니다.

### 계층형 아키텍처 기반의 비즈니스 로직 구현
Controller - Service - Repository로 이어지는 Spring의 표준적인 계층형 아키텍처를 준수하여, 각 계층이 자신의 역할에만 충실하도록 코드를 작성했습니다. 
이를 통해 비즈니스 로직의 변경이 다른 계층에 미치는 영향을 최소화했습니다.

---

## 3. 기술 스택

- Backend: Java, Spring Boot, Spring MVC, JPA
- Database:MySQL
- Infra & Tools: Git, Github, IntelliJ IDEA, Docker, Slack, Notion
- Testing: JUnit5 (학습 및 적용 예정)

---

## 4. 핵심 코드 (API 설계 및 DTO 활용 예시)

제가 가장 자신 있게 보여드릴 수 있는 부분은 견고한 API 설계를 위한 컨트롤러 역할 분리 및 DTO 활용입니다. 
아래는 상품 도메인의 핵심 컨트롤러인 `ProductController`의 코드입니다.

```java
// ProductController.java
@RestController
@RequestMapping("/api/products") // REST API 전용 URI
@RequiredArgsConstructor
public class ProductController {

    private final ProductService productService;

    // 상품 상세 정보 조회 API
    @GetMapping("/{id}")
    public ResponseEntity<ProductDetailResponse> getProductDetail(@PathVariable("id") Long productId) {
        // Service 계층에 작업 위임 후, 응답 전용 DTO로 변환하여 반환
        return ResponseEntity.ok(productService.getProductDetail(productId));
    }

    // 전체 상품 목록 페이징 조회 API
    @GetMapping()
    public ResponseEntity<PageProductAllResponse> getAllProducts(@RequestParam(defaultValue = "0") int page,
                                                                 @RequestParam(defaultValue = "24") int size) {
        // 검색 조건을 담는 DTO를 사용하여 Service 계층에 전달
        OrderSearchDto searchDto = createSearchDto(page, size);
        PageProductAllResponse allProducts = productService.getAllProducts(searchDto);
        return new ResponseEntity<>(allProducts, HttpStatus.OK);
    }
    
    // DTO 생성을 위한 private 헬퍼 메서드로 역할 분리
    private OrderSearchDto createSearchDto(int page, int size) {
        // ... 유효성 검증 로직 ...
        return OrderSearchDto.builder().page(page).size(size).build();
    }
}
위 코드에서 보시는 것처럼 저는 API의 명확한 역할 분리와 DTO를 통한 안전한 데이터 전송을 중요하게 생각합니다.
Entity가 아닌 ProductDetailResponse와 같은 DTO를 반환함으로써
API 스펙의 변경이 데이터베이스 모델(Entity)에 영향을 주지 않도록 설계했습니다
