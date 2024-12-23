# 도메인 분석 설계

![스크린샷 2024-11-12 145800.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/8082c7c0-79c7-4547-ba2b-78142b6ffb42/c864bd7a-8122-4c36-955a-2ee915678cd9/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2024-11-12_145800.png)

### Lombok이란?

<aside>
💡

**Lombok의 기능은 model 클래스나 Entity 같은 도메인 클래스 등에 반복되는 getter, setter, toString 등의 메소드를 자동으로 만들어주는 기능을 합니다.**

</aside>

---

### 단방향, 양방향 그리고 연관관계 주인

<aside>
💡

**단방향 : 객체 관계에서 한 쪽만 참조하는 것**

</aside>

<aside>
💡

**양방향 : 객체 관계에서 양쪽이 서로 참조하는 것**

</aside>

데이터 베이스는 외래키의 하나로 두 개의 테이블을 연관관계를 맺는다. 이때 데이터 베이스의 연관관계를 관리하는 지점은 외래키 하나인 반면에 엔티티를 양방향으로 매핑하면 두 개의 객체는 서로 참조 해서 객체의 연관 관계를 관리하는 지점은 2개가 된다.

<aside>
💡

**연관관계의 주인은 두 객체의 연관관계 중 하나를 정해서 데이터베이스의 외래키를 관리하는 것이다**

→연관관계 주인은 `@JoinColumn(name = “참조하는 테이블 기본키”)`

</aside>

- **1:N or N:1일 경우** : `N`이 연관관계 주인
- **1:1일 경우** : 주테이블이 연관관계 주인
- **M:N일 경우** : M과 N을 연결해주는 테이블을 추가하여 1:M, N:1을 만들어 **M과 N이 연관관계 주인**

### 일대다 관계(1:N)

<aside>
💡

**1:N 관계는 한 쪽 엔티티가 관계를 맺은 엔티티 쪽의 여러 객체를 가질 수 있는 것을 의미한다. 즉 테이블의 A의 한 개의 행이 테이블 B의 여러개의 행과 연결되는 관계이다**

</aside>

![스크린샷 2024-11-12 150013.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/8082c7c0-79c7-4547-ba2b-78142b6ffb42/948fa8fb-64f2-4e7e-b395-48ee49fb6f7e/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2024-11-12_150013.png)

### 다대다 관계(N:M)

<aside>
💡

**N:M 관계는 관계를 가진 양쪽 엔티티 모두에서 1:N관계가 존재할 때 나타나는 모습이다. 즉 서로가 서로를 1:N관계로 보고 있다.**

~~근데 실무에서는 잘 안 씀~~

</aside>

![스크린샷 2024-11-12 152809.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/8082c7c0-79c7-4547-ba2b-78142b6ffb42/cc27bb4e-2515-4be8-8507-acd6d006e9fa/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2024-11-12_152809.png)

→ **각 테이블의 PK를 FK로 참조하고 있는 매핑 테이블을 추가하여 일대다, 다대일의 관계로 풀어내야한다**

![스크린샷 2024-11-12 152912.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/8082c7c0-79c7-4547-ba2b-78142b6ffb42/0c75feb9-18c5-4c8d-898b-3c36014c844c/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7_2024-11-12_152912.png)

---

## 지연로딩 vs 즉시로딩

### FetchType

<aside>
💡

**FetchType이란, JPA가 하나의 Entity를 조회할 때, 연관관계있는 객체들을 어떻게 가져올 것이냐를 나타내는 설정값이다**

</aside>

→fetch의 디폴드 값은 @xxToOne에서는 EAGER(즉시), @xxToMany에서는 LAZY(지연)

### 지연로딩

<aside>
💡

**지연로딩이란 필요한 시점에 연관된 객체의 데이터를 불러오는 것이다.**

</aside>

### 즉시로딩

<aside>
💡

**즉시로딩이란 데이터를 조회할 때, 연관된 모든 객체의 데이터까지 한 번에 불러오는 것이다**

</aside>

→ **즉시로딩은 예측이 어렵과 어떤 SQL이 실행되는지 추적이 어려우므로 모든 연관관계는 지연로딩으로 설정하자!**

→ ~~실무에서도 모든 연관관계는 지연로딩으로 설정함~~

