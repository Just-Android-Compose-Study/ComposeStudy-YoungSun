## Composable 함수의 구성 요소

Compose App의 UI는 UI를 그리는 Composable이 작성된 Composable 함수를 호출하여 만들어진다.

**Composable 함수 :** @Composable 어노테이션을 포함하는 코틀린 함수.

@Composable : Composable 어노테이션은 Compose 컴파일러에게 해당 함수가 데이터를 UI 요소로 변환한다는 것을 알리는 역할을 한다. 따라서, Composable을 사용하여 UI를 그리기 위해 Composable 함수를 작성한다면, 반드시 어노테이션이 작성되어야 한다.

Composable 함수는 @Composable 어노테이션을 포함하고 코틀린 함수의 시그니처와 같은 구성요소를 가진다.

Composable 함수 명명규칙 → 명사 또는 형용사를 접두어로 갖는 명사

- 동사 또는 동사구가 될 수 없다.

## UI 요소 내보내기

### **androidx.compose.material.Text()**

```kotlin
		val textColor = color.takeOrElse {
		        style.color.takeOrElse {
		            LocalContentColor.current.copy(alpha = LocalContentAlpha.current)
		        }
		    }

    val mergedStyle = style.merge(
        TextStyle(
            color = textColor,
            fontSize = fontSize,
            fontWeight = fontWeight,
            textAlign = textAlign,
            lineHeight = lineHeight,
            fontFamily = fontFamily,
            textDecoration = textDecoration,
            fontStyle = fontStyle,
            letterSpacing = letterSpacing
        )
    )

    BasicText(
        text,
        modifier,
        mergedStyle,
        onTextLayout,
        overflow,
        softWrap,
        maxLines,
    )
```

Text()는 BasicText()를 호출하지만, Text()를 사용하는 이유는 테마의 스타일 정보를 사용하기 때문이다.

### **BasicText()**

```kotlin
		require(maxLines > 0) { "maxLines should be greater than 0" }

    val selectionRegistrar = LocalSelectionRegistrar.current
    val density = LocalDensity.current
    val fontFamilyResolver = LocalFontFamilyResolver.current

    val selectableId =
        rememberSaveable(text, selectionRegistrar, saver = selectionIdSaver(selectionRegistrar)) {
            selectionRegistrar?.nextSelectableId() ?: SelectionRegistrar.InvalidSelectableId
        }

    val controller = remember {
        TextController(
            TextState(
                TextDelegate(
                    text = AnnotatedString(text),
                    style = style,
                    density = density,
                    softWrap = softWrap,
                    fontFamilyResolver = fontFamilyResolver,
                    overflow = overflow,
                    maxLines = maxLines,
                ),
                selectableId
            )
        )
    }
    val state = controller.state
    if (!currentComposer.inserting) {
        controller.setTextDelegate(
            updateTextDelegate(
                current = state.textDelegate,
                text = text,
                style = style,
                density = density,
                softWrap = softWrap,
                fontFamilyResolver = fontFamilyResolver,
                overflow = overflow,
                maxLines = maxLines,
            )
        )
    }
    state.onTextLayout = onTextLayout
    controller.update(selectionRegistrar)
    if (selectionRegistrar != null) {
        state.selectionBackgroundColor = LocalTextSelectionColors.current.backgroundColor
    }

    Layout(modifier.then(controller.modifiers), controller.measurePolicy)
```

BasicText()는 최종적으로 Layout()이라는 컴포저블을 호출한다.

### Layout()

```kotlin
		val density = LocalDensity.current
    val layoutDirection = LocalLayoutDirection.current
    val viewConfiguration = LocalViewConfiguration.current
    ReusableComposeNode<ComposeUiNode, Applier<Any>>(
        factory = ComposeUiNode.Constructor,
        update = {
            set(measurePolicy, ComposeUiNode.SetMeasurePolicy)
            set(density, ComposeUiNode.SetDensity)
            set(layoutDirection, ComposeUiNode.SetLayoutDirection)
            set(viewConfiguration, ComposeUiNode.SetViewConfiguration)
        },
        skippableUpdate = materializerOf(modifier),
        content = content
    )
```

Layout()은 레이아웃을 위한 핵심 컴포저블 함수이고, 자식 요소의 크기와 위치를 지정한다.

ReusableComposeNode() : UI 요소인 Node를 내보낸다. (=Compose 내부 자료구조에 자식 Node를 추가하는 것)

- **factory** : Node를 생성
- **update** : Node에서 업데이트를 수행
- **skippableUpdate** : 변경자(modifier)를 조작
- **content** : 자식 Node가 되는 또 다른 Composable 함수를 포함

### ReusalbeComposeNode()

