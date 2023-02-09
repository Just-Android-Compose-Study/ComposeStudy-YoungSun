# 1장 정리
## 환영 인사 나타내기

```kotlin
@Composable
fun Welcome() {
    Text(
        text = stringResource(id = R.string.welcome),
        style = MaterialTheme.typography.subtitle1
    )
}
```

- 컴포저블 함수는 @Composable이라는 어노테이션을 사용하여 식별
- UI 요소를 반환
- `Text()` 는 텍스트를 나타내는 컴포저블 함수
    - **text**는 텍스트 문구
    - **style**은 텍스트의 스타일
    - **textAlign**은 텍스트의 가로로 어떻게 배치할지

## 열, 텍스트 필드, 버튼 사용

```kotlin
@Composable
fun TextAndButton(name: MutableState<String>, nameEntered: MutableState<Boolean>) {
    Row(modifier = Modifier.padding(top = 8.dp)) {

        TextField(
            value = name.value,
            onValueChange = {
                name.value = it
            },
            placeholder = {
                Text(text = stringResource(id = R.string.hint))
            },
            modifier = Modifier
                .alignByBaseline()
                .weight(1.0F),
            singleLine = true,
            keyboardOptions = KeyboardOptions(
                autoCorrect = false,
                capitalization = KeyboardCapitalization.Words,
            ),
            keyboardActions = KeyboardActions(onAny = {
                nameEntered.value = true
            })
        ) // TextField

        Button(modifier = Modifier
            .alignByBaseline()
            .padding(8.dp),
            onClick = {
                nameEntered.value = true
            }) {
            Text(text = stringResource(id = R.string.done))
        }
    } // Button
}
```

- `TextField()` 는 텍스트를 입력받는 컴포저블
    - Android View의 EditText와 동일
    - placeholder은 EditText의 hint와 동일.
    - value는 현재 입력된 텍스트를 나타내며, onValueChange{ }는 텍스트가 입력되고 지워지면서 값이 바뀜에 따라 안에 있는 코드가 동작한다.
    - singleLine은 사용자가 여러 줄을 입력할 수 있는 지를 나타낸다.
    - keyboardActions는 IME action에 대한 할 일을 명시하며, nameEntered의 값이 true로 변경된다.
        - Done(완료)버튼과 같은 동작.
- `Button()` 컴포저블은 버튼을 나타내는 컴포저블
    - { } 내부는 Row처럼 동작
    - onClick은 버튼이 눌렸을 때, 할 일을 나타낸다.
    - Android View의 Button과는 다르게, clickListener만 다루고, Text는 따로 그려주어야 한다.
    - [https://aroundck.tistory.com/8190](https://aroundck.tistory.com/8190)
- MutableState 객체는 변경할 수 있는 값을 뜻한다.

## 인사말 출력

```kotlin
@Composable
fun Hello() {
    val name = remember { mutableStateOf("") }
    val nameEntered = remember { mutableStateOf(false) }
    Box(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp),
        contentAlignment = Alignment.Center
    ) {
        if (nameEntered.value) {
            Greeting(name.value)
        } else {
            Column(horizontalAlignment = Alignment.CenterHorizontally) {
                Welcome()
                TextAndButton(name, nameEntered)
            }
        }
    }
}
```

- nameEntered의 값은 TextAndButton 함수에서 이름이 입력 완료 됨에 따라 바뀐다.
- Hello()는 stateFul 하다.
    - nameEntered의 값에 따라, 반환하는 UI 요소가 다르다.
    - mutableStateOf()를 사용해 상태를 생성하고, remember를 사용해 상태를 기억한다.
    - 이 상태는 TextAndButton() 함수에서 변경되고, 이를 상태 호이스팅(State Hoisting)이라고 한다.
- Welcome()은 stateLess 하다.

## 미리보기 설정

- Compose Preview 기능을 사용하기 위해서는 @Preview 어노테이션을 사용한다.
- @PreviewParameter를 사용하면, 컴포저블 함수에 파라미터를 전달하여 미리보기를 사용할 수 있다.
    - 다만 미리보기에만 값이 사용되는 제약이 있다.
    - background 프로퍼티를 사용하여 미리보기 배경 사용을 지정할 수 있다.
    - backgroundColor 를 통해 미리보기 배경 색상을 지정할 수 있다.
- 미리 보기 어노테이션을 사용하는 컴포저블 함수가 여러 개일때, 미리보기 화면이 다소 장황할 수 있는데, group 속성을 사용하여 미리보기를 사용할 컴포저블 함수들을 그룹화 할 수 있다.

## 액티비티에서 컴포저블 함수 사용

- 컴포즈에서는 setContent() 를 호출하는 반면, Android View 기반의 앱은 setContentView() 를 호출하여 레이아웃 아이디를 통해 루트 뷰를 전달한다는 차이점이 있다.

## 내부 살펴보기

**build.gradle**

```kotlin
buildFeatures {
	compose true
}
```

다음과 같은 플러그인이 활성화되도록 설정하고, 최소 API 레벨을 21이나 그 이상으로 설정해야 한다.
