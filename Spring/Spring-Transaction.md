# Spring Transaction

Transaction은 보통 `@Transactional` annotation으로 달성할 수 있다. 이를 사용하면 특정 메소드 또는 클래스에 트랜잭션 범위를 정할 수 있다.
Transaction은 어노테이션을 통해 전파할 수 있다. 

```kotlin
@Transactionl
fun create(user: User) {
    val user = userRepository.save(user)

    this.addUserCount(user.gender)
}

private addUserCount(gender: Gender) {
    val totalUserInfo = totalUserRepository.findByUserGender(gender)
    totalUserInfo.updateOne() // 실패시 rollback 됩니다. 
}
```

트랜잭션 선파 옵션은 다음과 같다. 

1. REQUIRED: 이미 트랜잭션이 존재하면 이를 사용하고 없으면 새로운 트랜잭션 시작
2. REQUIRED_NEW: 항상 새로운 트랜잭션을 시작
3. MANDATORY: 반드시 기존 트랜잭션 내에서 실행되어야 하며, 트랜잭션이 없을 시 예외 발생
4. SUPPORTS: 트랜잭션이 있으면 이를 사용하고 없으면 트랜잭션 없이 실행

또한 격리 수준도 줄 수 있다. 

- READ_UNCOMMITTED
- READ_COMMITTED
- REPEATABLE_READ
- SERIALIZABLE

## 트랜잭션 내부를 파보자 

### TransactionAspectSupport 

`@Transactional`은 AOP로 동작하기 때문에 Aspect method를 분석해야 동작 원리를 알 수 있는데 Aspect method를 call하는 부분이 바로 TransactionAspectSupport 클래스다. 

트랜잭션을 시작 하는 부분 코드부터 시작해보자

```java
	protected TransactionInfo createTransactionIfNecessary(@Nullable PlatformTransactionManager tm,
			@Nullable TransactionAttribute txAttr, final String joinpointIdentification) {

		// If no name specified, apply method identification as transaction name.
		if (txAttr != null && txAttr.getName() == null) {
			txAttr = new DelegatingTransactionAttribute(txAttr) {
				@Override
				public String getName() {
					return joinpointIdentification;
				}
			};
		}

		TransactionStatus status = null;
		if (txAttr != null) {
			if (tm != null) {
				status = tm.getTransaction(txAttr);
			}
			else {
				if (logger.isDebugEnabled()) {
					logger.debug("Skipping transactional joinpoint [" + joinpointIdentification +
							"] because no transaction manager has been configured");
				}
			}
		}
		return prepareTransactionInfo(tm, txAttr, joinpointIdentification, status);
	}

	protected TransactionInfo prepareTransactionInfo(@Nullable PlatformTransactionManager tm,
			@Nullable TransactionAttribute txAttr, String joinpointIdentification,
			@Nullable TransactionStatus status) {

		TransactionInfo txInfo = new TransactionInfo(tm, txAttr, joinpointIdentification);
		if (txAttr != null) {
			// We need a transaction for this method...
			if (logger.isTraceEnabled()) {
				logger.trace("Getting transaction for [" + txInfo.getJoinpointIdentification() + "]");
			}
			// The transaction manager will flag an error if an incompatible tx already exists.
			txInfo.newTransactionStatus(status);
		}
		else {
			// The TransactionInfo.hasTransaction() method will return false. We created it only
			// to preserve the integrity of the ThreadLocal stack maintained in this class.
			if (logger.isTraceEnabled()) {
				logger.trace("No need to create transaction for [" + joinpointIdentification +
						"]: This method is not transactional.");
			}
		}

		// We always bind the TransactionInfo to the thread, even if we didn't create
		// a new transaction here. This guarantees that the TransactionInfo stack
		// will be managed correctly even if no transaction was created by this aspect.
		txInfo.bindToThread();
		return txInfo;
	}
```

흐름은 다음과 같다. 

1. 트랜잭션 이름 설정 트랜잭션 이름이 지정되지 않으면 메소드 식별자를 트랜잭션 이름으로 정해 식별가능한 속성을 정의합니다.
2. 속성이 존재하고 트랜잭션 매니저가 존재하면 매니저를 통해 트랜잭션 상태를 가져옵니다.
3. 트랜잭션 정보를 담고 있는 객체를 생성하고 이 객체엔 트랜잭션 관리자, 트랜잭션 속성, 메소드 식별자를 포함합니다.
4. 트랜잭션 상태에 따라 트랜잭션이 필요한지 아닌지 여부를 결정합니다.
5. 트랜잭션 정보 스택이 올바르게 관리되도록 보장하기 위해 트랜잭션 정보를 현재 스레드에 바인딩합니다.

결론적으로 Spring Transaction 관리 매커니즘의 core 부분을 보았을 때 식별 가능한 트랜잭션 속성을 정의하고 정보를 생성해서 스레드에 바인딩 함으로써
트랜잭션이 일관성 있게 처리되게 도움을 줍니다. 

### Transaction rollback code

```java

```