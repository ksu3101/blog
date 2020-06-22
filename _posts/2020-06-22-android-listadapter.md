---
title: Jetpack - ListAdapter
author: 강성우
layout: post
category: [Android, RecyclerView, Jetpack]
tag: [android, jetpack]
---

`ListAdapter`는 support library에 추가된 라이브러리로서 기존 `RecyclerView`의 Adapter에 이전 데이터셋 과 새로운 데이터셋의 비교를 담당하는 `DiffUtil`과 함께 사용 된다.

`ListAdapter`의 상속구조를 보면 기존에 사용 되던 `RecyclerView.Adapter`를 상속 했음을 확인 할 수 있다.

```
java.lang.Object
   ↳	RecyclerView.Adapter<VH extends RecyclerView.ViewHolder>
 	   ↳	ListAdapter<T, VH extends RecyclerView.ViewHolder>
```

기본적으로 나가기에 앞서 `RecyclerView`에 대해 간단하게 정리 하면 아래와 같다. 

1. `RecyclerView`를 통해 보여줄 데이터 List는 `Adapter`에서 관리 한다. 
2. `Adapter`에서는 List의 각 항목에 대응될 item View에 대한 레퍼런스를 갖는 `ViewHoler`를 생성 하거나 재사용 한다.
3. `LayoutManager`에서는 item View들의 Pool을 관리 하며 필요 한 경우 재사용 하기 위해 해당 View를 `Adapter`에 전달 한다.

### 1. `DiffUtil`

예전에는 데이터의 변경을 알리기 위해서 `RecyclerView`의 `Adapter` 인스턴스에 대해 `notifyDataSetChanged()`와 같은 `notify~()`함수들을 호출 하여 데이터 변경을 알려 주었다. MVVM에서는 `ObservableList`을 구현하여 `OnListChangedListener`인터페이스 콜백을 구현하여 데이터의 변경을 알리는 방법과 `LiveData`를 이용 하는 방법등이 있었다.

위 두 방법은 데이터 셋의 변경에 대한 콜백을 만들거나 `notify~`함수들을 호출 하여 `Adapter`에 알려야 하는 보일러플레이트 코드가 생기며 필요한 경우 일일히 `notify~`콜을 해야 한다. 

`DiffUtil`은 위의 문제를 대신하여, 두 개의 `List`에서 원소의 차이를 계산하고 업데이트 해주는 유틸리티 클래스로서 `notify~`와 `OnListChangedListener`을 대신하여 리스트 항목의 차이를 위임하여 처리 하게 한다. 

> `DiffUtil`은 Eugene W.Myers의 difference algorithm을 기반으로 이전 List에서 새로운 List로 변환하기 위해 최소한의 업데이트를 계산하여 처리 해 준다. time complexity 는 O(N) O(N+D^2)이며, O(N)의 공간을 사용 한다.

`DiffUtil`에서는 `Adapter`에서의 업데이트를 계산하는데 사용 되며 이를 백그라운드 스레드 에서 동작 하게 하여 퍼포먼스 이슈를 피할수 도 있다. 이 경우 `DiffUtil.DiffResult`를 통해 `RecyclerView`에 main thread를 통해 적용 하면 된다. 

`DiffUtil`의 사용을 위해 참조할 만한 평균 실행 시간이다. (Android dev 페이지 에서 제공 하는 내용)

- 100 개 항목 및 10 개 수정 : 평균 : 0.39ms, 중앙값 : 0.35ms
- 100 개 항목 및 100 개 수정  : 3.82ms, 중앙값 : 3.75ms
- 100 항목 및 100 수정없이 이동 : 2.09 ms, 중앙값 : 2.06 ms
- 1000 개 항목 및 50 개 수정 : 평균 : 4.67ms, 중앙값 : 4.59ms
- 이동없이 1000 개의 항목과 50 개의 수정 : 평균 : 3.59ms, 중앙값 : 3.50ms
- 1000 개 항목 및 200 개 수정 : 27.07ms, 중앙값 : 26.92ms
- 이동없이 1000 개 항목 및 200 개 수정 : 13.54ms, 중앙값 : 13.36ms

#### 1.1 `DiffUtil.ItemCallback<T>`


```kotlin
// data 
data class Product(
   val id: Long,
   val name: String,
   // ...
)

// Diff callback implements
private class ProductItemDiffCallback : DiffUtil.ItemCallback<Product>() {
    override fun areItemsTheSame(oldItem: Product, newItem: Product): Boolean {
        return oldItem.id == newItem.id
    }

    override fun areContentsTheSame(oldItem: Product, newItem: Product): Boolean {
        return oldItem == newItem
    }
}
```

`Product`라는 데이터 원소가 있을 때 `DiffUtil.ItemCallback<Product>`의 구현 예제 이다. 

- `areItemsTheSame()` 
  - 이전 리스트 항목과 새로운 리스트의 항목이 같은지 여부를 비교 한다. 
  - 예제 에서는 유니크 한 `id`값을 이용하여 비교 하였다. 
