# 컴포지트 패턴 (Composite Pattern)

> 컴포지트 패턴은 개별 객체와 복합 객체를 동일한 인터페이스로 처리할 수 있게 해주는 디자인 패턴입니다.  
> 안드로이드 개발에서는 View 계층 구조, 메뉴 시스템, RecyclerView Adapter 등에서 복잡한 트리 구조를 관리할 때 유용합니다.  
> 이 글은 **안드로이드 개발할 때 컴포지트 패턴 구현과 실제 활용법**을 정리한 문서입니다.

## 1. 개념

- **개별 객체와 복합 객체를 동일한 인터페이스로 처리할 수 있게 해주는 디자인 패턴**
- 트리 구조를 표현하고, Leaf(개별 객체)와 Composite(복합 객체)를 구분 없이 처리
- **장점:** 새로운 객체 타입 추가가 쉬우며, 클라이언트가 단일 객체와 복합 객체를 구분하지 않아도 됨
- **단점:** 타입 안전성 문제와 런타임 예외 발생 가능성

---

## 2. 기본 구조

```kotlin
// 컴포넌트 인터페이스
interface Component {
    fun operation(): String
    fun add(component: Component)
    fun remove(component: Component)
    fun getChild(index: Int): Component?
}

// Leaf (개별 객체)
class Leaf(private val name: String) : Component {
    override fun operation(): String = "Leaf: $name"
    
    override fun add(component: Component) {
        throw UnsupportedOperationException("Leaf는 자식을 가질 수 없습니다")
    }
    
    override fun remove(component: Component) {
        throw UnsupportedOperationException("Leaf는 자식을 가질 수 없습니다")
    }
    
    override fun getChild(index: Int): Component? {
        throw UnsupportedOperationException("Leaf는 자식을 가질 수 없습니다")
    }
}

// Composite (복합 객체)
class Composite(private val name: String) : Component {
    private val children = mutableListOf<Component>()
    
    override fun operation(): String {
        val result = StringBuilder("Composite: $name\n")
        children.forEach { child ->
            result.append("  ${child.operation()}\n")
        }
        return result.toString()
    }
    
    override fun add(component: Component) {
        children.add(component)
    }
    
    override fun remove(component: Component) {
        children.remove(component)
    }
    
    override fun getChild(index: Int): Component? {
        return if (index in children.indices) children[index] else null
    }
}
```

---

## 3. 안드로이드에서의 활용 예제

### 1. View 계층 구조

#### 실제 Android View 시스템과의 연관성

안드로이드의 View 시스템은 컴포지트 패턴의 대표적인 예시입니다. `ViewGroup`은 Composite 역할을, `View`는 Leaf 역할을 합니다.

```kotlin
// 실제 Android View 시스템의 컴포지트 패턴 적용
class CustomLinearLayout @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : LinearLayout(context, attrs, defStyleAttr) {
    
    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        // LinearLayout은 Composite 역할
        // 자식 View들을 순회하며 그리기
        for (i in 0 until childCount) {
            val child = getChildAt(i)
            child.draw(canvas)
        }
    }
    
    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)
        // 자식 View들의 크기 측정
        measureChildren(widthMeasureSpec, heightMeasureSpec)
    }
}

// 실제 Android TextView (Leaf 역할)
class CustomTextView @JvmOverloads constructor(
    context: Context,
    attrs: AttributeSet? = null,
    defStyleAttr: Int = 0
) : TextView(context, attrs, defStyleAttr) {
    
    override fun onDraw(canvas: Canvas) {
        super.onDraw(canvas)
        // TextView는 Leaf 역할 - 자식을 가질 수 없음
        // 텍스트만 그리기
    }
    
    // TextView는 addView() 메서드가 없음 (Leaf의 특징)
    // ViewGroup만 자식을 추가할 수 있음
}
```

#### View 계층 구조의 컴포지트 패턴 분석

