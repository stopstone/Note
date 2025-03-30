# 뷰홀더 패턴 (ViewHolder Pattern)

> 이 문서는 안드로이드 개발에서 성능 최적화를 위한 중요한 디자인 패턴 중 하나인 **뷰홀더 패턴(ViewHolder Pattern)** 에 대해 설명합니다.  
> 스크롤 가능한 리스트 UI를 구현할 때 발생하는 성능 문제를 해결하기 위해 도입된 이 패턴은 특히 **RecyclerView**에서 필수적으로 사용됩니다.  
> 본 문서에서는 ViewHolder 패턴의 정의와 동작 원리, ListView와 RecyclerView에서의 차이점, 실용적인 구현 예시 및 주의사항을 다룹니다.  

---

## ViewHolder란?

**ViewHolder**는 각 View의 참조를 보관하는 객체입니다.  
`RecyclerView`, `ListView`와 같은 리스트 형태의 UI에서 `findViewById()`를 반복 호출하지 않도록 방지하는 역할을 합니다.

예를 들어, 10개의 데이터를 가진 리스트는 문제없이 작동할 수 있지만,  
수만 개의 아이템이 스크롤될 경우 매번 `findViewById()`를 호출하면 성능 저하가 심각해질 수 있습니다.  
이러한 상황을 방지하기 위해 ViewHolder 패턴이 도입되었습니다.

---

## ViewHolder 패턴이란?

ViewHolder 패턴은 다음과 같은 목적을 가집니다:

- 뷰 재사용: 리스트 항목 View를 매번 새로 만들지 않고 재사용합니다.
- `findViewById()` 최소화: ViewHolder에 구성 요소들을 저장하여 반복 호출을 줄입니다.
- 성능 개선: 스크롤이 부드러워지고, 메모리 사용도 최적화됩니다.

---

## ListView vs RecyclerView

### ListView의 문제점

ListView는 `getView()` 메서드를 통해 리스트 항목을 생성하며, `convertView`를 통해 재활용 가능한 View를 전달받습니다. 하지만 매번 `findViewById()`를 호출하는 경우가 많아 성능 저하가 발생할 수 있습니다.

`convertView`는 스크롤 시 사라진 View를 재사용하기 위해 넘겨주는 파라미터입니다.

```kotlin
override fun getView(position: Int, convertView: View?, parent: ViewGroup?): View {
    val viewHolder: ViewHolder
    val view: View

    if (convertView == null) {
        view = LayoutInflater.from(parent?.context).inflate(R.layout.item_list, parent, false)
        viewHolder = ViewHolder(view)
        view.tag = viewHolder
    } else {
        view = convertView
        viewHolder = view.tag as ViewHolder
    }

    viewHolder.textView.text = items[position]
    return view
}

class ViewHolder(view: View) {
    val textView: TextView = view.findViewById(R.id.textView)
}
```

ListView는 ViewHolder 패턴을 강제하지 않기 때문에, 개발자가 명시적으로 구현하지 않으면 매번 `findViewById()`를 호출하게 되고 성능 저하로 이어집니다.

### RecyclerView의 개선점

RecyclerView는 ViewHolder 패턴을 구조적으로 강제합니다. 다음 세 가지 메서드를 통해 뷰 재사용이 자연스럽게 이루어집니다:

- `onCreateViewHolder()`: View를 생성하여 ViewHolder에 전달
- `onBindViewHolder()`: ViewHolder에 데이터를 바인딩
- `getItemCount()`: 아이템 개수 반환

```kotlin
class CustomAdapter(private val dataSet: Array<String>) : RecyclerView.Adapter<CustomAdapter.ViewHolder>() {

    class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val textView: TextView = view.findViewById(R.id.textView)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val view = LayoutInflater.from(parent.context).inflate(R.layout.text_row_item, parent, false)
        return ViewHolder(view)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.textView.text = dataSet[position]
    }

    override fun getItemCount() = dataSet.size
}
```

`onCreateViewHolder()`에서만 `findViewById()`를 호출하고, `onBindViewHolder()`에서는 데이터를 단순히 바인딩하기 때문에 성능 저하 없이 부드러운 스크롤이 가능합니다.

---

## ViewHolder 사용 시 안티패턴

RecyclerView에서 ViewHolder 패턴을 사용할 때는 다음과 같은 안티 패턴을 피하고, 바람직한 사용 방식을 지켜야 합니다.

### 1. setOnClickListener를 반복 설정하는 안티 패턴

`onBindViewHolder()`에서 매번 `setOnClickListener`를 설정하면 불필요한 객체 생성이 발생하고 성능에 악영향을 줄 수 있습니다. 대신 ViewHolder 초기화 시 한 번만 설정하고 콜백으로 위치만 넘겨주는 방식이 좋습니다.