---

## 1주차 어노테이션 정리

- **@Entity**
    - **설명**: 이 클래스가 JPA 엔티티임을 나타내며 JPA가 데이터베이스 테이블과 매핑할 수 있도록 한다.
    - **기능**: 해당 클래스를 JPA가 관리하는 엔티티로 인식하게 한다.
- **@Table(name = "orders")**
    - **설명**: 엔티티가 매핑될 데이터베이스 테이블의 이름을 지정한다.
    - **기능**: 테이블 이름에 해당 엔티티가 저장되도록 매핑한다.
- **@Getter / @Setter**
    - **설명**: Lombok 라이브러리에서 제공하는 어노테이션으로 클래스의 모든 필드에 대해 getter와 setter 메서드를 자동으로 생성한다.
- **@Id**
    - **설명**: 엔티티의 기본 키를 나타냄
    - **기능**: `id` 필드를 해당 엔티티의 기본 키로 설정한다.
- **@GeneratedValue**
    - **설명**: 기본 키의 생성 전략을 지정한다.
    - **기능**: `id` 필드 값이 자동으로 생성되도록 설정한다.
- **@Column(name = "order_id")**
    - **설명**: 엔티티의 필드를 데이터베이스 테이블의 특정 열에 매핑한다.
    - **기능**: `id` 필드를 데이터베이스의 `order_id` 열에 매핑한다.
- **@ManyToOne(fetch = FetchType.LAZY)**
    - **설명**: 다대일 관계를 매핑하고 지연 로딩을 사용하여 관련 엔티티를 필요할 때 로드하도록 설정한다.
- **@JoinColumn(name = "member_id")**
    - **설명**: 외래 키를 매핑하고 데이터베이스 열 이름을 지정합니다.
    - **기능**: `member_id` 열을 외래 키로 지정하여 `Member` 엔티티와 연결한다.
- **@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)**
    - **설명**: 일대다 관계를 설정하며 `OrderItem` 엔티티에서 관계의 주인을 `order`로 지정한다. 그리고 모든 변경 사항(추가, 삭제 등)이 전파되도록 설정한다.
- **@OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)**
    - **설명**: 일대일 관계를 설정하며 모든 변경 사항이 전파되도록 하고 지연 로딩을 사용하여 필요할 때 관련 엔티티를 로드한다..
- **@JoinColumn(name = "delivery_id")**
    - **설명**: 외래 키 열의 이름을 지정하여 `Delivery` 엔티티와 연결한다.
    - **기능**: `delivery_id` 열을 외래 키로 지정하여 `Delivery` 엔티티와 연결한다.
- **@Enumerated(EnumType.STRING)**
    - **설명**: `enum` 타입을 데이터베이스에 문자열로 저장하도록 설정한다.
- **@Inheritance(strategy = InheritanceType.SINGLE_TABLE)**
    - **설명**: 엔티티 상속 전략을 지정한다.
    - `SINGLE_TABLE` 은 상속받은 모든 엔티티를 하나의 테이블에 저장하는 방식이다.
    - **기능**: 상속 구조를 가진 엔티티들을 하나의 테이블에 저장하여 테이블이 여러 개로 분할되지 않게 한다. 각 엔티티는 구분 열을 통해 개별적으로 식별된다
- **@DiscriminatorColumn(name = "dtype")**
    - **설명**: `SINGLE_TABLE` 전략을 사용할 때 엔티티 타입을 구분하는 열을 지정한다.
    - **기능**: 테이블에 `dtype`이라는 이름의 열을 추가하여 각 레코드가 어떤 서브 엔티티에 속하는지를 구분하게 한다
- **@DiscriminatorValue**
    - **설명** : 상속 관계에서 사용되는 JPA 어노테이션으로 단일테이블에서 각 자식 클래스의 구분값을 설정하는 데 사용된다. 이 어노테이션을 통해서 상속된 엔티티가 어떤 클래스인지 구분할 수 있다.
- **@Embeddable**: 값 타입으로 정의할 클래스에 사용합니다.
- **@Embedded**: `@Embeddable` 클래스의 객체를 엔티티에 포함할 때 사용하며, 해당 필드들이 엔티티 테이블의 컬럼으로 매핑됩니다.