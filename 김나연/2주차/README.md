## 섹션4.

<img width="954" alt="스크린샷 2024-11-19 오후 9 49 37" src="https://github.com/user-attachments/assets/5051df58-7da3-458a-9b62-d56ff6e8c5c8">


### MVC 모델

- **Model**: 데이터 및 비즈니스 로직
- **View**: 사용자 인터페이스
- **Controller**: 사용자 요청 처리 및 비즈니스 로직 연결
1. **Web 계층 (Controller와 View)**
    - **Controller**는 사용자의 요청을 받아 처리하고, 적절한 서비스를 호출하여 데이터를 가져옵니다.
    - 처리된 결과를 **View**로 전달해 사용자 인터페이스를 렌더링합니다.
    - 패키지 구조에서는 `web`에 해당하며, Spring MVC의 컨트롤러와 뷰 템플릿 코드가 위치합니다.
2. **Service 계층 (Model의 일부)**
    - **Service**는 비즈니스 로직과 트랜잭션을 처리하며, 도메인 로직을 활용해 데이터를 조작합니다.
    - 컨트롤러에서 받은 요청을 구체적으로 처리하거나, 데이터를 생성/변환해 반환합니다.
    - 패키지 구조에서는 `service`에 해당하며, 비즈니스 로직과 서비스 관련 코드를 분리해 모듈화합니다.
3. **Repository 계층 (Model의 일부)**
    - **Repository**는 데이터베이스 접근과 관련된 기능을 처리합니다.
    - 데이터베이스와 상호작용하기 위한 JPA나 엔티티 매니저 사용 코드가 위치하며, 서비스 계층에서 호출됩니다.
    - 패키지 구조에서는 `repository`에 해당합니다.
4. **Domain 계층 (Model의 중심)**
    - **Domain**은 엔티티 및 핵심 비즈니스 로직을 포함하는 부분으로, 애플리케이션의 중심적인 데이터 구조와 규칙을 정의합니다.
    - 모든 계층에서 사용될 수 있도록 설계되며, 서비스나 레포지토리에서 호출됩니다.
    - 패키지 구조에서는 `domain`에 위치합니다.

<aside>
💡

왜 계층화시킬까?(패키지 구조를 나눈 이유)

</aside>

- **관심사의 분리**
    - 코드의 책임을 계층별로 분리함으로써 각 패키지가 맡는 역할이 명확해집니다.
- **유지보수 용이성**
    - 프로젝트가 커질수록 각 계층에 맞는 코드를 모아두면 유지보수가 쉬워집니다.
    - 한 계층의 코드 수정이 다른 계층에 최소한의 영향을 주도록 설계합니다.
- **재사용성**
    - 계층화된 구조는 각 계층이 독립적으로 테스트 및 재사용 가능하도록 돕습니다.

## 섹션5.

**회원 레포지토리**

```java
@Repository
public class MemberRepository {
     @PersistenceContext
     private EntityManager em;
     public void save(Member member) {
         em.persist(member);
}
  
public Member findOne(Long id) {
    return em.find(Member.class, id);
}
public List<Member> findAll() {
    return em.createQuery("select m from Member m", Member.class)
            .getResultList();
}
public List<Member> findByName(String name) {
    return em.createQuery("select m from Member m where m.name = :name",
Member.class)
    }
}
```

**기술 설명**

- `@Repository`
    - 스프링 빈으로 등록, JPA 예외를 스프링 기반 예외로 예외 변환
    - DataBase에 접근하는 method를 가지고 있는 Class에서 쓰임
- `@PersistenceContext`
    - `EntityManager` 주입
    - Entity를 영구 저장하는 환경, 실제로 DB에 저장하는 게 아니라 영속성 컨텍스트를 통해서 Entity를 영속화함
- `@PersistenceUnit`
    - `EntityManagerFactory` (`EntityManager` 객체를 생성하는 팩토리) 주입

**회원 서비스**

```java
@Service
@Transactional(readOnly = true)
public class MemberService {
    @Autowired
    MemberRepository memberRepository;

		@Transactional
		public Long join(Member member) {
				validateDuplicateMember(member); //중복 회원 검증 
				memberRepository.save(member);
				return member.getId();
		}
    private void validateDuplicateMember(Member member) {
        List<Member> findMembers =
				 memberRepository.findByName(member.getName());
        if (!findMembers.isEmpty()) {
						throw new IllegalStateException("이미 존재하는 회원입니다."); 
				}
		}
		
    public List<Member> findMembers() {
		    return memberRepository.findAll();
		}
    public Member findOne(Long memberId) {
        return memberRepository.findOne(memberId);
		} 
}
```

**기술 설명**