```kotlin
// Android View 시스템의 컴포넌트 인터페이스 (실제 구현)
abstract class View {
    // 공통 인터페이스
    fun draw(canvas: Canvas) { /* 구현 */ }
    fun measure(widthMeasureSpec: Int, heightMeasureSpec: Int) { /* 구현 */ }
    fun layout(l: Int, t: Int, r: Int, b: Int) { /* 구현 */ }
    
    // Leaf에서는 사용 불가능한 메서드들
    fun addView(child: View) {
        throw UnsupportedOperationException("View는 자식을 가질 수 없습니다")
    }
    
    fun removeView(child: View) {
        throw UnsupportedOperationException("View는 자식을 가질 수 없습니다")
    }
}

// ViewGroup (Composite) - 실제 Android API
abstract class ViewGroup : View() {
    private val children = mutableListOf<View>()
    
    override fun addView(child: View) {
        children.add(child)
        // 실제 Android에서는 더 복잡한 로직
    }
    
    override fun removeView(child: View) {
        children.remove(child)
    }
    
    fun getChildAt(index: Int): View? {
        return if (index in children.indices) children[index] else null
    }
    
    fun getChildCount(): Int = children.size
}

// 실제 사용 예시
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 컴포지트 패턴이 적용된 실제 Android View 계층 구조
        val rootLayout = LinearLayout(this).apply {
            orientation = LinearLayout.VERTICAL
            layoutParams = ViewGroup.LayoutParams(
                ViewGroup.LayoutParams.MATCH_PARENT,
                ViewGroup.LayoutParams.MATCH_PARENT
            )
        }
        
        // Leaf 객체들 (자식을 가질 수 없음)
        val textView = TextView(this).apply {
            text = "안녕하세요"
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT,
                LinearLayout.LayoutParams.WRAP_CONTENT
            )
        }
        
        val button = Button(this).apply {
            text = "클릭하세요"
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT,
                LinearLayout.LayoutParams.WRAP_CONTENT
            )
        }
        
        // Composite 객체 (자식을 가질 수 있음)
        val subLayout = LinearLayout(this).apply {
            orientation = LinearLayout.HORIZONTAL
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.MATCH_PARENT,
                LinearLayout.LayoutParams.WRAP_CONTENT
            )
        }
        
        val subTextView = TextView(this).apply {
            text = "서브 텍스트"
            layoutParams = LinearLayout.LayoutParams(
                LinearLayout.LayoutParams.WRAP_CONTENT,
                LinearLayout.LayoutParams.WRAP_CONTENT
            )
        }
        
        // 컴포지트 패턴 적용: 동일한 인터페이스로 처리
        subLayout.addView(subTextView)  // Composite에 Leaf 추가
        rootLayout.addView(textView)    // Composite에 Leaf 추가
        rootLayout.addView(button)      // Composite에 Leaf 추가
        rootLayout.addView(subLayout)   // Composite에 Composite 추가
        
        setContentView(rootLayout)
    }
}
```

#### Fragment 시스템에서의 컴포지트 패턴

```kotlin
// Fragment도 컴포지트 패턴의 예시
abstract class Fragment {
    // 공통 인터페이스
    fun onCreate(savedInstanceState: Bundle?) { /* 구현 */ }
    fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? { /* 구현 */ }
    fun onViewCreated(view: View, savedInstanceState: Bundle?) { /* 구현 */ }
    
    // Fragment는 일반적으로 Leaf 역할 (자식을 가질 수 없음)
    fun addChildFragment(fragment: Fragment) {
        throw UnsupportedOperationException("일반 Fragment는 자식 Fragment를 가질 수 없습니다")
    }
}

// FragmentContainer (Composite) - 실제 Android API
class FragmentContainer : Fragment() {
    private val childFragments = mutableListOf<Fragment>()
    
    override fun addChildFragment(fragment: Fragment) {
        childFragments.add(fragment)
        // 실제 자식 Fragment 추가 로직
    }
    
    fun getChildFragment(index: Int): Fragment? {
        return if (index in childFragments.indices) childFragments[index] else null
    }
}
```

### 2. 메뉴 시스템

