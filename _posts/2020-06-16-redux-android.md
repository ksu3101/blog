---
title: Redux basic for Android developers
author: 강성우
layout: post
category: [Architecture, Redux]
tag: [recommended, android, redux, architecture]
image: /blog/assets/images/redux-banner.jpg
---

Redux 기반의 안드로이드 아키텍쳐를 이해하기 위해서 Redux 에 대한 기초를 안드로이드 개발자 입장에서 정리해 보았다. 

### 1. Redux

Redux는 자바스크립트 환경에서 사용 하는 글로벌 상태 관리 도구이다.

웹, 모바일 모든 어플리케이션에서는 비즈니스 로직과 뷰의 관계를 갖는데, 이 때 뷰의 업데이트를 위해서는 뷰에 대한 상태들을 관리할 필요성이 있다. 하지만 기존에는 상태를 비즈니스 로직의 콜백에서 접근 하여 화면을 그리고 업데이트 하는 환경이 대부분 이었다. 결국, 콜백이 많아질 수록 뷰에 대한 업데이트 코드는 늘어날 것 이고 화면의 명확한 상태를 보장하기 어려워 진다. 

Redux는 이런 문제를 해결하기 위해 화면의 상태를 마지막 화면의 상태를 정의한 `State`로서 단방향 데이터 흐름(Uni-Directional data Flow)을 기반으로 단순하게 처리 할 수 있도록 도와준다. 단방향 데이터 흐름의 다른 아키텍쳐로는 MVI(Model-View-Intent)가 있으며 이는 Flux, Redux의 영향을 깊게 받은것 으로 알려져 있다. 

### 2. Redux basic 

![redux_basic](/blog/assets/images/redux_basic_1.png)

#### 2.1 Action

```kotlin
// marker interface 
interface Action

// domain Action
sealed class MessageAction: Action
data class ToastMessageAction(val message: String): MessageAction()

// Action dispatch by store(AppStore)
class SomeViewModel: ViewModel() {
    val store: AppStore by injec()
    fun foo() {
        store.dispatch(ToastMessageAction("메시지 내용"))
    }
}

```

`Action`은 View(사용자로부터) 혹은 다른 Action 에 의해 발생한 Result, Success, Failed, Error 등의 Action 들을 정의 한 `immutable data class`이다. 

`Action`은 `State`를 변하게 할 수 있는 유일한 Trigger 라고 할 수 있다. Action 은 후술할 `Store`를 통해 `dispatch()` 되어 비즈니스 로직을 태우거나 새로운 Action을 생성 할 수도 있다. 

- Action 의 종류
  - __Action__ : 기본적인 domain하 에서 정의된 Action 이다. 보통 use-case에서 사용자 등에 의해 발생한 View event 에서 발생한다.
  - __Result Action__ : Action 을 후술할 `Middleware`에서 핸들링 하고 난 뒤 발생한 Action이다. Middleware 에서 원격 혹은 로컬 저장소를 거쳐 나온 result를 새로운 Action 데이터 클래스의 인스턴스로 만든다. 
  - __Error Action__ : 런타임 도 중 발생한 예외들에 대해 만들어진 Action 이다. 일반적으로 화면에 보여지게 될 오류 메시지와 예외에 대한 정보를 갖는다. 

#### 2.2 State

```kotlin
// marker interface
interface State

// domain State
sealed class MessageState: State
data class ToastMessageState(val message: String): MessageState()
```

`State`는 후술할 `Reducer`를 통해 만들어진 `immutable data class`로서 화면의 상태를 정의 한다. 

`State`는 `Store`에 마지막 인스턴스혹은 Initialized State만 저장 된다. 그리고 Store 의 State는 오직 dispatch된 `Action`으로만 변경 될 수 있다. 

##### 2.2.1 at Redux base Android architecture

Redux 기반 안드로이드 아키텍쳐 에서 `Store`에 `BehaviorSubject`로 퍼블리싱 되는 `Observable`로 래핑된 `State`인스턴스는 `ViewModel`의 `render()` 함수를 통해서 핸들링 된다. 

```kotlin
// rendering of State
class SomeViewModel: ViewModel() {
    val messageHelper: MessageHelper by inject()
    fun render(state: State): Boolean {
        return when (state) {
            is ToastMessageState -> {
                messageHelper.showToast(state.message)
            }
            else -> false
        }
    }
}
```

