# Tutorial

## 1. Composable functions

Jetpack Compose 是围绕可组合函数构建的。这些函数可让您以程序化方式定义应用的界面，只需描述应用界面的外观并提供数据依赖项，而不必关注界面的构建过程（初始化元素、将其附加到父项等）。如需创建可组合函数，只需将 `@Composable` 注解添加到函数名称中即可。



**Add a text element**

可以通过定义内容块并调用 [`Text`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#Text(kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Color,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.font.FontStyle,androidx.compose.ui.text.font.FontWeight,androidx.compose.ui.text.font.FontFamily,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextDecoration,androidx.compose.ui.text.style.TextAlign,androidx.compose.ui.unit.TextUnit,androidx.compose.ui.text.style.TextOverflow,kotlin.Boolean,kotlin.Int,kotlin.Function1,androidx.compose.ui.text.TextStyle)) 可组合函数来实现。`setContent` 块定义了 activity 的布局，我们会在其中调用可组合函数。可组合函数只能从其他可组合函数调用。

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            Text("Hello world!")
        }
    }
}
```



**Define a composable function**

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MessageCard("Android")
        }
    }
}

@Composable
fun MessageCard(name: String) {
    Text(text = "Hello $name!")
}
```



**Preview function in Andorid Studio**

借助 `@Preview` 注解，可以在 Android Studio 中预览可组合函数，而无需构建应用并将其安装到 Android 设备或模拟器中。该注解必须用于不接受参数的可组合函数。因此，您无法直接预览 `MessageCard` 函数，而是需要创建另一个名为 `PreviewMessageCard` 的函数，由该函数使用适当的参数调用 `MessageCard`。请在 `@Composable` 上方添加 `@Preview` 注解。

```kotlin
@Composable
fun MessageCard(name: String) {
    Text(text = "Hello $name!")
}

@Preview
@Composable
fun PreviewMessageCard() {
    MessageCard("Android")
}
```



## 2. Layouts

界面多层次结构

**Add multiple texts**

构建一个简单的消息界面，界面上显示可以展开且具有动画效果的消息列表。通过显示消息发送者和消息内容，使消息可组合项更丰富。您需要先将可组合参数更改为接受 `Message` 对象（而非 `String`），然后在 `MessageCard` 可组合项中添加另一个 `Text` 可组合项。

```kotlin
// ...

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MessageCard(Message("Android", "Jetpack Compose"))
        }
    }
}

data class Message(val author: String, val body: String)

@Composable
fun MessageCard(msg: Message) {
    Text(text = msg.author)
    Text(text = msg.body)
}

@Preview
@Composable
fun PreviewMessageCard() {
    MessageCard(
        msg = Message("Colleague", "Hey, take a look at Jetpack Compose, it's great!")
    )
}
```

在内容视图中创建两个文本元素。由于未提供有关如何排列这两个文本元素的信息，因此它们会相互重叠。



**Using a Clolumn**