- `@Service`: 비즈니스 로직을 수행하는 Class라는 것을 나타내는 용도
- `@Transactional`
    - 트랜잭션, 영속성 컨텍스트
    - ACID-트랜젝션의 성질(원자성, 일관성, 독립성, 지속성) 트랜잭션과 관련된 작업들이 부분적으로 실행되다가 중단되지 않는 것을 보장하며, 트랜잭션 처리 전과 처리 후 데이터 모순이 없는 상태를 유지하고, 트랜잭션을 수행 시 다른 트랜잭션의 연산 작업이 끼어들지 못하도록 보장하며, 성공적으로 수행된 트랜잭션은 영원히 반영되어야 함을 의미한다.(트랜잭션의 작동원리나 흐름에 대해서는 다음에 찾아보겠습니다..ㅎ)
    `readOnly=true` - 데이터의 변경이 없는 읽기 전용 메서드에 사용, 영속성 컨텍스트를 플러시 하지 않으므로 약간의 성능 향상(읽기 전용에는 다 적용)
    - 데이터베이스 드라이버가 지원하면 DB에서 성능 향상
- `@Autowired`
    - 생성자 Injection에 많이 사용, 생성자가 하나면 생략 가능
    - 스프링 컨테이너에 의해 관리되는 빈(Bean)을 자동으로 주입하여, 개발자가 객체 생성 및 관리 작업을 줄이고 애플리케이션의 구조를 간결하게 유지할 수 있도록 돕는다
    - 타입(Type)을 기준으로 빈을 찾아 주입(생성자, setter, 필드)

<aside>
💡

**참고:** 실무에서는 검증 로직이 있어도 멀티 쓰레드 상황을 고려해서 회원 테이블의 회원명 컬럼에 유니크 제약 조건을 추가하는 것이 안전하다.

</aside>

<aside>
💡

필드 주입보다는 생성자 주입을 사용하는 것을 권장

</aside>

**필드 주입 방식**

```java
public class MemberService {
    @Autowired
    private MemberRepository memberRepository;
}
```

**생성자 주입 방식**

```java
public class MemberService {
    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

## 생성자 주입을 권장하는 이유

### 1. **불변성(Immutable) 보장**

- 생성자 주입 방식에서는 주입된 의존성에 `final`을 사용할 수 있습니다.
- `final` 필드는 한 번만 초기화되고 변경할 수 없으므로, 객체의 상태가 안정적이고 예기치 않은 변경을 방지할 수 있습니다.

### 2. **테스트 용이성**

- 생성자 주입은 테스트 코드에서 **Mock 객체**를 쉽게 주입할 수 있습니다.
- 반면, 필드 주입은 객체에 직접 접근해야 하므로 테스트 시 불편하며, 리플렉션(reflection)을 사용해야 하는 경우도 있습니다.

### 3. **의존성 명시성**

- **생성자 주입**은 객체가 생성될 때 반드시 필요한 의존성을 명확히 드러냅니다.
- 반면, **필드 주입**은 클래스 내부에서 의존성이 설정되므로 어떤 의존성이 필요한지 외부에서 바로 파악하기 어렵습니다.

### 4. **순환 의존성 방지**

- **순환 의존성(Circular Dependency)**: A 클래스가 B 클래스를 의존하고, B 클래스가 다시 A 클래스를 의존하는 구조.
- 생성자 주입에서는 **순환 의존성 문제**가 애플리케이션 실행 시점에 명확히 드러납니다.
- 필드 주입은 순환 의존성을 감지하기 어려워 런타임 중에 예기치 않은 문제를 일으킬 수 있습니다.

### 5. **Spring 프레임워크 의존성 줄이기**

- 필드 주입은 스프링 프레임워크에 강하게 의존합니다.필드에 직접 `@Autowired`를 붙여야 하기 때문입니다.
- 생성자 주입은 순수 자바 코드로도 의존성을 주입할 수 있으므로 스프링 프레임워크와의 결합도를 낮출 수 있습니다.

### 6. **객체 완전성 보장**

- 생성자 주입을 사용하면 객체가 생성되는 시점에 모든 의존성이 주입됩니다.
- 필드 주입에서는 주입 시점이 불분명할 수 있으며, 특히 **빈 초기화 순서**에 따라 문제가 발생할 가능성이 있습니다.

## 롬복

`@RequiredArgsConstructor` :Lombok이 제공하는 어노테이션으로, 클래스의 `final` 필드나 `@NonNull`이 붙은 필드를 자동으로 생성자의 매개변수로 설정하고 생성자 코드를 생성해주는 역할을 함

```java
@RequiredArgsConstructor
public class MemberService {
    private final MemberRepository memberRepository;
}
```

위 코드는 Lombok에 의해 생성자가 자동으로 추가됨

테스트코드

```java
@Transactional
public class MemberServiceTest {
		@Autowired MemberService memberService;
    @Autowired MemberRepository memberRepository;
		@Test
		public void 회원가입() throws Exception {
				//Given
        Member member = new Member();
        member.setName("kim");
				//When
        Long saveId = memberService.join(member);
				//Then
        assertEquals(member, memberRepository.findOne(saveId));
	  }
		@Test(expected = IllegalStateException.class) 
		public void 중복_회원_예외() throws Exception {
				//Given
        Member member1 = new Member();
        member1.setName("kim");
        Member member2 = new Member();
        member2.setName("kim");
				//When
				memberService.join(member1); 
				memberService.join(member2); //예외가 발생해야 한다.
				//Then
				fail("예외가 발생해야 한다."); 
		}
}
```

**기술 설명**

- `@RunWith(SpringRunner.class)` : 스프링과 테스트 통합
- `@SpringBootTest` : 스프링 부트 띄우고 테스트(이게 없으면 `@Autowired` 다 실패)
- `@Transactional` : 반복 가능한 테스트 지원, 각각의 테스트를 실행할 때마다 트랜잭션을 시작하고 테스트가 끝나면 트랜잭션을 강제로 롤백 (이 어노테이션이 테스트 케이스에서 사용될 때만 롤백)

Given, When, Then에 대해서는 다음에 공부..ㅎ

## 섹션6.

**상품 엔티티 코드**

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "dtype")
@Getter @Setter
public abstract class Item {
		@Id @GeneratedValue
    @Column(name = "item_id")
    private Long id;
    private String name;
    private int price;
    private int stockQuantity;
    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<Category>();
     
		public void addStock(int quantity) {
				this.stockQuantity += quantity;
    }
    public void removeStock(int quantity) {
        int restStock = this.stockQuantity - quantity;
        if (restStock < 0) {
            throw new NotEnoughStockException("need more stock");         
            }
        this.stockQuantity = restStock;
    }
}
```

