---
title: AndroidX - Navigation
author: 강성우
layout: post
category: [Dagger, Kotlin]
tag: [dagger, kotlin, android]
---

`Navigation`은 안드로이드 컴포넌트가 어떤것으로 구현되었는지 여부에 상관없이 Fragment, Activity간 안정적인 이동을 구현하기 위한 API를 제공 한다. 
필요한 의존 항목의 선언은 `Navigation`을 사용할 모듈의 `build.gradle`에 아래 항목을 `dependencies`에 추가 하면된다. 

```
// Kotlin
implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
implementation "androidx.navigation:navigation-ui-ktx:$nav_version"

// Dynamic Feature Module Support
implementation "androidx.navigation:navigation-dynamic-features-fragment:$nav_version"

// Testing Navigation
androidTestImplementation "androidx.navigation:navigation-testing:$nav_version"
```

### 1. Safe args 

안드로이드 컴포넌트간 탐색시 Safe args라는 gradle 플러그인을 이용 하여 레퍼런스에 대해 안정성을 제공하는 API도구 클래스들을 사용 할 수 있다. Safe args는 `Bundle`인스턴스를 통해 패러미터를 전달 했던 기존 과정을 더 쉽고 안정성 있게 처리 할 수 있도록 도와준다. 

프로젝트에 Safe Args를 추가 하려면 최상위 root `build.gradle`파일에 아래와 같은 `classpath`를 `dependencies`블럭 에 추가 한다. 

```
buildscript {
    dependencies {
        def nav_version = "2.3.0-beta01"
        classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$nav_version"
    }
}
```

그리고 Safe Args를 사용할 대상 모듈의 `build.gradle`파일에 아래 plugin적용을 추가 한다. 

```
apply plugin: 'androidx.navigation.safeargs'        // for Java
apply plugin: 'androidx.navigation.safeargs.kotlin' // for Kotlin  
```

Safe Args를 사용하게 될 경우 생성될 코드 에는 송,수신 대상과 함께 아래와 같은 레퍼런스 타입 안전 클래스와 메소드가 포함된다. 

- 작업이 시작되는 각 대상에 클래스가 생성 된다. 클래스 이름은 대상 클래스에 `Directions`라는 단어가 추가된다. 예를 들어 `SomeFragment`가 발신 대상이라면 생성될 클래스의 이름은 `SomeFragmentDirections`가 된다. 
- 작업은 인수로 선언된 인스턴스 인데, 이 이름을 기반으로 내부 클래스가 만들어 진다. 예를 들어 인수 이름이 `val someAction` 이라면 `SomeAction`이라는 클래스 이름으로 생성 된다. 
- 수신 대상에 클래스가 만들어지며, 해당 클래스 이름은 `Args`라는 단어가 추가된다. 예를 들어 `SomeFragment`라면 생성될 클래스의 이름은 `SomeFragmentArgs`가 된다. 이 클래스에서 인수는 `fromBundle()`메소드를 이용하여 인수를 찾을 수 있다. 

> "작업"이란 화면을 이동시키게 만드는 어떠한 Fragment, Activity간 흐름을 만들어 내는 Trigger라고 생각 하면 될거 같다. 예를 들어 사용자가 id,pw를 모두 입력 후 "로그인 하기" 버튼을 누른 다음 어떠한 비즈니스 로직 후 메인 화면으로 이동 하는 일들 모두를 "작업" 이라고 할 수 있다. 

### 2. 예제 프로젝트로 알아보기

예제 프로젝트를 통해서 `Navigation`의 구현 예제를 알아보도록 하자. 예제 프로젝트는 책의목록을 restful api를 통해 가져와 보여주고 목록의 책 항목을 선택시 상세 화면으로 이동 한다. 이 예제로 사용될 프로젝트의 구조는 아래와 같다. 

- `BookSearchActivity` : 메인 엑티비티. 
  - `BookSearchFragment` : 책의 목록을 가져와서 보여준다. 
  - `BookDetailFragment` : 위 목록중 책을 선택 시 해당 책의 상세 화면으로 이동 한다. 

#### 2.1 `BookSearchActivity`

`BookSearchActivity`액티비티는 메인 화면으로 예제 프로젝트에서 사용 될 휴식장소의 목록 을 보여주는 화면이다. 우선 메인 화면을 먼저 구성함으로서 시작 화면이 어디인지 정의 해 준다. 

메인 엑티비티의 레이아웃 구조는 아래와 같다. 

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/booksearch_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/white">

    <!-- 관련 없는 위젯은 생략 했음 -->
    <com.google.android.material.appbar.AppBarLayout
        ...>
        <androidx.appcompat.widget.Toolbar
            .../>
    </com.google.android.material.appbar.AppBarLayout>

    <fragment
        android:id="@+id/fragmentContainer"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_main" />
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

`fragment`를 사용 하였는데 내부에 `android:name`프로퍼티와 `app`네임 스페이스하의 `defaultNavHost`와, `navGraph`를 확인 할 수 있다. 

- `android:name` : `Navigation`에서 어떠한 `NavHost`가 구현되어 사용될지 정의한다. 예제 프로젝트 에서는 fragment의 navigation을 사용할 것 이기 때문에 `NavHostFragment`으로 설정 하였다. 
- `app:defaultNavHost` : 기본 navigation host를 설정 하여 back키(뒤로가기) 기능등을 사용하도록 설정 한다. 
- `app:navGraph` : `Navigation`이 정의된 리소스 xml파일 이다.