```kotlin
class MyAdapter(private val onItemClick: (Int) -> Unit) : RecyclerView.Adapter<MyAdapter.MyViewHolder>() {

    class MyViewHolder(itemView: View, onItemClick: (Int) -> Unit) : RecyclerView.ViewHolder(itemView) {
        init {
            itemView.setOnClickListener {
                onItemClick(adapterPosition)
            }
        }
    }

    // 기타 생략
}
```

ViewHolder를 inner class로 선언하면 어댑터에 대한 암묵적 참조가 생겨 메모리 누수 위험이 높아집니다. 따라서 ViewHolder는 inner class가 아닌 **static nested class 형태**로 정의하는 것이 바람직합니다.

### 2. 뷰 상태 바인딩 누락으로 인한 재활용 문제

CheckBox 상태를 사용자가 변경한 뒤 `onBindViewHolder()`에서 상태를 다시 바인딩하지 않으면, 재활용된 뷰가 이전 상태를 유지하게 됩니다.

```kotlin
override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
    val item = itemList[position]

    holder.checkBox.isChecked = item.isChecked

    holder.itemView.setOnClickListener {
        item.isChecked = !item.isChecked
        notifyItemChanged(position)
    }
}
```

데이터 모델(`item.isChecked`)을 통해 상태를 저장하고 항상 바인딩해야 재활용 문제를 방지할 수 있습니다.

### 3. ViewHolder를 inner class로 선언하는 경우

```kotlin
inner class MyViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
    // ...
}
```

위와 같이 ViewHolder를 inner class로 선언하면 Adapter와 Activity를 포함한 외부 클래스에 대한 참조가 암묵적으로 생기기 때문에, 해당 ViewHolder가 오래 생존할 경우 **메모리 누수**로 이어질 수 있습니다. 따라서 반드시 **정적 중첩 클래스(static nested class)**로 선언하는 것이 권장됩니다.

---

## ViewBinding과 ViewHolder

ViewBinding을 사용하면 `findViewById()`를 사용할 필요 없이 자동으로 View를 참조할 수 있어 코드가 더 간결하고 안전합니다.

```kotlin
class MyAdapter(private val items: List<String>) : RecyclerView.Adapter<MyAdapter.ViewHolder>() {

    class ViewHolder(val binding: ItemLayoutBinding) : RecyclerView.ViewHolder(binding.root)

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val binding = ItemLayoutBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return ViewHolder(binding)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        holder.binding.textView.text = items[position]
    }

    override fun getItemCount(): Int = items.size
}
```

---

## 면접 질문 정리

### 1. ViewHolder 패턴이란 무엇이며, 왜 사용하나요?

ViewHolder 패턴은 리스트 뷰에서 `findViewById()` 호출을 최소화하여 스크롤 성능을 향상시키는 디자인 패턴입니다. ViewHolder에 뷰 참조를 저장하여 불필요한 리소스 낭비를 줄이고 부드러운 UI 성능을 확보합니다.

### 2. ListView와 RecyclerView의 차이점은 무엇인가요?

ListView와 RecyclerView는 모두 리스트 형태의 UI를 구성할 수 있는 컴포넌트입니다. 하지만 RecyclerView는 ListView의 단점을 개선한 진화된 형태입니다.  
ViewHolder 패턴의 적용 방식에서 차이가 있습니다. ListView는 ViewHolder 패턴을 직접 구현해야 하지만, RecyclerView는 구조적으로 ViewHolder를 강제하여 성능 저하를 방지합니다.  
레이아웃 구성의 유연성입니다. ListView는 수직 방향의 단일 레이아웃만 지원하지만, RecyclerView는 Linear, Grid, Staggered 등 다양한 LayoutManager를 지원합니다.  
애니메이션 및 데코레이터 기능이 있습니다. RecyclerView는 아이템 변경 애니메이션, 구분선 등 다양한 확장 기능을 제공하여 UI를 더 풍부하게 구성할 수 있습니다.

---

> ViewHolder는 재사용 가능한 뷰에 데이터를 바인딩하는 역할만을 수행해야 하며,
> 무거운 연산, 뷰 상태 저장, 네트워크 호출 등의 로직은 외부(ViewModel 등)로 분리하는 것이 유지보수성과 성능 측면에서 바람직합니다.


## 참고 자료

- [Android Developers - RecyclerView](https://developer.android.com/guide/topics/ui/layout/recyclerview)
- [Haero 블로그 - findViewById 원리](https://math-coding.tistory.com/251)
- [Bellgori 블로그 - ViewHolder Pattern](https://bellgori.tistory.com/entry/Android-pattern-01-ViewHolder-pattern)
- [PartsTech 블로그 - ViewHolder 설명](https://blog.naver.com/partstech2700)