- `addStock()` 메서드는 파라미터로 넘어온 수만큼 재고를 늘린다. 이 메서드는 재고가 증가하거나 상품 주문을 취소해서 재고를 다시 늘려야 할 때 사용한다.
- `removeStock()` 메서드는 파라미터로 넘어온 수만큼 재고를 줄인다. 만약 재고가 부족하면 예외가 발생한다. 주로 상품을 주문할 때 사용한다.

### `@ManyToMany` 란?

JPA에서 두 엔티티 간의 다대다 관계를 매핑할 때 사용하는 어노테이션

다대다 관계: 하나의 엔티티가 여러 개의 다른 엔티티와 관계를 맺고, 그 반대도 가능한 관계

<aside>
💡

객체 지향언어에서는 2개의 컬렉션 객체로 다대다 관계를 표현 할 수 있지만, 관계형 데이터베이스는 정규화된 2개의 테이블로 다대다 관계를 표현할 수 없어 중간 테이블을 생성해야 한다.

</aside>

### 다대다 맵핑 방법 1- `@ManyToMany` 어노테이션 활용

**장점**

- `@ManyToMany`를 사용하면 연결 테이블을 자동으로 처리해주므로 도메인 모델이 단순해지고 편리하다.

**단점**

- JPA에서 다대다 관계를 설정하면 내부적으로 중간 테이블이 존재하기때문에 예상치 못한 쿼리가 발생한다.
- 연결 테이블에 컬럼을 추가할 수 없다.

### 다대다 맵핑 방법 2- 중간 테이블을 **Entity**로 만들어 일대다 - 다대일 관계를 매핑해주는 방식(더욱 선호되는 방식)

**상품 레포지토리 코드**

```java
@Repository
@RequiredArgsConstructor
public class ItemRepository {
		private final EntityManager em;
    public void save(Item item) {
		    if (item.getId() == null) {
		        em.persist(item);
        } else {
            em.merge(item);
        }
    }
    public Item findOne(Long id) {  
        return em.find(Item.class, id);
    }
    public List<Item> findAll() {
        return em.createQuery("select i from Item i",Item.class).getResultList();
    }
}
```

**상품 서비스 코드**
```java
@Service
@Transactional(readOnly = true)
@RequiredArgsConstructor
public class ItemService {
		private final ItemRepository itemRepository;
    @Transactional
    public void saveItem(Item item) {
		    itemRepository.save(item);
    }
    public List<Item> findItems() {
        return itemRepository.findAll();
    }
    public Item findOne(Long itemId) {
	      return itemRepository.findOne(itemId);
		} 
}
```