```kotlin
// 메뉴 아이템 인터페이스
interface MenuItem {
    fun display()
    fun execute()
    fun addItem(item: MenuItem)
    fun removeItem(item: MenuItem)
    fun getChildItem(index: Int): MenuItem?
}

// 메뉴 (Composite)
class Menu(private val name: String) : MenuItem {
    private val items = mutableListOf<MenuItem>()
    
    override fun display() {
        println("Menu: $name")
        items.forEach { item ->
            print("  ")
            item.display()
        }
    }
    
    override fun execute() {
        println("메뉴 '$name' 실행")
        items.forEach { item ->
            item.execute()
        }
    }
    
    override fun addItem(item: MenuItem) {
        items.add(item)
    }
    
    override fun removeItem(item: MenuItem) {
        items.remove(item)
    }
    
    override fun getChildItem(index: Int): MenuItem? {
        return if (index in items.indices) items[index] else null
    }
}

// 메뉴 액션 (Leaf)
class MenuAction(
    private val name: String,
    private val action: () -> Unit
) : MenuItem {
    
    override fun display() {
        println("Action: $name")
    }
    
    override fun execute() {
        println("액션 '$name' 실행")
        action()
    }
    
    override fun addItem(item: MenuItem) {
        throw UnsupportedOperationException("MenuAction은 자식을 가질 수 없습니다")
    }
    
    override fun removeItem(item: MenuItem) {
        throw UnsupportedOperationException("MenuAction은 자식을 가질 수 없습니다")
    }
    
    override fun getChildItem(index: Int): MenuItem? {
        throw UnsupportedOperationException("MenuAction은 자식을 가질 수 없습니다")
    }
}

// 복잡한 메뉴 시스템 예제
class NavigationMenu {
    private val rootMenu = Menu("메인 메뉴")
    
    fun setupMenu() {
        // 파일 메뉴
        val fileMenu = Menu("파일")
        fileMenu.addItem(MenuAction("새로 만들기") { println("새 파일 생성") })
        fileMenu.addItem(MenuAction("열기") { println("파일 열기") })
        fileMenu.addItem(MenuAction("저장") { println("파일 저장") })
        
        // 편집 메뉴
        val editMenu = Menu("편집")
        editMenu.addItem(MenuAction("복사") { println("복사") })
        editMenu.addItem(MenuAction("붙여넣기") { println("붙여넣기") })
        
        // 도구 메뉴
        val toolsMenu = Menu("도구")
        val settingsMenu = Menu("설정")
        settingsMenu.addItem(MenuAction("일반 설정") { println("일반 설정") })
        settingsMenu.addItem(MenuAction("고급 설정") { println("고급 설정") })
        toolsMenu.addItem(settingsMenu)
        toolsMenu.addItem(MenuAction("도구 실행") { println("도구 실행") })
        
        // 루트 메뉴에 추가
        rootMenu.addItem(fileMenu)
        rootMenu.addItem(editMenu)
        rootMenu.addItem(toolsMenu)
    }
    
    fun displayMenu() {
        rootMenu.display()
    }
    
    fun executeMenu() {
        rootMenu.execute()
    }
}
```

### 3. RecyclerView Adapter

```kotlin
// RecyclerView 아이템 인터페이스
interface RecyclerViewItem {
    fun getViewType(): Int
    fun bind(holder: RecyclerView.ViewHolder)
    fun addItem(item: RecyclerViewItem)
    fun removeItem(item: RecyclerViewItem)
    fun getChildItem(index: Int): RecyclerViewItem?
}

// 헤더 아이템 (Leaf)
class HeaderItem(private val title: String) : RecyclerViewItem {
    override fun getViewType(): Int = VIEW_TYPE_HEADER
    
    override fun bind(holder: RecyclerView.ViewHolder) {
        if (holder is HeaderViewHolder) {
            holder.bind(title)
        }
    }
    
    override fun addItem(item: RecyclerViewItem) {
        throw UnsupportedOperationException("HeaderItem은 자식을 가질 수 없습니다")
    }
    
    override fun removeItem(item: RecyclerViewItem) {
        throw UnsupportedOperationException("HeaderItem은 자식을 가질 수 없습니다")
    }
    
    override fun getChildItem(index: Int): RecyclerViewItem? {
        throw UnsupportedOperationException("HeaderItem은 자식을 가질 수 없습니다")
    }
}

// 콘텐츠 아이템 (Leaf)
class ContentItem(private val data: String) : RecyclerViewItem {
    override fun getViewType(): Int = VIEW_TYPE_CONTENT
    
    override fun bind(holder: RecyclerView.ViewHolder) {
        if (holder is ContentViewHolder) {
            holder.bind(data)
        }
    }
    
    override fun addItem(item: RecyclerViewItem) {
        throw UnsupportedOperationException("ContentItem은 자식을 가질 수 없습니다")
    }
    
    override fun removeItem(item: RecyclerViewItem) {
        throw UnsupportedOperationException("ContentItem은 자식을 가질 수 없습니다")
    }
    
    override fun getChildItem(index: Int): RecyclerViewItem? {
        throw UnsupportedOperationException("ContentItem은 자식을 가질 수 없습니다")
    }
}

// 섹션 아이템 (Composite)
class SectionItem(private val sectionTitle: String) : RecyclerViewItem {
    private val items = mutableListOf<RecyclerViewItem>()
    
    override fun getViewType(): Int = VIEW_TYPE_SECTION
    
    override fun bind(holder: RecyclerView.ViewHolder) {
        if (holder is SectionViewHolder) {
            holder.bind(sectionTitle)
        }
    }
    
    override fun addItem(item: RecyclerViewItem) {
        items.add(item)
    }
    
    override fun removeItem(item: RecyclerViewItem) {
        items.remove(item)
    }
    
    override fun getChildItem(index: Int): RecyclerViewItem? {
        return if (index in items.indices) items[index] else null
    }
    
    fun getItems(): List<RecyclerViewItem> = items.toList()
}

// 복합 RecyclerView Adapter
class CompositeRecyclerViewAdapter : RecyclerView.Adapter<RecyclerView.ViewHolder>() {
    private val items = mutableListOf<RecyclerViewItem>()
    
    fun addItem(item: RecyclerViewItem) {
        items.add(item)
        notifyDataSetChanged()
    }
    
    fun addSection(section: SectionItem) {
        items.add(section)
        // 섹션의 모든 아이템도 추가
        section.getItems().forEach { item ->
            items.add(item)
        }
        notifyDataSetChanged()
    }
    
    override fun getItemCount(): Int = items.size
    
    override fun getItemViewType(position: Int): Int {
        return items[position].getViewType()
    }
    
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        return when (viewType) {
            VIEW_TYPE_HEADER -> HeaderViewHolder.create(parent)
            VIEW_TYPE_CONTENT -> ContentViewHolder.create(parent)
            VIEW_TYPE_SECTION -> SectionViewHolder.create(parent)
            else -> throw IllegalArgumentException("Unknown view type: $viewType")
        }
    }
    
    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        items[position].bind(holder)
    }
    
    companion object {
        const val VIEW_TYPE_HEADER = 0
        const val VIEW_TYPE_CONTENT = 1
        const val VIEW_TYPE_SECTION = 2
    }
}
```