`render()`함수 에서는 반환 타입으로 `Boolean`타입을 갖는데 이는 퍼블리싱 된 `State`의 컨슘 여부이다. `render()`에서 반환될 Boolean 값이 `true` 일 경우 외부에서는 이 State가 ViewModel 에서 핸들링 되었으며 다시 핸들링 되지 않아야 한다. `false` 일 경우 ViewModel에서의 핸들링 여부에 상관없이 외부에선느 이 State를 참조 하여 사용 할 수 있다. 

참고로 Rx에서 `BehaviorSubject`는 스트림에 대해 subscribe최초 시점에 마지막으로 생성한 `Observable`을 다시 스트림으로 내보내 준다. 

#### 2.3 Store

```kotlin
interface Store<S : State> {
    fun dispatch(action: Action)
    fun getStateListener(): Observable<S>
    fun getCurrentState(): S
}
```

`Store`는 어플리케이션 생명주기에 단 하나의 인스턴스만 존재해야 하는 클래스이다. `Store`가 하는 일은 아래와 같다. 

- 마지막 혹은 Initialize `State`를 저장 한다. 
- `State`를 변경 하기 위해서 `dispatch(Action)`함수를 통해 `Action`을 전달한다.
- `State`의 변화를 감지 하고 변화된 `State`를 구독 하기 위해서 `getStateListener()`를 통해 콜백을 등록 한다. 
- 마지막 혹은 Initialize 상태를 얻기 위해서 `getCurrentState()`함수를 이용 할 수 있다. 



##### 2.3.1 AppStore

아래 `AppStore`예제는 실제로 작성자가 사용했었던 구조이다. Store에서 여러개의 State를 쉽게 관리 하기 위해서 `AppState`라는 글로벌한 상태를 두고 내부에 `Map`으로 각 하위 도메인 State들을 저장하게 하는 방법을 사용 하였다. 몰론 이 방법은 정석이 아니며 더 좋은 방법이 있으면 해당 방법을 고민하는게 좋을것 같다. 이 예제는 예상대로 구동되는 한가지의 __예제__ 이므로 참고만 하고 더 나은 구조를 고민 하는것을 추천 한다. 

안드로이드 에서는 Application내 도메인이 여러개가 만들어 질 수 있다. Redux에서는 단 한개의 `Store`만 존재 해야 하므로 `Store` 인터페이스를 구현한 `AppStore`라는 클래스를 만들어 사용 하게 된다. 

```kotlin
// application state 
class AppState(
    val states: Map<String, State>
): State { 
    // ...
}

// Action dispatcher function
typealias Dispatcher = (Action) -> Unit

// application store
class AppStore(
    val reducer: Reducer<AppState>,
    initliazedState: AppState
): Store<AppState> {
    private val stateEmitter: BehaviorSubject<AppState> = BehaviorSubject.create()
    private val middleWares: Array<MiddleWare<AppState>> // Koin을 이용해 주입받거나 다른 방법을 통해 초기화 
    private var state: AppState = initliazedState
    private var dispatcher: Dispatcher

    init {
        dispatcher = middleWares.foldRight(
            { dispatchedAction: Action ->
                appState = reducer.reduce(appState, dispatchedAction)
                stateEmitter.onNext(appState)
            }
        ) { middleWare, next ->
            middleWare.create(this, next)
        }
    }

    override fun getStateListener(): Observable<AppState> = 
        stateEmitter.hide().observeOn(AndroidSchedulers.mainThread())

    override fun getCurrentState(): AppState = state

    override fun dispatch(action: Action) {
        dispatcher(action)
    }
}
```

안드로이드 에서는 도메인 `State`들 을 `Map`콜렉션으로 갖는 `AppState`를 정의 하고 Map 에서는 State의 클래스 를 key, value로 State자체를 저장 한다. `Store`의 인터페이스에서 제공하는 API의 구현을 확인 하도록 하자. 

`Dispatcher`는 dispatch된 `Action`을 어떻게 후술할 `Middleware`를 이터레이셔닝 하는지 보여준다. `foldRight()`함수는 이터레이셔닝 가능한 콜렉션을 대상으로 마지막 index 로부터 0번째 index까지 역으로 이터레이셔닝 하면서 적용한 람다를 통해 반환된 객체를 다음 람다에 전달 해 주는 함수 이다. 여기서 전달되는 객체는 `Dispatcher`함수의 인스턴스이다. 

`dispatch()` 후 Action이 어떻게 이터레이셔닝 되는지 예를 보면 아래와 같다. 