[`Column`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary?hl=zh-cn#Column(androidx.compose.ui.Modifier,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,kotlin.Function1)(androidx.compose.ui.Modifier,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,kotlin.Function1)) 函数可让您垂直排列元素。向 `MessageCard` 函数中添加一个 `Column`。
您可以使用 [`Row`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary?hl=zh-cn#Row(androidx.compose.ui.Modifier,androidx.compose.foundation.layout.Arrangement.Horizontal,androidx.compose.ui.Alignment.Vertical,kotlin.Function1)(androidx.compose.ui.Modifier,androidx.compose.foundation.layout.Arrangement.Horizontal,androidx.compose.ui.Alignment.Vertical,kotlin.Function1)) 水平排列各项，并使用 [`Box`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/layout/package-summary?hl=zh-cn#Box(androidx.compose.ui.Modifier)) 堆叠元素。

```kotlin
@Composable
fun MessageCard(msg: Message) {
    Column {
        Text(text = msg.author)
        Text(text = msg.body)
    }
}

```



**Add an image element**

添加消息发送者的个人资料照片。使用 [Resource Manager](https://developer.android.com/studio/write/resource-manager?hl=zh-cn#import) 从照片库中导入图片，或使用[这张图片](https://developer.android.com/static/images/jetpack/compose-tutorial/profile_picture.png?hl=zh-cn)。添加一个 `Row` 可组合项，以实现良好的设计结构，并向该可组合项中添加一个 [`Image`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/package-summary?hl=zh-cn#Image(androidx.compose.ui.graphics.painter.Painter,kotlin.String,androidx.compose.ui.Modifier,androidx.compose.ui.Alignment,androidx.compose.ui.layout.ContentScale,kotlin.Float,androidx.compose.ui.graphics.ColorFilter)) 可组合项。

```kotlin
@Composable
fun MessageCard(msg: Message) {
    Row {
        Image(
            painter = painterResource(R.drawable.profile_picture),
            contentDescription = "Contact profile picture",
        )
    
       Column {
            Text(text = msg.author)
            Text(text = msg.body)
        }
  
    }
  
}
```



**Configure layout**

为了装饰或配置可组合项，Compose 使用了**修饰符**。通过修饰符，可以更改可组合项的大小、布局、外观，还可以添加高级互动，例如使元素可点击。可以将这些修饰符链接起来，以创建更丰富的可组合项。

```kotlin
@Composable
fun MessageCard(msg: Message) {
    // Add padding around our message
    Row(modifier = Modifier.padding(all = 8.dp)) {
        Image(
            painter = painterResource(R.drawable.profile_picture),
            contentDescription = "Contact profile picture",
            modifier = Modifier
                // Set image size to 40 dp
                .size(40.dp)
                // Clip image to be shaped as a circle
                .clip(CircleShape)
        )

        // Add a horizontal space between the image and the column
        Spacer(modifier = Modifier.width(8.dp))

        Column {
            Text(text = msg.author)
            // Add a vertical space between the author and message texts
            Spacer(modifier = Modifier.height(4.dp))
            Text(text = msg.body)
        }
    }
}
  
```



## 3. Material Design 3

**Use Material Design**

Jetpack Compose 原生提供 Material Design 及其界面元素的实现。

使用在您的项目中创建的 Material 主题 `ComposeTutorialTheme` 和 `Surface` 来封装 `MessageCard` 函数。 在 `@Preview` 和 `setContent` 函数中都需要执行此操作。这样一来，可组合项即可沿用应用主题中定义的样式，从而在整个应用中确保一致性。

Material Design 是围绕 `Color`、`Typography`、`Shape` 这三大要素构建的。您将逐一添加这些要素。

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            ComposeTutorialTheme {
                Surface(modifier = Modifier.fillMaxSize()) {
                    MessageCard(Message("Android", "Jetpack Compose"))
                }
            }
        }
    }
}