### 4. 복잡한 실제 시나리오: 파일 탐색기

```kotlin
// 파일 시스템 컴포넌트
interface FileSystemComponent {
    fun getName(): String
    fun getSize(): Long
    fun display(indent: String = "")
    fun addComponent(component: FileSystemComponent)
    fun removeComponent(component: FileSystemComponent)
    fun getChildComponent(index: Int): FileSystemComponent?
    fun search(name: String): List<FileSystemComponent>
}

// 파일 (Leaf)
class File(private val name: String, private val size: Long) : FileSystemComponent {
    override fun getName(): String = name
    
    override fun getSize(): Long = size
    
    override fun display(indent: String) {
        println("$indent📄 $name (${size} bytes)")
    }
    
    override fun addComponent(component: FileSystemComponent) {
        throw UnsupportedOperationException("파일은 자식을 가질 수 없습니다")
    }
    
    override fun removeComponent(component: FileSystemComponent) {
        throw UnsupportedOperationException("파일은 자식을 가질 수 없습니다")
    }
    
    override fun getChildComponent(index: Int): FileSystemComponent? {
        throw UnsupportedOperationException("파일은 자식을 가질 수 없습니다")
    }
    
    override fun search(name: String): List<FileSystemComponent> {
        return if (this.name.contains(name, ignoreCase = true)) {
            listOf(this)
        } else {
            emptyList()
        }
    }
}

// 디렉토리 (Composite)
class Directory(private val name: String) : FileSystemComponent {
    private val components = mutableListOf<FileSystemComponent>()
    
    override fun getName(): String = name
    
    override fun getSize(): Long {
        return components.sumOf { it.getSize() }
    }
    
    override fun display(indent: String) {
        println("$indent📁 $name (${getSize()} bytes)")
        components.forEach { component ->
            component.display("$indent  ")
        }
    }
    
    override fun addComponent(component: FileSystemComponent) {
        components.add(component)
    }
    
    override fun removeComponent(component: FileSystemComponent) {
        components.remove(component)
    }
    
    override fun getChildComponent(index: Int): FileSystemComponent? {
        return if (index in components.indices) components[index] else null
    }
    
    override fun search(name: String): List<FileSystemComponent> {
        val results = mutableListOf<FileSystemComponent>()
        
        // 자신이 매칭되는지 확인
        if (this.name.contains(name, ignoreCase = true)) {
            results.add(this)
        }
        
        // 자식들에서 검색
        components.forEach { component ->
            results.addAll(component.search(name))
        }
        
        return results
    }
}

// 파일 탐색기
class FileExplorer {
    private val root = Directory("Root")
    
    fun setupFileSystem() {
        val documents = Directory("Documents")
        documents.addComponent(File("report.pdf", 1024000))
        documents.addComponent(File("presentation.pptx", 2048000))
        
        val images = Directory("Images")
        images.addComponent(File("photo1.jpg", 512000))
        images.addComponent(File("photo2.jpg", 768000))
        
        val projects = Directory("Projects")
        val androidProject = Directory("AndroidApp")
        androidProject.addComponent(File("MainActivity.kt", 5000))
        androidProject.addComponent(File("build.gradle", 2000))
        projects.addComponent(androidProject)
        
        root.addComponent(documents)
        root.addComponent(images)
        root.addComponent(projects)
    }
    
    fun displayFileSystem() {
        println("파일 시스템 구조:")
        root.display()
    }
    
    fun searchFiles(query: String) {
        println("'$query' 검색 결과:")
        val results = root.search(query)
        results.forEach { component ->
            println("- ${component.getName()}")
        }
    }
    
    fun getTotalSize(): Long {
        return root.getSize()
    }
}

// 사용 예시
fun main() {
    val explorer = FileExplorer()
    explorer.setupFileSystem()
    
    explorer.displayFileSystem()
    println("\n총 크기: ${explorer.getTotalSize()} bytes")
    
    explorer.searchFiles("photo")
    explorer.searchFiles("Android")
}
```

