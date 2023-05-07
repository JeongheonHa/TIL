# 스프링 데이터 JPA 분석

## 1. 스프링 데이터 JPA가 사용하는 구현체

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID> ...{

    @Transactional
    public <S extends T> S save(S entity) {
        if (entityInformation.isNew(entity)) {
            em.persist(entity);
            return entity;
        } else {
            return em.merge(entity);
        } 
    }
... 
}

```

### 1.1 @Repository 적용
- ComponentScan의 대상을 인식
- JPA 예외를 스프링이 추상화한 예외로 변환

### 1.2 @Transactional 트랜잭션 적용

- 서비스 계층에서 트랜잭션을 시작하지 않으면 리파지토리에서 트랜잭션 시작
- 서비스 계층에서 트랜잭션을 시작하면 리파지토리는 해당 트랜잭션을 전파 받아서 사용
- 그래서 스프링 데이터 JPA를 사용할 때 트랜잭션이 없어도 데이터 등록, 변경이 가능했음(사실은 트랜잭션이 리포지토리 계층에 걸려있는 것임)
- @Transactional(readOnly = true)로 스냅샷을 생성하지 않고 flush를 하지않아 변경 감지를 생략해 성능 최적화

### 1.3 save()
- 새로운 엔티티면 저장 (persist)
- 새로운 엔티티가 아니면 (merge)

#### 새로운 엔티티인지 판단하는 방법
- 기본 전략

    + 식별자가 객체이고 값이 null이면 새로운 엔티티로 판단
    + 식별자가 기본 타입이고 값이 0이면 새로운 엔티티로 판단

- `Persistable` 인터페이스를 구현해서 판단 로직 변경 가능

#### Persistable
JPA 식별자 생성 전략이 @Id만 사용해서 식별자를 직접 할당하는 경우 이미 식별자 값이 있는 상태로 save()가 호출되기 때문에 새로운 엔티티라고 판단하지않는다. 따라서, merge()를 호출하게 되고 select 쿼리를 날려 1차 캐시 및 DB에 해당 식별자에 해당하는 데이터를 찾고 없다면 새로운 엔티티를 생성해 1차 캐시에 저장하게 되고 비영속 상태의 엔티티의 데이터를 1차 캐시에 있는 엔티티로 옮기게된다. 이 과정에서 비영속 상태의 엔티티의 필드 값이 null이더라도 null 값이 저장되어 데이터가 잘못될 수 있다. 따라서 merge()를 사용하는 것 보다 Persistable인터페이스를 구현해 새로운 엔티티인지 판단하는 것이 좋다.

- `isNew()` 메서드를 구현하여 새로운 엔티티인지를 판단하면되는데 이 때, `@CreatedDate`를 사용하면 쉽게 판단할 수 있다.

```java
package org.springframework.data.domain;

public interface Persistable<ID> {
    ID getId();
    boolean isNew();
}
```

```java
@Entity
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Item implements Persistable<String> {

    @Id
    private String id;

    @CreatedDate
    private LocalDateTime createdDate;

    public Item(String id) {
        this.id = id;
    }

    @Override
    public String getId() {
        return id; 
    }

    @Override
    public boolean isNew() {
        return createdDate == null;
    }
}
```

## Reference
> 실전! 스프링 데이터 JPA [김영한]