@Preview
@Composable
fun PreviewMessageCard() {
    ComposeTutorialTheme {
        Surface {
            MessageCard(
                msg = Message("Colleague", "Take a look at Jetpack Compose, it's great!")
            )
        }
    }
}
```



**Color**

通过 `MaterialTheme.colors`，使用已封装主题中的颜色来设置样式。您可以在需要颜色的任意位置使用主题中的这些值。设置标题样式，并为图片添加边框：

```kotlin
@Composable
fun MessageCard(msg: Message) {
   Row(modifier = Modifier.padding(all = 8.dp)) {
       Image(
           painter = painterResource(R.drawable.profile_picture),
           contentDescription = null,
           modifier = Modifier
               .size(40.dp)
               .clip(CircleShape)
               .border(1.5.dp, MaterialTheme.colors.secondary, CircleShape)
       )

       Spacer(modifier = Modifier.width(8.dp))

       Column {
           Text(
               text = msg.author,
               color = MaterialTheme.colors.secondaryVariant
           )

           Spacer(modifier = Modifier.height(4.dp))
           Text(
               text = msg.body,
               style = MaterialTheme.typography.body2
           )
       }
   }
}
```



**Typography**

`MaterialTheme` 中提供了 Material Typography 样式，只需将其添加到 `Text` 可组合项中即可。



**Shape**

将消息正文封装在 [`Surface`](https://developer.android.com/reference/kotlin/androidx/compose/material/package-summary?hl=zh-cn#Surface(androidx.compose.ui.Modifier,androidx.compose.ui.graphics.Shape,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.foundation.BorderStroke,androidx.compose.ui.unit.Dp,kotlin.Function0)) 可组合项中。这样即可自定义消息正文的形状和高度。此外，还要为消息添加内边距，以改进布局。

```kotlin
Surface(shape = MaterialTheme.shapes.medium, elevation = 1.dp) {
       Text(
           text = msg.body,
           modifier = Modifier.padding(all = 4.dp),
           style = MaterialTheme.typography.body2
       )
}
```



**Enable dark theme**

启用[深色主题](https://developer.android.com/guide/topics/ui/look-and-feel/darktheme?hl=zh-cn)（或夜间模式），以避免显示屏过亮（尤其是在夜间），或者只是节省设备电量。由于支持 Material Design，Jetpack Compose 默认能够处理深色主题。使用 Material Design 颜色、文本和背景时，系统会自动适应深色背景。

可以在文件中以单独函数的形式创建多个预览，也可以向同一个函数中添加多个注解。

添加新的预览注解并启用夜间模式。

```kotlin
@Preview(name = "Light Mode")
@Preview(
    uiMode = Configuration.UI_MODE_NIGHT_YES,
    showBackground = true,
    name = "Dark Mode"
)
@Composable
fun PreviewMessageCard() {
   ComposeTutorialTheme {
    Surface {
      MessageCard(
        msg = Message("Colleague", "Hey, take a look at Jetpack Compose, it's great!")
      )
    }
   }
}
```



## 4. Lists and animations

**Create a list of messages**

创建一个可显示多条消息的 `Conversation` 函数。使用 Compose 的 [`LazyColumn`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/package-summary?hl=zh-cn#LazyColumn(androidx.compose.ui.Modifier,androidx.compose.foundation.lazy.LazyListState,androidx.compose.foundation.layout.PaddingValues,kotlin.Boolean,androidx.compose.foundation.layout.Arrangement.Vertical,androidx.compose.ui.Alignment.Horizontal,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean,kotlin.Function1)) 和 [`LazyRow`](https://developer.android.com/reference/kotlin/androidx/compose/foundation/lazy/package-summary?hl=zh-cn#LazyRow(androidx.compose.ui.Modifier,androidx.compose.foundation.lazy.LazyListState,androidx.compose.foundation.layout.PaddingValues,kotlin.Boolean,androidx.compose.foundation.layout.Arrangement.Horizontal,androidx.compose.ui.Alignment.Vertical,androidx.compose.foundation.gestures.FlingBehavior,kotlin.Boolean,kotlin.Function1))。这些可组合项只会呈现屏幕上显示的元素，因此，对于较长的列表，使用它们会非常高效。

在此代码段中，可以看到 `LazyColumn` 包含一个 `items` 子项。它接受 `List` 作为参数，并且其 lambda 会收到我们命名为 `message` 的参数（可以随意为其命名），该参数是 `Message` 的实例。简而言之，系统会针对提供的 `List` 的每个项调用此 lambda。

```kotlin
@Composable
fun Conversation(messages: List<Message>) {
    LazyColumn {
        items(messages) { message ->
            MessageCard(message)
        }
    }
}

@Preview
@Composable
fun PreviewConversation() {
    ComposeTutorialTheme {
        Conversation(SampleData.conversationSample)
    }
}
```



**Animate messages while expanding**

添加展开消息以显示更多内容的功能，同时为内容大小和背景颜色添加动画效果。为了存储此本地界面状态，需要跟踪消息是否已展开。为了跟踪这种状态变化，必须使用 `remember` 和 `mutableStateOf` 函数。

可组合函数可以使用 `remember` 将本地状态存储在内存中，并跟踪传递给 `mutableStateOf` 的值的变化。该值更新时，系统会自动重新绘制使用此状态的可组合项（及其子项）。这称为[重组](https://developer.android.com/jetpack/compose/mental-model?hl=zh-cn#recomposition)。

通过使用 Compose 的状态 API（如 `remember` 和 `mutableStateOf`），系统会在状态发生任何变化时自动更新界面。

```kotlin
class MainActivity : ComponentActivity() {
   override fun onCreate(savedInstanceState: Bundle?) {
       super.onCreate(savedInstanceState)
       setContent {
           ComposeTutorialTheme {
               Conversation(SampleData.conversationSample)
           }
       }
   }
}

