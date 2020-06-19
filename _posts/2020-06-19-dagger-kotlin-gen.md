---
title: Dagger with Kotlin generic
author: 강성우
layout: post
category: [Dagger, Kotlin]
tag: [dagger, kotlin, android]
---

이 문서에서는 코틀린과 Dagger를 같이 사용하면서 발생했던 generic 관련 이슈를 정리 해 보았다. 

### 1. 상황 예제

```kotlin
@Module
class SomeModule {
    @Provides
    @IntoSet
    fun provideReducer(): Reducer<State> {
        return SomeReducer()
    }
}

class SomeFragment @Inject constructor(
    val reducer: Set<Reducer<*>>
}
```  

위 예제를 보면 `SomeModule`모듈 클래스에서 `SomeReducer`의 인스턴스를 `Reducer<State>`타입 으로 Set에 주입해줄것을 어노테이션으로 명시 했다. 코틀린에서 컴파일러는 문제가 없다 하지만 APT에서는 아래와 같은 오류가 발생한다. 

```
error: [Dagger/MissingBinding] java.util.Set<? extends Reducer<?>> cannot be provided without an @Provides-annotated method.
```

오류 내용은 `SomeFragment`의 생성자에 주입될 `Set<? extends Reducer<?>>`인스턴스를 제공 하는 provider를 찾지 못해 인스턴스를 가져 올 수 없다는 것 이다. 

오류 내용에서 볼 수 있듯이 `SomeFragment`의 생성자에 서 주입될 인스턴스의 제네릭 타입들은 `?`와일드 카드로 대체 되었다. 이 것은 APT에 의해 생성된 자바 파일에서도 확인 할 수 있다. 

```java
// apt에 의해 생성된 클래스 내용 중 ...

public static SomeFragment_Factory create(Provider<Set<? extends Reducer<?>>> reducer) {
  return new SomeFragment_Factory(reducer);
}

// ...
```

Dagger는 Provider에서 반환 될 인스턴스의 타입 과 주입될 인스턴스의 타입이 정확하게 일치 해야 된다. 이는 코틀린의 기준이 아니라 Java기준이다. 그렇다면 `?` 와일드 카드를 제거 하고 정확하게 주입 받기 위해서는 `*` 가 아닌 `State`의 타입임을 명시 해줘야 한다. 

이전 `SomeFragment`를 변경 한다. 

```kotlin
class SomeFragment @Inject constructor(
    val reducer: Set<Reducer<State>>
) {
```

동작시켜도 결과는 동일하다. 하지만 오류메시지가 조금 다르다. 

```
error: [Dagger/MissingBinding] java.util.Set<? extends Reducer<State>> cannot be provided without an @Provides-annotated method.
```

여전히 `Set<? extends Reducer<State>>`을 찾지 못하고 있다. 아직 `?`와일드 카드가 남아있으니 이를 바꿔야 하는데 잘 보면 `? extends Reducer` 라고 되어 있다. 

코틀린에서는 `@JvmSuppressWildcards`라는 어노테이션을 제공 한다. 이 어노테이션은 코틀린에서 자바 코드로 변환될때의 `? extends T`를 하지 않고 어노테이션 뒤에 명시한 타입으로 적용하게 해준다. 

```kotlin
@Module
class SomeModule {
    @Provides
    @IntoSet
    fun provideReducer(): Reducer<@JvmSuppressWildcards State> {
        return SomeReducer()
    }
}

class SomeFragment @Inject constructor(
    val reducer: @JvmSuppressWildcards Set<Reducer<State>>
) {
// ...
}
```

`@JvmSuppressWildcards` 어노테이션을 provider의 리턴타입 제네릭 타입과 주입 받는 클래스의 생성자에 Set제네릭 타입에 적용 한다. 

이렇게 하면 모듈 클래스의 `provideReducer()`함수는 `Reducer<State>`을 반환하며, 주입 받는 `SomeFragment`의 `reducer`패러미터의 타입은 `Set<Reducer<State>>`가 되어 정상적으로 주입이 잘 되는 것 을 확인 할 수 있다. 

이것으로 보면 문서 작성 시점에서의 Dagger를 사용 할때 코틀린 제네릭 타입을 사용하게 된다면 `@JvmSuppressWildcards`나 `@JvmWildcards`등의 어노테이션을 자주 써야 할 것이다. 

### 2. 결말

결국 코틀린의 `<*>`는 Dagger에서 사용 할 수 없다. 위의 경우 처럼 인터페이스혹은 클래스로만 타입 캐스팅이 되어 주입되기 때문에 유연한 코드를 작성할 수 없어 결국 일일히 주입될 대상을 `@Provide, @Binds`될 메소드의 패러미터에 일일히 하나씩 넣어주어야 한다. 몰론 패러미터로 주입될 대상도 provide되어야 한다.. 아니면 주입 대상 함수 내 에서 new 하든지. 

Dagger는 러닝커브도 높아져만 가고 제대로 이해하기 전 까지는 제대로 사용하는 길은 너무나도 멀어보인다. 당연히 파고 들어가 삽질좀 하면서 하나 하나 이해할때까지 사용해 봐야 할 거 같다. 

