---
title: AndroidX - ActivityResultContract
author: 강성우
layout: post
category: [Android]
tag: [android]
---

안드로이드 컴포넌트 중 하나인 Activity간의 데이터를 처리 하는 방법으로 `startActivityForResult()`과 `onActivityResult()` 콜백이 있다. 이 방식의 Activity result처리 방법을 개선한 `ActivityResultContract`에 대해서 간단하게 알아보려 한다. 

`ActivityResultContract` API 패키지는 `activity:1.2.0-alpha02, fragment:1.3.0-alpha02`버전 부터 도입 되었으며, `alpha04`버전 부터는 `startActivityForResult(), onActivityResult()`메소드 들은 Deprecate 되었다. 

`ActivityResultContract`는 이 글 작성시점에서 정식 릴리즈되지 않았기 때문에 사용은 추천되지 않으며 실제 이 글과 정식 릴리즈 버전과 차이가 있을 수 있다. 이 글에서는 `ActivityResultContract`에 대한 찍어먹기로 어떤 개념인지 가볍게 알아보려 한다. 

### 1. `startActivityForResult(), onActivityResult()` 방법

이전 Activity간 결과 데이터를 핸들링 하기 위해서는 아래와 같은 방법으로 사용 되었다. 

```kotlin
class SomeActivity: Activity() {
    fun onClickedNextButton() {
        val intent = Intent(context, MainActivity::class.java)
        activity.startActivityForResult(intent, REQUEST_CODE_MAIN_ACTIVITY)
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (requestCode == REQUEST_CODE_MAIN_ACTIVITY) {
            if (resultCode == Activity.RESULT_OK) {
                // ...
            } 
            else if (resultCode == Activity.RESULT_CANCELED) {
                // ...
            }
        }
    }
}
```

### 2. 새로운 API 

androidX 에서 제공하는 API의 사용 예는 아래와 같다. 

```kotlin
class SomeActivity: Activity() {
    val activityResultHandler: ActivityResultLauncher<Intent> = registerForActivityResult(
            ActivityResultContracts.StartActivityForResult()
        ) { result ->
            // ... 
        }

    fun startMainActivity() {
        val intent = Intent(context, MainActivity::class.java)
        activityResultHandler(intent)
    }
}
```

새로운 API 에서는 `ActivityResultLauncher`을 사용 한다. 기존 방법과 동일하게 `Intent`를 사용 하지만 `startActivityForResult()`와 `onActivityResult()`을 사용 하지 않으며 멤버로 구현된 `ActivityResultLauncher` 을 이용 하여 `ActivityResult`를 핸들링 하는 람다가 있음을 확인할 수 있다. (참고로 Fragment에서의 사용법은 동일 하다.)

자세히 보면, `registerForActivityResult()`함수를 이용 해서 `StartActivityForResult`을 넘겨 `ActivityResult`를 `ActivityResultCallback`인터페이스를 람다 구현의 블럭내로 전달 해 준다. `ActivityResultCallback`의 구현 내부는 이전 과 동일하긴 하지만 `ActivityResult`라는 데이터 클래스가 사용 된다. 

```java
public final class ActivityResult implements Parcelable {
    private final int mResultCode;
    @Nullable
    private final Intent mData;

    //...
}
```

기존 `onActivityResult()`에서 사용 되던 `requestCode`는 더이상 사용되지 않아 없어졌으며 `resultCode`와 `Intent`인 `data`는 그대로 임을 확인 할 수 있다. `requestCode`는 이전 result를 구분하기 위해 launch시점의 구분 플래그 역활을 했었지만 이제는 람다 함수로 구현되어 콜백을 받기 때문에 requestCode로 호출자를 구분할 필요가 없어져서 그런것 같다. 

### 3. 결말 

아직 알바버전에 안정된 정식 릴리즈버전이 아니어서 실제 사용에는 추천되지 않는다. 사용해본 바 에는 이전의 메소드 재정의를 통한 콜백으로 result를 받아 내부 코드가 비대해지는 일이 많았는데 람다로 구현 되면서 더 직관적이고 유지, 보수가 쉬운 코드가 될 수 있을거 같아 기대중이다. 


