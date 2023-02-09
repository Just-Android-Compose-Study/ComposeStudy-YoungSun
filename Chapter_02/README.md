# 2장 정리
## 안드로이드 뷰 시스템 살펴보기

안드로이드 뷰 기반의 앱에서는 XML 언어를 사용하여 레이아웃 파일을 정의한다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout 
		xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/message"
        style="@style/TextAppearance.AppCompat.Medium"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textAlignment="center"
        ... />

    <EditText
        android:id="@+id/name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginEnd="8dp"
        android:hint="@string/hint"
        android:imeOptions="actionDone"
        android:importantForAutofill="no"
        android:inputType="textPersonName"
        android:minEms="8"
        ... />

    <Button
        android:id="@+id/done"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/done"
        ... />

</androidx.constraintlayout.widget.ConstraintLayout>
```

→ 다음과 같은 XML 레이아웃 코드는 **부모(root) 노드**(ConstraintLayout)와 **자식 노드**(TextView, EditText, Button)의 **계층 형태**로 이루어지며, 

View가 더 많아진다면 훨씬 더 **중첩적인 구조**를 가질 수 있다. (계층이 많아지는 ⇒ depth 가 높아짐)

## 레이아웃 파일 인플레이팅

Android View 시스템의 Activity에서는 일반적으로 (뷰 바인딩, 데이터 바인딩을 사용하지 않고) 

setContentView()를 사용하여 레이아웃의 id 값을 전달한다.

```kotlin
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.main)
	      ...
    }
}
```

setContentView 메서드에 `R.layout.main` id값을 전달한 코드이다.

```kotlin
private lateinit var doneButton : Button
...
doneButton = findViewById(R.id.done)
```

또한, 레이아웃 파일에 있는 View 컴포넌트를 호출하여 사용하기 위해서는 

다음과 같이 findViewById()를 사용하여 View의 id 값을 받아와 정의해주어야 한다.

이와 같은 방식은 더 큰 규모의 앱과는 더욱 잘 맞지 않는다.

- View 컴포넌트를 호출하기 위해서는 미리 초기화하여 호출해야 하고, 초기화하지 않고 호출하였을 때는 런타임에서 크래시가 발생할 수 있다.
- 또한 컴포넌트의 개수가 많아지면, 코드는 급격히 비대해진다.

## 컴포넌트 계층 구조

- XML 파일에 있는 태그는 클래스 명을 나타낸다. 모든 안드로이드의 뷰는 클래스이다.
    - 각 뷰의 속성들은 해당 뷰 클래스의 멤버에 대응된다.
    - inflate()는 객체 트리를 생성한다.
    - 객체 트리를 수정해서 UI 구조를 변경할 수 있다.
    
- 안드로이드 UI의 최상위 요소는 android.view.View이다.

## 컴포넌트 계층 구조의 한계

- 안드로이드의 Button은 일반적으로 텍스트를 포함하며, android.widget.TextView를 확장(상속)한다.
- 이미지를 보여주는 버튼이 필요하다면,  ImageButton이 필요하며, android.widget.ImageView를 확장(상속)한다.
- 컴포넌트의 클래스들은 자바의 클래스를 사용하는데, 자바의 클래스는 다중 상속이 지원이 안될뿐 더러,
    - 다중 상속이 지원이 된다 하더라도, 특정 컴포넌트의 개별 기능을 분리할 수 없으므로, 다른 컴포넌트의 개별 기능을 조합할 수 없다.
    - 이러한 개별 기능을 분리하기 위해서는 컴포넌트의 개념에서 벗어나야 한다.
- Flutter에서는 이러한 개별 기능들을 속성으로 정의하여 간단한 구성 요소끼리 조합하는 방식을 사용한다.
    - **Composition Over inheritance (상속보다는 조합 방식을 사용)**
    - 젯팩 컴포즈에서는 컴포저블 함수를 사용하여 구현

## 함수를 통한 UI 구성

- 젯팩 컴포즈에서 **UI의 진입점은 컴포저블 함수**이며, **컴포저블 함수**에서 **다른 컴포저블 함수가 호출**된다.
- 컴포저블 함수는 주로 content라는 매개변수를 전달받는데, 이 content는 다른 컴포저블 함수이다.
- 컴포저블 함수는 항상 실제 값으로 호출된다.
    - 그리고, 뷰를 의도적으로 변경하기 위해서, 프로퍼티 값을 변경하게 되는데, 이 때 변경할 수 있는 값을 **상태(State)**라고 한다.
    - `mutableStateOf()`를 사용하면 상태를 생성할 수 있다.
    - 이러한 상태를 참조하는 컴포저블 함수는 이 **상태가 변경됨에 따라** 뷰는 **재구성**된다. (recomposition)
        
        ```kotlin
        @Composable
        fun Factorial() {
            var expanded by remember { mutableStateOf(false) }
            var text by remember { mutableStateOf(factorialAsString(0)) }
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                Text(
                    modifier = Modifier.clickable {
                        expanded = true
                    },
                    text = text,
                    style = MaterialTheme.typography.h2
                )
                DropdownMenu(
                    expanded = expanded,
                    onDismissRequest = {
                        expanded = false
                    }) {
                    for (n in 0 until 10) {
                        DropdownMenuItem(onClick = {
                            expanded = false
                            text = factorialAsString(n)
                        }) {
                            Text("${n.toString()}!")
                        }
                    }
                }
            }
        }
        ```
        
        - expanded 와 text는 모두 **상태**이다.
        - expanded 의 값이 변경됨에 따라 DropDownMenu를 다시 그린다.
        - text 의 값이 변경됨에 따라 Text()를 다시 그린다.

## 클릭 동작에 반응

- **Button**과 같은 특정 컴포넌트를 그리는 컴포저블 함수같은 경우에는 **클릭 이벤트 처리를 하는 전용 onClick** 매개변수를 갖는다.
- **Text**와 같은 컴포저블 함수는 **modifier 매개변수에 clickable{ … } 값을 전달**하여 클릭 이벤트 처리를 구현할 수 있다.

## UI 요소 크기 조절과 배치

- 컴포넌트 중심의 UI 프레임워크에서는 크기와 위치를 화면에 나타내는 것이 핵심이다.
- **RelativeLayout** - 특정 View의 상대적인 위치에 배치한다.
    - toStartOf, toEndOf
- **FrameLayout** - 스택에 자식들을 배치한다.
    - 젯팩 컴포즈 → **Box**()
    - 코드로 **작성한 순서에 따라 컴포넌트를 배치**한다.a
    - **먼저 작성된 컴포넌트가 제일 아래에 깔려서 보인다.**
- **LinearLayout** - 수평 또는 수직으로 자식 컴포넌트를 배치시킨다.
    - 젯팩 컴포즈 → **Row**() , **Column**()
    
- Modifier(변경자)는 **크기를 요청**하거나 **위치를 재정의**하는데에 사용될 수 있다.
    - **modifier = Modifier.align(** Alignment.XXX **) ⇒ 위치 재정의**
    - modifier = Modifier.fillMaxSize() ⇒ 가능한 한 크게
    - modifier = Modifier.size( width = xx.dp, height = xx.dp )
- background() 변경자를 통해 컴포저블 함수의 **배경색을 설정**할 수 있다.
    - modifier.background(Color.xxx)