@Composable
fun MessageCard(msg: Message) {
    Row(modifier = Modifier.padding(all = 8.dp)) {
        Image(
            painter = painterResource(R.drawable.profile_picture),
            contentDescription = null,
            modifier = Modifier
                .size(40.dp)
                .clip(CircleShape)
                .border(1.5.dp, MaterialTheme.colors.secondaryVariant, CircleShape)
        )
        Spacer(modifier = Modifier.width(8.dp))

        // We keep track if the message is expanded or not in this
        // variable
        var isExpanded by remember { mutableStateOf(false) }

        // We toggle the isExpanded variable when we click on this Column
        Column(modifier = Modifier.clickable { isExpanded = !isExpanded }) {
            Text(
                text = msg.author,
                color = MaterialTheme.colors.secondaryVariant,
                style = MaterialTheme.typography.subtitle2
            )

            Spacer(modifier = Modifier.height(4.dp))

            Surface(
                shape = MaterialTheme.shapes.medium,
                elevation = 1.dp,
            ) {
                Text(
                    text = msg.body,
                    modifier = Modifier.padding(all = 4.dp),
                    // If the message is expanded, we display all its content
                    // otherwise we only display the first line
                    maxLines = if (isExpanded) Int.MAX_VALUE else 1,
                    style = MaterialTheme.typography.body2
                )
            }
        }
    }
}
```

现在，可以根据点击消息时消息的 `isExpanded` 状态，更改消息内容的背景颜色。您将使用 `clickable` 修饰符来处理可组合项上的点击事件。为背景颜色添加动画效果，使其值逐步从 `MaterialTheme.colors.surface` 更改为 `MaterialTheme.colors.primary`（反之亦然），而不只是切换 `Surface` 的背景颜色。为此，您将使用 `animateColorAsState` 函数。最后，您将使用 `animateContentSize` 修饰符顺畅地为消息容器大小添加动画效果：

```kotlin
@Composable
fun MessageCard(msg: Message) {
    Row(modifier = Modifier.padding(all = 8.dp)) {
        Image(
            painter = painterResource(R.drawable.profile_picture),
            contentDescription = null,
            modifier = Modifier
                .size(40.dp)
                .clip(CircleShape)
                .border(1.5.dp, MaterialTheme.colors.secondaryVariant, CircleShape)
        )
        Spacer(modifier = Modifier.width(8.dp))

        // We keep track if the message is expanded or not in this
        // variable
        var isExpanded by remember { mutableStateOf(false) }
        // surfaceColor will be updated gradually from one color to the other
        val surfaceColor by animateColorAsState(
            if (isExpanded) MaterialTheme.colors.primary else MaterialTheme.colors.surface,
        )

        // We toggle the isExpanded variable when we click on this Column
        Column(modifier = Modifier.clickable { isExpanded = !isExpanded }) {
            Text(
                text = msg.author,
                color = MaterialTheme.colors.secondaryVariant,
                style = MaterialTheme.typography.subtitle2
            )

            Spacer(modifier = Modifier.height(4.dp))

            Surface(
                shape = MaterialTheme.shapes.medium,
                elevation = 1.dp,
                // surfaceColor color will be changing gradually from primary to surface
                color = surfaceColor,
                // animateContentSize will change the Surface size gradually
                modifier = Modifier.animateContentSize().padding(1.dp)
            ) {
                Text(
                    text = msg.body,
                    modifier = Modifier.padding(all = 4.dp),
                    // If the message is expanded, we display all its content
                    // otherwise we only display the first line
                    maxLines = if (isExpanded) Int.MAX_VALUE else 1,
                    style = MaterialTheme.typography.body2
                )
            }
        }
    }
}
```



