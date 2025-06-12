# notifyDataSetChanged() 대신 무엇을 써야 할까?

> RecyclerView에서 데이터가 변경되었을 때 UI에 반영하는 대표적인 메서드 중 하나인 `notifyDataSetChanged()`는 간단해 보이지만, 실제로는 피해야 할 이유가 많은 메서드입니다.  
> 이 글에서는 왜 `notifyDataSetChanged()`를 쓰면 안 되는지, 어떤 문제가 발생하는지, 그리고 대안은 무엇인지 정리해보겠습니다.  

## 1. 모든 항목이 다시 바인딩된다

`notifyDataSetChanged()`는 RecyclerView에게 **모든 아이템이 바뀌었다**고 통지합니다.  
따라서 실제 변경된 아이템이 일부일지라도, **전체 아이템이 다시 바인딩**됩니다.  

이는 다음과 같은 문제를 유발합니다:

* 불필요한 연산 증가
* 복잡한 뷰의 경우 UI 깜빡임 발생
* 스크롤 지연 등 UX 저하

## 2. 애니메이션 효과가 무시된다

RecyclerView는 아이템 변경에 따른 **삽입/삭제/이동 애니메이션**을 제공합니다.  
하지만 `notifyDataSetChanged()`는 **전체를 리프레시하기 때문에 애니메이션이 동작하지 않습니다.**

이로 인해 사용자 입장에서 어떤 아이템이 바뀐 건지 알 수 없어 UX가 떨어지고, 변경의 흐름을 자연스럽게 따라가기 어렵습니다.

## 3. ViewHolder 재사용이 비효율적이다

RecyclerView는 ViewHolder를 재활용하면서 효율적으로 UI를 렌더링합니다.  
그런데 `notifyDataSetChanged()`는 내부적으로 **모든 아이템을 재바인딩**하면서 **ViewHolder 풀의 재사용 효율성을 떨어뜨립니다.**

그 결과:

* ViewHolder가 매번 재생성되며 오버헤드 발생
* setHasStableIds(true) 설정 시에도 안정된 ID 기반 최적화가 무력화됨

## 4. Android Studio에서도 경고한다

안드로이드 Lint는 다음과 같은 경고 메시지를 표시합니다:

```
It will always be more efficient to use more specific change events if you can.  
Rely on notifyDataSetChanged as a last resort.  
```

이는 곧, `notifyDataSetChanged()`는 **정말 마지막 수단으로만 사용하라**는 뜻입니다.

---

## 대안: 변경 범위를 명확히 지정하자

RecyclerView\.Adapter에는 다음과 같은 변경 통지 메서드들이 있습니다:

| 메서드                                    | 설명             |
| -------------------------------------- | -------------- |
| `notifyItemChanged(position)`          | 특정 위치의 아이템만 갱신 |
| `notifyItemInserted(position)`         | 아이템이 추가됨을 통지   |
| `notifyItemRemoved(position)`          | 아이템이 삭제됨을 통지   |
| `notifyItemMoved(from, to)`            | 위치가 이동됨을 통지    |
| `notifyItemRangeChanged(start, count)` | 범위 지정 변경       |

이러한 메서드들을 사용하면 **정확한 변경 범위만을 알려줄 수 있어**, 애니메이션도 자연스럽고, ViewHolder 재사용도 최적화됩니다.

---

## 더 나아가기: DiffUtil과 ListAdapter 활용

`notifyItemXXX()` 시리즈도 수동으로 작성하기 번거롭다면, 다음과 같은 고수준 도구를 사용하는 것이 좋습니다.

### 1. DiffUtil

* 새로운 리스트와 이전 리스트를 비교해 변경점을 계산
* `Adapter.notifyXXX()` 호출을 자동화해줌

### 2. ListAdapter

* `DiffUtil`을 내부적으로 포함한 추상화된 Adapter
* `submitList(newList)`만 호출하면 자동으로 변경 사항 계산 및 UI 반영

### 3. payload 활용

* `notifyItemChanged(position, payload)`를 통해 **부분 업데이트** 수행
* `onBindViewHolder(holder, position, payloads)`에서 payload 기반으로 뷰 업데이트 가능

---

## 참고 자료

* [RecyclerView.Adapter - Android Developers](https://developer.android.com/reference/androidx/recyclerview/widget/RecyclerView.Adapter#notifyDataSetChanged%28%29)
