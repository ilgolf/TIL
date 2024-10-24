# 가변성을 제한하는 방법과 Kotlin에서의 Collection

이펙티브 코틀린 아이템 1에선 가변성은 제한하는 것이 권장된다. 이유는 다음과 같다.

```
가변성은 코드를 읽을 떄 설명이 필요하고 동시성 프로그래밍에서 적절한 동기화를 해주지 않으면 자칫하면 예상한 결과와 다른 결과가 나올 수 있다.
```

그럼 어떻게 가변성을 제한할 수 있을까? 

## defensive copy

대표적인 예시론 `defensive copy`가 존재한다. 이는 다음과 같이 쑬 수 있다. 

```kotlin
class OrderMutator(
    override val orderId: Long,
    override val name: String,
    override val amount: BigDecimal,
    override val stock: Int,
    override val orderDate: Instant,
    override val orderState: OrderState,
    override val payment: Payment,
    override val orderItem: List<OrderItem>,
): Order {

    override fun changeState(state: OrderState): Order {
        return OrderMutator(
            orderId = this.orderId,
            name = this.name,
            amount = this.amount,
            stock = this.stock,
            orderDate = this.orderDate,
            orderState = state,
            payment = this.payment,
            orderItem = this.orderItem,
        )
    }
}
```
위 코드는 val로 외부에서 변경될 수 없게 가변성을 제한하고 내부적으론 copy를 통해 상태를 변경하고 있다. 다음과 같이 코드를 작성하면 상태 변화를 log를 통해 직관적으로 관찰이 가능하고 
Thread safe하기 때문에 안전하게 상태를 변경할 수 있다. 또한 상태 추적이 가능하기 때문에 비즈니스 로직을 보더라도 이해하기 쉬운 코드가 작성이 가능해진다. 

## Immutable Collection

kotlin에서는 Immutable Collection을 제공한다. 다음과 같이 선언이 가능하다.

```kotlin
val immutableList = listOf()         // 가변 X
val mutableList = mutableListOf()    // 가변 O
```

기본적으로 kotlin에서 제공해주는 list는 immutable이다. 그렇기 떄문에 add 같은 내부 상태를 바꾸는 함수를 제공해주지 않는다. 반면에 상태를 바꾸고 싶다면 mutable list를 사용하는 것이 좋다. 

## 개인적인 생각

Immutable Collection은 굉장히 유용하다. 상태 변화로부터 우리 로직을 안전하게 보호할 수 있기 때문이다. 하지만 defensive copy는 로직이 늘어나고 가변성이 제한되기 때문에 JPA를 사용할 경우 더티체킹같은
이로운 기능들을 사용할 순 없다. 직관적으로 변경된 copy 된 project를 save 해줘야한다.