#### 2.2 `nav_main.xml` 리소스 정의

`resources`폴더 하 에 `navigation`리소스 폴더를 추가 한다. 그리고 `navigation`폴더 안에 위 `navGraph`프로퍼티에서 정의된 `nav_main.xml`을 만들어 추가 한다. 추가된 `nav_main.xml`을 디자인 뷰 로 보면 아직은 아무것도 없다. 

상단 **New Destination**버튼을 클릭 하여 `BookSearchActivity`에서 최초로 보여질 `BookSearchFragment`를 찾아 추가 한다.

![navigation1](/blog/assets/images/and_nav/navigation1.png)

![navigation2](/blog/assets/images/and_nav/navigation2.png)

추가하고 난 뒤 디자인 뷰 에서는 아래와 같이 `BookSearchFragment`가 보인다. 

![navigation3](/blog/assets/images/and_nav/navigation3.png)

위 추가된 `BookSearchFragment`을 자세히 보면 라벨 왼쪽에 Home아이콘이 추가 되어 있다. 이는 최초 실행시 보여지게 될 화면을 뜻하며 상단 메뉴 중 동일한 모양의 아이콘을 이용해 다른 화면을 Home으로 변경 할 수도 있다. 

다음으로 `BookDetailFragment`를 추가 한다. 추가 방법은 동일 하며 그 결과는 아래와 같다. 

![navigation4](/blog/assets/images/and_nav/navigation4.png)

#### 2.3 Action 설정

`Navigation` 에서 Action은 각 컴포넌트간 이동을 정의 한 것 이다. 보통 `from`에서 `destination`을 정의 하며 추가적으로 transition animation을 설정 하거나 backstack에 저장여부 까지 설정 할 수 있다. Action의 종류에는 여러가지가 더 있지만 이번에는 일반적으로 사용 되는 destination을 설정 하는 방법으로 이용 해 보겠다. 

Action의 추가 방법은 여러가지가 있는데, 

1. navigation graph ui 화면에서 추가한 컴포넌트의 모서리 둥근 버튼을 누른 상태에서 드래그 하여 이동할(액션을 지정할) 대상 컴포넌트로 이동 하면 기본적인 Action이 정의 된다. 

![navigation5](/blog/assets/images/and_nav/navigation5.png)

2. Action이 시작되는 "from" 컴포넌트 에서 마우스오른쪽 버튼을 눌러 "Add Action" 을 한다. 

![navigation6](/blog/assets/images/and_nav/navigation6.png)

3. 아니면 오른쪽 "Attributes" 메뉴 아래에서 Action을 추가 할 수도 있다. 

추가된 기본액션은 아래처럼 graph ui 에서 화살표로 보여진다. 

![navigation7](/blog/assets/images/and_nav/navigation7.png)

추가된 화살표를 마우스 오른쪽을 눌러 "Edit" 메뉴를 선택 하면 해당 Action의 상세 설정을 할 수 있다. 

![navigation8](/blog/assets/images/and_nav/navigation8.png)

#### 2.4 Safe args 추가 하기 

Safe args는 예전에 안드로이드 컴포넌트간 데이터전달을 하기 위해 `Intent`에 `Bundle`을 담아 주던 것 과 비슷하다고 생각 하면 된다. 하지만 `Bundle`보다 데이터 안정성이 보장 되며 좀 더 편하게 사용 할 수 있다. 

Safe args설정은 데이터를 전달 받을 수신측 컴포넌트에 추가 한다. 예를 들어 예제의 `BookDetailFragment`에서는 책의 상세정보를 보여주기 위하여 `Book` data class 객체를 받도록 설정 한다. 

Safe args를 추가하려면 오른쪽의 `attributes`항목에서 `Arguments`의 +버튼을 눌러 추가하면 된다. 

![navigation9](/blog/assets/images/and_nav/navigation9.png)

+ 버튼을 누르고 등장하는 `Add Argument`팝업 에서 데이터를 설정 할 수 있다. 기본적으로 제공 되는 Int, String 에서부터 `Parcelable`인터페이스를 구현한 데이터 클래스 도 할 수 있다. 

![navigation10](/blog/assets/images/and_nav/navigation10.png)


#### 2.5 Navigation with Arguments

`navigation`리소스의 상세를 완성하고 난 뒤 자동으로 생성되는 `~Directions`와 `navArgs()`함수를 통해 argument를 위임받아 초기화 할 수 있게 된다. 

##### 2.5.1 네비게이션 예제 

```kotlin
val direction = BookSearchFragmentDirections.actionBookSearchFragmentToBookDetailFragment(book)
activity.findNavController(R.id.fragmentContainer).navigate(direction)
```

위 예제 코드를 보면 생성된 `~Directions`의 정적메소드를 통해서 navigation함을 알 수 있다. 

##### 2.5.2 arguemtn 예제 

```kotlin
class BookDetailFragment: BaseFragment<BookSearchState>() {
  // ...
  private val vm: BookDetailViewModel by viewModels { vmFactory }
  private val bookDetailArgs by navArgs<BookDetailFragmentArgs>()

  override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        vm.setBook(bookDetailArgs.book)
    }
}
```

예제에서 보면 `navArgs()`함수를 이용 하여 argument를 위임하여 초기화 하고 있다. `BookDetailFragmentArgs` 또한 자동으로 생성된 클래스 이다. 
