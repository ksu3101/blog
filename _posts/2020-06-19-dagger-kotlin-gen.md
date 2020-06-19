---
title: Dagger with Kotlin generic
author: 강성우
layout: post
category: [Dagger, Kotlin]
tag: [dagger, kotlin, android]
---

Dagger는 좋은 DI 도구 이지만 Kotlin과 함께 사용 하기에는 아직까진 문제가 조금 있다. 예를 들어 자바의 Generic과 코틀린의 Generic의 사용에 있어서 서로 다른점이 있기 때문이다. 

이 문서는 Dagger와 Kotlin의 Generic을 적용 하면서 고통받은 내용을 정리 해 보았다. 

### 1. 예제 

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
// ...
}
```

위 예제를 보면 `SomeModule`모듈 클래스에서 `SomeReducer`의 인스턴스를 `Reducer<State>`타입 으로 Set에 주입해줄것을 어노테이션으로 명시 했다. 컴파일 시점에 문제가 없지만 이 를 돌려보면 아래와 같은 오류가 발생한다. 

```
error: [Dagger/MissingBinding] java.util.Set<? extends Reducer<?>> cannot be provided without an @Provides-annotated method.
```

오류 내용은 `SomeFragment`의 생성자에 주입될 `Set<? extends Reducer<?>>`인스턴스를 제공 하는 provider를 찾지 못해 인스턴스를 가져 올 수 없다는 것 이다. 

오류 내용에서 볼 수 있듯이 `SomeFragment`의 생성자에 서 주입될 인스턴스의 타입들은 `?`와일드 카드로 대체 되었다. 그렇다면 어노테이션 프로세서와 컴파일러에 의해 생성된 자바 코드는 어떨까? 

```java
// @Inject 어노테이션으로 생성된 클래스 내용  ...
public static SomeFragment_Factory create(Provider<Set<? extends Reducer<?>>> reducer) {
  return new SomeFragment_Factory(reducer);
}
```

Dagger에서 중요한 점은 Provider에서 반환하는 인스턴스의 타입 과 주입될 인스턴스의 타입이 정확하게 일치 해야 주입이 된다. 이는 코틀린의 기준이 아니라 Java의 기준이다. 그렇다면 `?` 와일드 카드를 제거 하고 정확하게 주입 받기 위해서는 `*` 가 아닌 `State`의 타입임을 명시 해줘야 한다. 

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

`@JvmSuppressWildcards` 어노테이션을 provider의 리턴타입 제네릭 타입과 주입 받는 클래스의 생성자에 Set 의 제네릭 타입에 적용 한다. 

이렇게 하면 모듈 클래스의 `provideReducer()`함수는 `Reducer<State>`을 반환하며, 주입 받는 `SomeFragment`의 `reducer`패러미터의 타입은 `Set<Reducer<State>>`가 되어 정상적으로 주입이 잘 되는 것 을 확인 할 수 있다. 

이것으로 보면 코틀린에서 Dagger를 쓸때 제네릭을 자주 쓴다면 위와 같은 `@JvmSuppressWildcards`나 `@JvmWildcards`등의 어노테이션을 자주 쓰게 될 것이다. 

### 2. 결말

솔직히 토이프로젝트 건드리면서 Koin의 유혹을 뿌리치는게 너무나도 힘들었다. Dagger를 잘 쓴다면 좋긴 하지만 태생적으로 컴파일 시점에 Java코드로 변환되는 점 때문에 코틀린으로 코드를 작성함에도 자바를 생각 하면서 해야 한다. 솔직히 말하면 이 점 때문에 Dagger의 도입이 조금 더 생각하게 된다.

만약 Dagger를 프로젝트에 도입 하고 싶다면, 

- Dagger에 대한 모든 API를 이해하고 내부 프로세스를 알고 있어야 정말 제대로 사용 할 수 있다. 
- Dagger에 대한 이해도가 낮다면 코드는 순식간에 엉망이 되고 어쩔수 없이 보일러 플레이트 코드를 작성해야 한다. 이는 보일러 플레이트를 복사-붙여넣기 하는 일 들이 발생 할 테고 추 후 유지, 보수에 악영향을 미칠 것 이다. 
- 이해도가 낮은 경우 작성한 코드가 동작 하더라도 Dagger에서 의도한 바 대로 동작하는지 개발자는 이해하지 못한상태로 넘어가게 된다.
- Kotlin의 장점중 일부들을 어쩔수 없이 포기해야 한다. 

이 것들을 감안해야 할거 같다. 
