# 핵심 키워드

## Domain
- **도메인**
    - 소프트웨어로 해결하고자 하는 문제 및 관심사
- **도메인 모델**
    - 일반적으로 UML 다이어그램과 같이 표현된 개념 모델
- **도메인 객체 모델**
    - 도메인 모델을 기반으로으로 한 구체적인 클래스와 객체 구조
- **DDD(domain driven development, 도메인 주도 개발)**
    - 도메인 모델을 중심으로 소프트웨어를 개발 설계하는 접근법
    - 목표: 높은 응징력과 낮은 결합도로 도메인을 구성
    - 장점: 비즈니스 중심 설계를 통해 유연하고 이해하기 쉬운 구조로 만들 수 있음
    - 핵심 개념
        1. Ubiquitous Language
            - 도메인 전문가와 개발자가 동일항 용어와 개념을 사용하도록 합의하고 이를 모델과 코드에 반영
        2. Bounded Context
            - 도메인이 여러 하위 영역으로 나뉘어 있을 때 각 영역을 경계로 구분하고 독립된 모델을 유지하는 개념
        3. 도메인 모델
            - 도메인 지슥을 코드 형태로 체계화한 추상화 모델
            - Entity, value object, service, repository 등의 구성 요소로 이루어짐
        4. Entity와 Value Object
            - Entity: 고유 식별자를 갖고, 라이프 사이클 전반에 걸쳐 상태가 변할 수 있는 객체
            - Value Object: 식별자 없이 불변성을 띠며, 동일한 속성이면 같은 객체로 취급
        5. Aggregate
            - Entity와 value object를 그룹화한 객체군
        6. Domain Service
            - 모델 객체(entity, value object)로 표현하기 어려운 도메인 규칙이나 로직을 담당
        7. Repository
            - 엔티티나 애그리거트를 저장하고 검색하는 추상화 계층
            - 역할: persistence 로직을 캡슐화하여 도메일 로직과 데이터 접근 로직 분리

---
##  양방향 매핑

- 개념
    - 연관관계 주인이 아닌 엔티티에도 `mappedBy`를 설정해, 양쪽에서 서로를 참조할 수 있도록 구성하는 방식
- 장점: **객체 그래프 탐색으로 인한 이점**
    - 양방향 매핑을 하면  A→B 뿐만 아니라 B→A로도 접근할 수 있어 양쪽 방향으로 객체를 탐색할 수 있음
- 단점: **순환 참조 문제**
    - 순환 참조: 두 개 이상의 객체가 서로 참조함으로써 의존 관계에 사이클이 생기는 상황
    - 순환 참조로 인해 시스템에 무한 루프가 발생할 수 있음
    - 예) JSON 직렬화
        - Mebmer와 Order의 1:N 관계에서 Member 객체를 API응답으로 반환할 때 스프링은 Member 객체를 JSON 문자열로 바꾸는 작업(직렬화) 수행할 때
        - Member 안에 orders(Order 객체의 리스트)가 있음 → orders 안에 Order가 있음 → Order 안에 member가 있음 ⇒ 끝없이 JSON을 생성하려다보니 StackOverflowError 발생
- 연관관계 편의 메서드
    - **양방향 연관관계**일 경우, 객체 A와 객체 B가 서로를 참조해야 하기 때문에 **양쪽 모두 관계를 설정해줘야 한다.**
    - 만약 한 쪽만 관계를 설정하면, **객체 상태는 불일치하게 되고**, 디버깅이 어려운 버그를 초래할 수 있다.
    - 그래서 **한 메서드에서 양쪽 객체 모두의 연관관계를 설정**하는 것이 안전하다.
    - 일반적으로는 **더 자주 사용하는 쪽**(예: 부모 → 자식)에서 이 메서드를 정의하는 것이 좋다.




---
## N+1 문제
- 정의
    - ORM에서 자주 발생하는 성능 문제로 1번의 쿼리로 N개의 데이터를 가져온 후, 그 N개의 각 항목에 대해 추가 쿼리가 N번 더 실행되는 현상
    - 성능 저하, 불필요한 DB 부하, 느린 응답 속도 유발
- 원인
    - 관계형 데이터베이스와 객체지향 언어간의 패러다임 차이로 인해 발생한다.
    - 객체는 연관 객체를 레퍼런스를 통해 메모리 내에서 즉시 접근(Random Access) 할 수 있지만,

      관계형 데이터베이스에서는 ****SELECT 쿼리를 통해서만 연관 데이터를 조회할 수 있다.

    - 이 차이로 인해, 연관 객체를 접근할 때마다 매번 추가 쿼리가 발생하게 되어 성능 문제가 생긴다.
- 해결방법
    1. **Fetch Join (JPQL)**
        - `JOIN FETCH`를 사용하면 **연관된 엔티티를 함께 한 번에 조회**함으로써 N개의 추가 쿼리 방지
        - 예시 코드

            ```jsx
            SELECT m FROM Member m JOIN FETCH m.orders
            ```

    2. **@BatchSize**
        - 한 번에 여러 개의 ID 값을 `IN` 조건으로 묶어서 **N개의 쿼리를 1~2개로 줄이는 방식**
        - 예시 코드

            ```jsx
            @OneToMany(mappedBy = "member")
            @BatchSize(size = 100)
            private List<Order> orders;
            ```

    3. **EntityGraph**
        - JPA에서 제공하는 기능으로, **어떤 연관 필드를 함께 로딩할지 명시**
        - 리포지토리 인터페이스에서 직접 fetch 설정 가능
        - 예시 코드

            ```jsx
            @EntityGraph(attributePaths = {"orders"})
            List<Member> findAll();  // Member와 Order를 함께 로딩
            ```