1. `dispatch()` 함수를 통해 어떤 `Action` 이 `Dispatcher`함수에 전달 된다.
2. `Dispatcher` 함수에서는 `(Action) -> Unit` 의 람다 형태 인데, `dispatch()` 된 Action을 람다의 패러미터로 받는다. 이 패러미터를 이용하여 `AppStore`의 `Reducer`를 통해 핸들링 하고 새로운 혹은 이전의 `AppState`를 만든다. 
3. `Dispatcher`를 통해서 생성된 `AppState`를 `BehaviorSubject<AppState>`를 통해 `Observable<AppState>`스트림 으로 발행 한다. 
4. `Dispatcher`를 `Array<MiddleWare<AppState>>`을 이터레이셔닝 하면서 핸들링 한다. 핸들링 이란 위 (1)번부터 (3)번의 행동을 반복한다고 생각 하면 된다.

#### 2.4 Middleware

`Middleware`는 dispatch된 `Action`을 핸들링 하여 새로운 `Action`을 만들거나 그대로 반환한다. 

`Middleware`는 여러개가 존재 할 수 있으며 이를 컬렉션에 저장 하여 `Store`에서 이터레이션 한다. 미들웨어는 dispatch된 `Action`을 핸들링 하는데, 예를 들면 비즈니스로직이나 network API, local DAO 등 비동기 작업들이 주 대상이다. 

비동기 작업을 핸들링 하는 경우 이를 `ActionProcessor`라고 하는데 이를 미들웨어로 만들면 `ActionProcessorMiddleware`가 된다. 이 미들웨어에서는 비즈니스로직만 수행하며 이는 테스트 코드 작성하는데도 다른 코드들을 신경 안써도 되는 장점을 갖고 있다. 

미들웨어 에서 핸들링 되어 나오는 `Action`은 새로운 Action 혹은 이전 Action 그대로 반환하는 경우도 있다. 일반적으로 `Action`은 순차적으로 발행되진 않는다. 하드웨어의 자원 가용 여부 혹은 OS의 우선순위 등 여러가지 조건에 따라서 dispatch된 Action은 순차적으로 수행되지 않을 수도 있다는 것 이다. 

```kotlin
interface MiddleWare<S: State> {
    fun create(store: Store<S>, next: Dispatcher): Dispatcher
}
```

아래는 `Middleware`의 간단한 예제로서 dispatch 된 `Action`과 이 dispatch시점 에서의 마지막 `State`를 로깅 하는 미들웨어 클래스 이다.

```kotlin
class LoggerMiddleware<S : State> : Middleware<S> {
    override fun create(store: Store<S>, next: Dispatcher): Dispatcher {
        return { action: Action ->
            if (BuildConfig.DEBUG) {
                Log.d(LOG_TAG, "action dispatch : [${action.getSuperClassNames()}] $action")
            }
            val prevState = store.getCurrentState()
            next(action)

            if (BuildConfig.DEBUG) {
                val currentState = store.getCurrentState()
                if (prevState != currentState) {
                    (currentState as AppState).printStateLogs()
                }
            }
        }
    }

}
```

#### 2.5 Reducer

```kotlin
interface Reducer<S : State> {
    val initializeState: S

    fun reduce(oldState: S, resultAction: Action): S
}
```

`Reducer`는 `Middleware`를 통해 전달 받은 `Action`을 이전 `State`와 함께 핸들링 하여 새로운 `State`혹은 이전 `State`를 그대로 반환한다. 

`Reducer`는 도메인에 대해 1:1 관계를 가질수 있으며 필요하다면 서브 도메인 으로 쪼개서 여러개를 가질 수 있다. 

### 3. 결말

View의 상태를 관리 하기 위한 도구로서 Redux의 단일 방향 구조는 훌륭하다고 생각 된다. 단일 데이터 흐름으로 인해 갖는 장점은 유지, 보수에도 큰 장점을 준다. 단점이라면 아무래도 안드로이드 개발자가 Redux나 Flux를 알지 못하니 그에 대한 러닝커브가 존재 하는 점 이다. 그리고 Action이나 State를 생성 할 때 deep copy를 피하고 이전 객체를 최대한 재사용 해야 하는 점이 있다. 

안드로이드에서 적용 될 Redux구조는 Rx와 활용하여 MVVM 구조에서 최고의 효율을 보여준다고 생각 한다. 여기에 Koin과 같은 DI 도구를 사용하면 더 편리하게 쓸 수 있다. (Dagger와도 같이 써보려고 시도 하고 있지만 dagger.anddroid 특유의 높디 높은 러닝커브가 아직은 힘들다)