---

## 4. 실제 안드로이드 앱에서의 활용

### 1. UI 컴포넌트 시스템

```kotlin
// UI 컴포넌트 인터페이스
interface UIComponent {
    fun render()
    fun addComponent(component: UIComponent)
    fun removeComponent(component: UIComponent)
    fun getChildComponent(index: Int): UIComponent?
}

// 컨테이너 (Composite)
class Container : UIComponent {
    private val components = mutableListOf<UIComponent>()
    
    override fun render() {
        println("컨테이너 렌더링")
        components.forEach { component ->
            component.render()
        }
    }
    
    override fun addComponent(component: UIComponent) {
        components.add(component)
    }
    
    override fun removeComponent(component: UIComponent) {
        components.remove(component)
    }
    
    override fun getChildComponent(index: Int): UIComponent? {
        return if (index in components.indices) components[index] else null
    }
}

// 버튼 (Leaf)
class Button(private val text: String) : UIComponent {
    override fun render() {
        println("버튼 렌더링: $text")
    }
    
    override fun addComponent(component: UIComponent) {
        throw UnsupportedOperationException("버튼은 자식을 가질 수 없습니다")
    }
    
    override fun removeComponent(component: UIComponent) {
        throw UnsupportedOperationException("버튼은 자식을 가질 수 없습니다")
    }
    
    override fun getChildComponent(index: Int): UIComponent? {
        throw UnsupportedOperationException("버튼은 자식을 가질 수 없습니다")
    }
}
```

### 2. 데이터 구조 관리

```kotlin
// 데이터 노드 인터페이스
interface DataNode {
    fun getValue(): Any
    fun addChild(child: DataNode)
    fun removeChild(child: DataNode)
    fun getChild(index: Int): DataNode?
    fun traverse(visitor: (DataNode) -> Unit)
}

// 데이터 노드 (Leaf)
class DataLeaf(private val value: Any) : DataNode {
    override fun getValue(): Any = value
    
    override fun addChild(child: DataNode) {
        throw UnsupportedOperationException("DataLeaf는 자식을 가질 수 없습니다")
    }
    
    override fun removeChild(child: DataNode) {
        throw UnsupportedOperationException("DataLeaf는 자식을 가질 수 없습니다")
    }
    
    override fun getChild(index: Int): DataNode? {
        throw UnsupportedOperationException("DataLeaf는 자식을 가질 수 없습니다")
    }
    
    override fun traverse(visitor: (DataNode) -> Unit) {
        visitor(this)
    }
}

// 데이터 컨테이너 (Composite)
class DataContainer(private val name: String) : DataNode {
    private val children = mutableListOf<DataNode>()
    
    override fun getValue(): Any = name
    
    override fun addChild(child: DataNode) {
        children.add(child)
    }
    
    override fun removeChild(child: DataNode) {
        children.remove(child)
    }
    
    override fun getChild(index: Int): DataNode? {
        return if (index in children.indices) children[index] else null
    }
    
    override fun traverse(visitor: (DataNode) -> Unit) {
        visitor(this)
        children.forEach { child ->
            child.traverse(visitor)
        }
    }
}
```


## 참고 자료
[Composite Pattern - Refactoring Guru](https://refactoring.guru/design-patterns/composite)    
[Kotlin Design Patterns](https://github.com/dbacinski/Design-Patterns-In-Kotlin) 
