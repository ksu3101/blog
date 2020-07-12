---
title: Dagger - Hilt
author: 강성우
layout: post
category: [Android]
tag: [android, dagger, hilt]
---

dagger에서 [hilt](https://dagger.dev/hilt/)를 이용하여 안드로이드에서 더 효율높은 DI를 사용 하는 방법들에 대해 정리 하였다. 

이전에 사용하던 dagger를 안드로이드에 적용 하면서 많은 보일러플레이트 코드가 생겨났다. 높디 높은 러닝커브를 제켜두고서라도 dagger를 사용 하면서 필연적으로 발생할 수 밖에 없는 보일러 플레이트 코드를 어떻게든 제거해보려고 `dagger.android`를 사용 해 보거나 Koin을 이용 하여 dagger를 대체해보려고도 했었다. 

그러다 dagger의 hilt를 발견 하였고 일단 기존 토이 프로젝트에서 새로 브랜치를 따서 dagger hilt를 마이그레이션 해보고 어떤지 느낌을 정리 해보려고 한다. 

> 글 작성 시점에서 dagger-hilt는 `2.28-alpha`버전으로 아직 알파이다. 아래내용은 시간에 따라 변경 될 수 있다. 

## 1. build.gradle 종속 변경 

root 프로젝트의 `build.gradle`파일에 dagger-hilt의 classpath를 `dependencies`항목에 추가 한다. 

```
buildscript {	
	ext.dagger_hilt = '2.28-alpha'
	dependencies {
		classpath "com.google.dagger:hilt-android-gradle-plugin:$dagger_hilt"        
	}
}
```

그리고 dagger-hilt를 사용할 서브 모듈의 `build.gradle`파일에 dagger-hilt 플러그인 과 `dependencies`항목내에 아래와 같은 라이브러리 의존을 추가 한다. 

dagger-hilt는 java8의 기능을 사용하기 때문에 `compileOptions`를 아래처럼 추가 해 준다. 

```
apply plugin: 'dagger.hilt.android.plugin'

android {
    // ... 
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
  }
}

dependencies {
    implementation "com.google.dagger:hilt-android:$dagger_hilt"
	kapt "com.google.dagger:hilt-android-compiler:$dagger_hilt"
}
```

## 2. Application 파일 변경 

`Application`을 상속받는 프로젝트의 Application클래스에 `@HiltAndroidApp`어노테이션으로 변경 한다.
만약 `DaggerApplication`클래스를 상속받아 `applicationInjector()`메소드를 재정의해서 사용 하고 있다면 상속을 제거 하고 재정의된 메소드도 모두 제거 해야 한다. 

```kotlin
@HiltAndroidApp
class BookSearchApplication: MultiDexApplication() {
    // ... 
}
```

## 3. 주입대상 안드로이드 컴포넌트에 대한 설정 

안드로이드 컴포넌트들에 대해 `@AndroidEntryPoint`어노테이션을 추가 하여 주입대상임을 설정 한다. 

```kotlin
@AndroidEntryPoint
abstract class BookSearchActivity : AppCompatActivity() {
    @Inject lateinit var messageHelper: MessageHelper()
    // ... 
}
```

`Activity`나 `Fragment`의 경우 `HasAndroidInjector`인터페이스를 구현함으로서 android injector를 반환하게 하는데 `@AndroidEntryPoint`를 설정했을 경우 제거 해 준다. 

주입대상 안드로이드 컴포넌트에 대한 주입 인스턴스의 동기화된 라이프사이클은 [이 링크](https://developer.android.com/training/dependency-injection/hilt-android#component-lifetimes)를 참고 하도록 하자. 

> 안드로이드 컴포넌트 클래스에 `@AndroidEntryPoint`을 추가하였을 경우 그 컴포넌트(혹은 클래스)에 종속적인 클래스에도 같은 어노테이션을 추가 해야 한다. 예를 들어 Activity에 어노테이션을 추가 했을 때 이 Activity에 사용될 Fragment에도 같은 어노테이션을 추가 해야 한 다. 

> 추상클래스의 경우 `@AndroidEntryPoint`를 적용하지 않아도 `@Inject`어노테이션을 통해 주입 받을 수 있다. 

## 4. Application module

기존에 만들었던 `ApplicationComponent`와 같은 Component인터페이서는 제거 한다. 그리고 기존 Module클래스 만 남겼다. 모듈클래스는 이전과 같이 `@Module`로 선언하고 쓰면 되며 주입될 종속 컴포넌트 대상에 대한 인터페이스 지정을 `@InstallIN()`어노테이션을 통해 지정 하면 된다. 

예를 들면 Application에 종속된 모듈의 경우 `@InstallIn(ApplicationComponent::class)` 가 되며, Activity에 종속된 모듈의 경우에는 `@InstallIn(ActivityComponent::class)`을 추가 하면 자동으로 해당되는 코드들이 생성된다. 

아래 예제는 기존에 사용하던 모듈을 수정한 것 이다. 

```kotlin
@Module
@InstallIn(ApplicationComponent::class)
object ApplicationModule {
    @Singleton
    @Provides
    fun provideMiddlewares(): @JvmSuppressWildcards List<MiddleWare<AppState>> {
        return listOf(
            ActionProcessorMiddleware(
                CombinedActionProcessor(
                    listOf()
                )
            )
        )
    }

    @Singleton
    @Provides
    fun provideAppStore(
        messageReducer: MessageReducer,
        bookSearchReducer: BookSearchReducer,
        middlewares: @JvmSuppressWildcards List<MiddleWare<AppState>>
    ): AppStore {
        return AppStore(
            AppState(HandledMessageState, InitializedState),
            AppReducer(
                messageReducer,
                bookSearchReducer
            ),
            middlewares
        )
    }

    @Singleton
    @Provides
    fun provideMessageHelper(context: Context): MessageHelper {
        return MessageHelperImpl(context)
    }

    @Singleton
    @Provides
    fun provideResourceHelper(context: Context): ResourceHelper {
        return ResourceHelperImpl(context)
    }
}
```

아래는 다른 Application component에 종속된 다른 모듈의 예 이다. 

```kotlin
@Module
@InstallIn(ApplicationComponent::class)
class NetworkModule {
    private val TIMEOUT_SEC = 10L

    @Singleton
    @Provides
    fun provieOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .connectTimeout(TIMEOUT_SEC, TimeUnit.SECONDS)
            .readTimeout(TIMEOUT_SEC, TimeUnit.SECONDS)
            .addInterceptor(AddKakaoAkHeaderIntercepter())
            .addInterceptor(HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BODY))
            .build()
    }

    @Singleton
    @Provides
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .addConverterFactory(MoshiConverterFactory.create())
            .addCallAdapterFactory(RxJava3CallAdapterFactory.create())
            .baseUrl("https://...")
            .client(okHttpClient)
            .build()
    }

    @Singleton
    @Provides
    fun provideBookSearchApi(retrofit: Retrofit): BookSearchApi {
        return retrofit.create(BookSearchApi::class.java)
    }

    @Singleton
    @Provides
    fun provideBookSearchRepository(api: BookSearchApi): BookSearchRepository {
        return BookSearchRepositoryImpl(api)
    }
}

@Module
@InstallIn(ApplicationComponent::class)
class ReducerModule {

    @Singleton
    @Provides
    fun provideMessageReducer(): MessageReducer {
        return MessageReducer()
    }

    @Singleton
    @Provides
    fun provideBookSearchReducer(): BookSearchReducer {
        return BookSearchReducer()
    }

}
```

## 5. Activity module

Activity에 대한 모듈 선언 또한 비슷하지만 `@InstallIn`의 대상이 `ActivityComponent`인터페이스 인 것이 다르다. 

```kotlin
@Module
@InstallIn(ActivityComponent::class)
object BookSearchActivityModule {
    @ActivityScoped
    @Provides
    fun provideBookSearchNavigationHelper(
        @ActivityContext context: Context
    ): BookSearchNavigationHelper {
        return BookSearchNavigationHelperImpl(context as BookSearchActivity)
    }
}
```

`@ActivityScoped`어노테이션으로 provide될 모듈의 인스턴스가 Activity와 동일한 라이프사이클을 갖음을 설정했다. provide메소드 패러미터로 context가 있는데 이 context는 `@ActivityContext`을 이용 하여 Activity 컨텍스트 임을 적용 하였다. 이는 이 모듈이 `ActivityComponent`이기 때문에 가능하다. 
다만 패러미터로 받는 context는 `Context`타입으로 들어오기 때문에 타입 캐스팅 해 주어야 한다. 

## 6. Fragment module

Fragment또한 비슷하다. 대상이 `FragmentComponent`인터페이스임을 확인 할 수 있다. 

```kotlin
@Module
@InstallIn(FragmentComponent::class)
abstract class BookSearchFragmentModule {
    @Binds
    @IntoMap
    @ViewModelKey(BookSearchViewModel::class)
    abstract fun bindBookSearchViewModel(viewModel: BookSearchViewModel): ViewModel
}
```

ViewModel factory를 통해 map으로 저장될 viewmodel에 대한 선언 메소드가 기존 dagger android support과 동일하게 사용 한다. 

ViewModel factory에 대한 모듈은 아래와 같다. 

```kotlin
@Module
@InstallIn(FragmentComponent::class)
abstract class ViewModelBuilder {
    @Binds
    abstract fun bindViewModelFactory(factory: ViewModelFactory): ViewModelProvider.Factory
}
```

## 7. 마무리 

예제를 통해 마이그레이션을 진행 하면서 확실히 보일러플레이터 코드가 많이 줄어들었음을 확인 할 수 있었다. 특히 dagger android support을 사용 하면서 더 높아진 러닝커브로 인하여 디버깅 하느라 하루종일 시간을 보내면서 고생한 경험이 있어 괜찮은 경험이라고 생각 되었다. 

특히 간단해진 코드로 인하여 코드의 직관성이 높아지고 이전에 사용되던 코드를 그대로 사용할 수 있어 괜찮았다. 

하지만 제네릭에 대한 지원에 문제가 있는거 같았고 이 때문에 `@AndroidEntryPoint`대상의 제네릭을 제거 해야 했다. 그런데 어차피 dagger를 사용 하면서 제네릭에 대한 적용이 어려워서 `@JvmSuppressWildcards`와 같은 어노테이션을 도배하듯이 사용 해야 했던 경험이 있었고 이 경험은 그대로 dagger-hilt에도 이어진다. 

결국 dagger(hilt)는 사용하기 까다롭지만 어쩔수 없이 선택할 수 밖에 없다는 생각이 들었다. 몰론 koin이라는 대안이 있기는 하지만 말이다. 그만큼 dagger는 아직 의존성 주입이라는 도구의 틀 안에서 가장 좋은 효율을 내고 있으며 dagger hilt를 통해 더 나은 코드 생산성을 제공 해주고 있어 조금 더 기대를 해도 될거 같다. 