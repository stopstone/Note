# XML 레이아웃의 include와 merge

> Android에서 `include` 태그와 `merge` 태그는 **중복되는 레이아웃 구조를 개선하고,  
> 계층을 간소화하는 데 유용한 구성 요소들**입니다.  
> 이 글에서는 각각의 태그가 어떤 역할을 하는지, 언제 사용해야 하는지,  
> 그리고 사용 시 얻을 수 있는 이점에 대해 정리해보겠습니다.

## 1. include 태그는 무엇일까?

`<include>` 태그는 다른 XML 레이아웃을 현재 레이아웃에 포함하는 방식입니다.  
가장 편리하고 구조적으로도 명확합니다.

```xml
<include layout="@layout/view_toolbar" />
```

구조적으로 다른 레이아웃을 같은 위치에 다시 사용할 때 사용합니다.

## 2. merge 태그는 무엇일까?

`<merge>` 태그는 특이한 용도가 있습니다.  
간단히 말하면, 각각 레이아웃의 가장 바깥 container 개체를 제거하고 불필요한 계층을 줄이며 성능을 개선하는 요소입니다.

```xml
<!-- view_toolbar.xml -->
<merge xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView ... />
    <ImageButton ... />
</merge>
```

이와 같은 경우 `<merge>`가 적용된 외부 XML의 root는 `LinearLayout` 또는 `ConstraintLayout` 같은 그룹을 가지고 있어야 합니다.

```xml
<LinearLayout ...>
<include layout="@layout/view_toolbar" />
        </LinearLayout>
```

이때 `<include>`로 들어오지만, `view_toolbar.xml`의 root가 `<merge>`이기 때문에 `LinearLayout`이 두 개 생길 필요가 없고, 실제로는 하나의 `LinearLayout`에 삽입되는 구조가 됩니다.

## 3. include와 merge의 차이점

| 요소       | include                    | merge                                    |
| -------- | -------------------------- | ---------------------------------------- |
| 구조적 특징   | 다른 XML을 불러오며 root view도 포함 | root가 제거되고 내부의 view들을 지정                 |
| View 계층  | 외부 구조 + 내부 XML root        | 외부 구조가 root로, 내부 view들을 다시 넣음            |
| 사용 가능 경우 | 여러 레이아웃에 도움                | 직접 container 가지는 XML에 추가 넣을 때 복잡성 제거를 위해 |
| 성능 이점    | 없음                         | 계층 간소화 제공, 복잡한 구조 방지                     |

## 4. 언제 사용해야 할까?

### include

* 공통적인 방식으로 여러 레이아웃에 반복적으로 사용될 경우 (예: 툴바, 푸터 등)

### merge

* 복잡한 root view 계층을 마지막 계층에서 제거하고, 직접 내용 view를 넣어야 할 경우
* ConstraintLayout 또는 LinearLayout 등을 많이 사용하는 경우
* 성능 최적화를 고려해야 할 경우

## 참고자료

* [Android Developers - Reusing Layouts](https://developer.android.com/guide/topics/ui/declaring-layout#ReusingLayouts)
* [Android Developers - ViewStub](https://developer.android.com/reference/android/view/ViewStub)