- `areContentsTheSame()`
  - 이전 리스트 항목과 새로운 리스트의 항목 두개의 데이터가 같은지(equality)를 비교 한다. 
  - 이 함수에서는 해당 클래스에 대해 `equals()`메소드를 호출 하여 데이터의 레퍼런스를 비교 한다. 코틀린에서는 동일한 역활을 갖는 `==`키워드를 사용 한다. 
  - `areItemsTheSame()`에서 `true`를 반환했을 때 이 함수가 호출 된다. 

결론은 변경이 일어난 것 으로 감지된 새로운 데이터 와 이전 데이터를 비교 하는데, 

1. `areItemsTheSame()` 함수에서 이전 데이터 항목과 새로운 데이터항목의 가시적 항목을 비교 한다. (예를 들면 보여주는 id, text 등. 이 때 꼭 가시적인 항목이 아닌 유니크한 id처럼 보이지 않는 값 일 수도 있다) 
2. `areItemsTheSame()` 에서 true로 반환된 항목일 경우 `areContentsTheSame()`에서 이전 데이터와 새로운데이터가 같은 레퍼런스를 갖는지 확인 한다. 

#### 1.2 `AsyncListDiffer` 

`AsyncListDiffer`는 `DiffUtil`을 통한 데이터의 비교를 백그라운드 스레드를 통해 `execute()`한다고 생각 하면 된다. 그렇기 때문에 많은 항목이 존재 하는 데이터 목록 에서 예측 되는 많은 변화에 대해 퍼포먼스 측면에서 안전하게 UI를 갱신할 수 있도록 해 준다. 몰론 데이터의 비교를 위해서는 `DiffUtil`의 콜백 구현이 필요하다. 

사용 하기 위해서는 `AsyncListDiffer`인스턴스의 `currentList`를 데이터 목록처럼 사용 하면 된다. 예제 코드는 아래와 같다.

```kotlin
class RecyclerViewModelAsyncDiffAdapter<E>(
    diffCallback: DiffUtil.ItemCallback<E>
): RecyclerView.Adapter<RecyclerViewModelAsyncDiffAdapter.DataBindingVieWHolder<E>>() {
    private val differ = AsyncListDiffer(this, diffCallback)

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): DataBindingVieWHolder<E> {
        val binding = DataBindingUtil.inflate<ViewDataBinding>(
            LayoutInflater.from(parent.context), viewType, parent, false)
        val viewHolder = DataBindingVieWHolder<E>(binding)        
        return viewHolder
    }

    override fun getItemCount(): Int = differ.currentList.size

    override fun onBindViewHolder(holder: DataBindingVieWHolder<E>, position: Int) {
        holder.bind(differ.currentList[position])
    }

    class DataBindingVieWHolder<E>(
        private val binding: ViewDataBinding
    ): RecyclerView.ViewHolder(binding.root) {
        fun bind(item: E) {
            binding.setVariable(BR.vm, item)
            binding.executePendingBindings()
        }
    }
}
```

예제에서는 제네릭 `<E>`로 데이터를 표현했으며 실제 구현에서는 제네릭에 해당 되는 데이터 클래스의 `DiffUtil`콜백일 직접 구현해야 한다. 

### 2. `ListAdapter`

`ListAdapter`는 기존 `Adapter`만드는 과정을 많이 줄여준다. 그 얘기는 유지, 보수를 위해 손을 대야 하는 부분이 많이 줄어들었다는 것 이다. 하지만 그만큼의 제약이 생기게 되는데 감수하고 쓸만 하다고 생각 된다. 

```kotlin
class RecyclerViewModelAdapter<E>(
    diffCallback: DiffUtil.ItemCallback<E>
) : ListAdapter<E, RecyclerViewModelAdapter.DataBindingViewHolder<E>>(diffCallback) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): DataBindingViewHolder<E> {
        val binding = DataBindingUtil.inflate<ViewDataBinding>(
            LayoutInflater.from(parent.context), viewType, parent, false)
        val viewHolder = DataBindingViewHolder<E>(binding)
        return viewHolder
    }

    override fun onBindViewHolder(holderDataBinding: DataBindingViewHolder<E>, position: Int) {
        holderDataBinding.bind(getItem(position))
    }

    class DataBindingViewHolder<E>(
        private val binding: ViewDataBinding
    ) : RecyclerView.ViewHolder(binding.root) {
        fun bind(item: E) {
            binding.setVariable(BR.vm, item)
            binding.executePendingBindings()
        }
    }
}
```

이전 `RecyclerViewModelAsyncDiffAdapter`와 비교하면 내부에 `AsyncListDiff`인스턴스의 존재 및 사용만 다를 뿐 이다. 

이렇게 만들어진 `Adapter`의 인스턴스에 리스트 목록을 제공 하기 위해서는 `ListAdapter`의 `submitList()`함수를 통해 전달 하면 된다. 
