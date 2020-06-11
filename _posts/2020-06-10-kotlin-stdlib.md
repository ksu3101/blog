---
title: 코틀린 표준 라이브러리 함수
author: 강성우
layout: post
---

코틀린의 표준화 된 함수들중 일부를 정리 하였다. 몰론 개발자에 따라서 이 함수를 꼭 사용할 필요는 없다고 생각 한다. 왜냐하면 코틀린에서는 다양한 확장함수 를 직접 만들 수 있고 개발중인 도메인, 피쳐 등에 따라 성격과 네이밍이 바뀔 수 있으므로 그때마다 확장함수를 만들어서 사용해도 무방하다고 생각 한다. (몰론 기술적인 측면에서 표준 라이브러리 함수들과 성능상의 차이 등이 없어야 할 것이다)

표준 라이브러리 자체는 여러 개발자가 한 프로젝트를 개발할 때 유용하다고 할 수 있겠다. 각 필요한 확장함수를 코드리뷰및 검토 후 추가하는 방법도 좋지만 더 많은 사람들에게 보여주는 경우(오픈소스 와 같은) 더 빠르게 이해할 것 이다. 몰론 이는 이 함수들을 알고 있다는 것을 전제로 한다. 

### apply, also, let, run, with

![stdlib](/blog/assets/images/kotlin_stdlib1.png)

- apply, also : 객체(리시버)의 프로퍼티를 변경 하고 그 객체를 반환 할때.
- let, run : 객체(리시버) 자체 혹은 프로퍼티들을 이용해서 무엇인가를 하고 다른 객체를 반환 할 때.
- with : 결과 반환이 필요하지 않은 객체를 받아 무엇인가를 할 때.

```kotlin
// T is Receiver, block is lambda
inline fun <T> T.apply(block: T.() -> Unit): T { 
  block()
  return this 
}

inline fun <T> T.also(block: (T) -> Unit): T { 
  block(this)
  return this 
}

inline fun <T, R> T.let(block: (T) -> R): R { 
  return block(this) 
}

inline fun <T, R> T.run(block: T.() -> R): R { 
  return block() 
}

inline fun <T, R> with(receiver: T, block: T.() -> R): R { 
  return receiver.block() 
}
```
  
하단 예제 코드를 참고 하도록 하자. 
- `A.apply { this.b = "b" }.receiverACodes()...`
    ```kotlin    
    // 리시버의 프로퍼티만 변경할 수 있음 (but, it's variables only)
    val product = Product().apply {
        this.name = "Park"
        this.price = 100        
    }
    ```

- `A.also { a -> a.b = "b" }.receiverACodes()...`
    ```kotlin
    val product2 = Product().also() { product -> 
        product.name = "Lee"
        product.price = 125
    }
    ```

- `A?.let { a -> "b"}.returningBCodes()... `
    ```kotlin
    val result = product2.let { product ->
          product.name
    }
    println(result)
    ```

- `A.run { this.b = "b" ; this.b }.returningBCodes()...`
    ```kotlin
      val newPrice = product2.run { 
        this.price += 300
        this.price
    }
    println(newPrice)
    ```

- `val result = with(a) { this.b = "b"; this.b }`
    ```kotlin
    val newPrice2 = with(product2) {
        this.price += 125
        this.price
    }
    println(newPrice2)
    ```
  
### synchronized

```kotlin
inline fun <R> synchronized(lock: Any, block: () -> R): R
```

`lock` 객체를 mutex(스레드 락을 잡기 위한 대상 객체), 뮤텍스를 얻었을 경우 동기화 블럭인 `block` 을 실행 하게 한다. 