```kotlin
		if (currentComposer.applier !is E) invalidApplier()
    currentComposer.startReusableNode()
    if (currentComposer.inserting) {
        currentComposer.createNode(factory)
    } else {
        currentComposer.useNode()
    }
    currentComposer.disableReusing()
    Updater<T>(currentComposer).update()
    currentComposer.enableReusing()
    SkippableUpdater<T>(currentComposer).skippableUpdate()
    currentComposer.startReplaceableGroup(0x7ab4aae9)
    content()
    currentComposer.endReplaceableGroup()
    currentComposer.endNode()
```

ReusalbeComposeNode는 새로운 Node가 생성되어야 할지, 기존의 Node를 재사용할지 결정하는 역할을 한다.

그러고 나서 업데이트를 수행하고, 마지막으로 content()를 호출해 콘텐츠를 노드에 내보낸다.

**currentComposer :** androidx.compose.runtime.Composables에 있는 Composer타입의 최상위 변수

```kotlin
val currentComposer: Composers
    @ReadOnlyComposable
    @Composable get() { throw NotImplementedError("Implemented as an intrinsic") }
```

Layout()은 **factory인자에 ComposeUiNode.constructor를 전달**하므로, **currentComposer.createNode(factory)**에 ComposeUiNode 타입이 factory에 담겨 전달되며, 이를 통해 생성된 **UI 요소를 나타내는 노드의 기능이 ComposeUiNode 인터페이스에 정의됨**을 알 수 있다.

### ComposeUiNode

```kotlin
var measurePolicy: MeasurePolicy
var layoutDirection: LayoutDirection
var density: Density
var modifier: Modifier
var viewConfiguration: ViewConfiguration

    /**
     * Object of pre-allocated lambdas used to make use with ComposeNode allocation-less.
     */
companion object {
	val Constructor: () -> ComposeUiNode = LayoutNode.Constructor
	val SetModifier: ComposeUiNode.(Modifier) -> Unit = { this.modifier = it }
	val SetDensity: ComposeUiNode.(Density) -> Unit = { this.density = it }
	val SetMeasurePolicy: ComposeUiNode.(MeasurePolicy) -> Unit = { this.measurePolicy = it }
	val SetLayoutDirection: ComposeUiNode.(LayoutDirection) -> Unit = { this.layoutDirection = it }
	val SetViewConfiguration: ComposeUiNode.(ViewConfiguration) -> Unit = { this.viewConfiguration = it }
}
```

Node

- 컴포즈 계층 구조의 요소
- 컴포즈 내부 동작의 일부
- Node의 중요한 4가지 구성요소 (앱과 관련된 중요한 자료 구조와 개념)
    - measurePolicy
    - layoutDirection
    - density
    - modifier

## 값 반환 (값을 반환하는 컴포저블 함수)

- 컴포저블 함수의 **대부분은 반환 값이 필요가 없**기 때문에 명시하지 않는다.
    - 주 목적이 UI를 구성하는 것이기 때문 (구성 == 그리기 == 내보내기)

- 초기의 상태 값을 설정할 때 사용하는 **remember**{ }

    ```kotlin
    @Composable
    inline fun <T> remember(calculation: @DisallowComposableCalls () -> T): T =
        currentComposer.cache(false, calculation)
    ```

- **stringResource**( )

    ```kotlin
    @Composable
    @ReadOnlyComposable
    fun stringResource(@StringRes id: Int, vararg formatArgs: Any): String {
        val resources = resources()
        return resources.getString(id, *formatArgs)
    }
    ```

    - resources 역시 컴포저블 함수이다.
    - 컴포저블 함수를 호출하기 위해서는, 컴포저블 함수여야 하며, 컴포저블 함수는 @Composable 어노테이션을 반드시 포함해야 한다.
    - 반환되는 데이터가 컴포즈와 아무런 관련이 없더라도, 이 데이터는 컴포저블 함수에서 호출될 것이기 때문
    - 따라서, 구성과 재구성의 일부인 무언가를 반환해야한다면, 그 함수는 반드시 컴포저블 함수로 만들어야 한다.
    - 명명규칙 → 카멜 표기법 + 동사 또는 동사구로 구성된다.


## UI의 구성과 재구성

- 명령형 UI 프레임워크와는 달리, 젯팩 컴포즈는 선제적으로 컴포넌트 트리를 변경하는 행위에 의존하지 않는다.
- 컴포즈의 UI는 현재 앱의 데이터를 기반으로 선언되며, 데이터의 값이 변화됨에 따라 변화를 자체적으로 감지하고, **영향을 받는 부분만 갱신**한다.
- 사실, 개념상으로 컴포즈는 영향을 받은 부분만 갱신하는 것이 아니라 **값이 변경됨에 따라 UI 전체를 다시 생성**한다. 이는, 높은 처리 능력과 많은 시간, 성능을 요하며, 사용자가 인지하게 될 수도 있다.

## 컴포저블 함수 간 상태 공유

여러 개의 컴포저블 함수에서 한 개의 상태를 공유하고 싶을 수도 있다.

```kotlin
class ColorPickerDemoActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            BoxWithConstraints(
                contentAlignment = Alignment.Center,
                modifier = Modifier.fillMaxSize()
            ) {
                Column(
                    modifier = Modifier.width(min(400.dp, maxWidth)),
                    horizontalAlignment = Alignment.CenterHorizontally
                ) {
                    val color = remember { mutableStateOf(Color.Magenta) }
                    ColorPicker(color)
                    Text(
                        modifier = Modifier
                            .fillMaxWidth()
                            .background(color.value),
                        text = "#${color.value.toArgb().toUInt().toString(16)}",
                        textAlign = TextAlign.Center,
                        style = MaterialTheme.typography.h4.merge(
                            TextStyle(
                                color = color.value.complementary()
                            )
                        )
                    )
                }
            }
        }
    }
}
```

### Color.complementary()

```kotlin
fun Color.complementary() = Color(
    red = 1F - red,
    green = 1F - green,
    blue = 1F - blue
)
```

현재 color의 보색을 반환하는 함수.

### ColorPicker()

```kotlin
@Composable
fun ColorPicker(color: MutableState<Color>) {
    val red = color.value.red
    val green = color.value.green
    val blue = color.value.blue

    Column {
        Slider(
            value = red,
            onValueChange = { color.value = Color(it, green, blue) })
        Slider(
            value = green,
            onValueChange = { color.value = Color(red, it, blue) })
        Slider(
            value = blue,
            onValueChange = { color.value = Color(red, green, it) })
    }
}
```

- color 매개변수에 Color 타입의 상태 값을 인자로 전달 받는다.
- Slider는 초기 color 의 red, green, blue로 설정된다.
- Slider의 값이 바뀜에 따라(onValueChange) 상태 값을 나타내는 매개변수 color의 value 값이 변경된다.
- 색상의 변경은 ColorPicker에서 일어나므로, 값이 변경됨을 외부 호출자에게 알리려면, 함수의 매개변수 타입을 사용하는  것이 좋고, 변경 가능하게(mutable) 해야 한다.
- 컴포저블 함수의 모습과 행위에 영향을 주는 모든 데이터는 다음과 같이 매개변수로 전달하는 것이 좋다.
- 이러한, **전달받은 상태가 변경됨**에 따라 **컴포저블을 호출한 곳으로 상태를 옮기는 것**을 **상태 호이스팅**이라고 한다.

### 중요사항

- 컴포저블을 **부수 효과가 없게 만들자.**
    - 동일한 인자로 함수를 반복적으로 호출했을 때, 계속해서 동일한 결과가 나오도록
    - 호출자로부터 모든 관련 데이터를 얻는 것. (매개변수에 전달받은 인자 값에 의해서만 결정)
    - 전역 프로퍼티에 의존 X
    - 예측 불가능한 값을 반환 X
- **재구성**이 언제 얼마나, 자주 발생할 지 **예측할 수 없다.**
    - 따라서 **컴포저블을 가능한 성능이 빠르게** 만들자.
        - 시간이 소요되는 연산 (데이터를 불러오거나 저장, 네트워크 처리)을 하지말자
- **재구성의 순서는 불규칙**하다.
    - 소스코드 상으로 **더 앞에 작성된 컴포저블에 의해서 재구성**될 수 있다.
    - **특정 순서에 의존하거나 다른 곳에서 필요로 하는 무언가**를 **컴포저블 내부에서 연산하지 말아야 한다**.

### 액티비티 내에서 컴포저블 계층 구조 나타내기

- setContent를 사용해서 계층 구조를 임베디드
- **setContent**
    - parent : CompositionContext
    - content : 선언하는 UI를 위한 컴포저블 함
    - 대부분의 경우, parent를 생략.

    ```kotlin
    public fun ComponentActivity.setContent(
        parent: CompositionContext? = null,
        content: @Composable () -> Unit
    ) {
        val existingComposeView = window.decorView
            .findViewById<ViewGroup>(android.R.id.content)
            .getChildAt(0) as? ComposeView
    
        if (existingComposeView != null) with(existingComposeView) {
            setParentCompositionContext(parent)
            setContent(content)
        } else ComposeView(this).apply {
            // Set content and parent **before** setContentView
            // to have ComposeView create the composition on attach
            setParentCompositionContext(parent)
            setContent(content)
            // Set the view tree owners before setting the content view so that the inflation process
            // and attach listeners will see them already present
            setOwners()
            setContentView(this, DefaultActivityContentLayoutParams)
        }
    }
    ```

  **setParentCompositionContext()**

    ```kotlin
    fun setParentCompositionContext(parent: CompositionContext?) {
    	parentContext = parent
    }
    ```

    - parent를 생략할 경우 어떻게 되는지 ??