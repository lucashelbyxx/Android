减少样板代码、避免空指针异常、充分利用JVM


## NULL 检查机制
Kotlin的空安全设计对于声明可为空的参数，在使用时要进行空判断处理，有两种处理方式，字段后加!!像Java一样抛出空异常，另一种字段后加?可不做处理返回值为 **null** 或配合 ?: 做空判断处理
```kotlin
//类型后面加?表示可为空
var age: String? = "23" 
//抛出空指针异常
val ages = age!!.toInt()
//不做处理返回 null
val ages1 = age?.toInt()
//age为空返回-1
val ages2 = age?.toInt() ?: -1
```

## 类型检查及自动类型转换
is 类似 Java 的 instanceof
```kotlin
fun getStringLength(obj: Any): Int? {
  if (obj is String) {
    // 做过类型判断以后，obj会被系统自动转换为String类型
    return obj.length 
  }

  //在这里还有一种方法，与Java中instanceof不同，使用!is
  // if (obj !is String){
  //   // XXX
  // }

  // 这里的obj仍然是Any类型的引用
  return null
}
```

## 比较数字
Kotlin 中没有基础数据类型，只有封装的数字类型，你每定义的一个变量，其实 Kotlin 帮你封装了一个对象，这样可以保证不会出现空指针。数字类型也一样，所以在比较两个数字的时候，就有比较数据大小和比较两个对象是否相同的区别了。
在 Kotlin 中，三个等号 === 表示比较对象地址，两个 == 表示比较两个值大小。



# Control Flow

## Conditional expresssions

### If

Kotlin provides `if` and `when` for checking conditional expressions.

If you have to choose between `if` and `when`, we **recommend using `when`** as it leads to more robust and safer programs.



There is no ternary operator `condition ? then : else` in Kotlin. Instead, `if` can be used as an expression. When using `if` as an expression, there are no curly braces `{}`:

Kotlin 中没有三元运算符 `condition ? then : else` 。相反， `if` 可以用作表达式。用作 `if` 表达式时，没有大括号 `{}` ：

~~~kotlin
val a = 1
val b = 2
println(if (a > b)) a else b) // Return a value: 2
~~~



### When

Use `when` when expression with multiple branches.`when` can be used either as a statement or as an expression.

`when` as a statement:

```kotlin
val obj = "Hello"

when (obj) {
    // Checks whether obj equals to "1"
    "1" -> println("One")
    // Checks whether obj equals to "Hello"
    "Hello" -> println("Greeting")
    // Default statement
    else -> println("Unknown")     
}
// Greeting
```

`when` as an expression:

```kotlin
val temp = 18

val description = when {
    // If temp < 0 is true, sets description to "very cold"
    temp < 0 -> "very cold"
    // If temp < 10 is true, sets description to "a bit cold"
    temp < 10 -> "a bit cold"
    // If temp < 20 is true, sets description to "warm"
    temp < 20 -> "warm"
    // Sets description to "hot" if no previous condition is satisfied
    else -> "hot"             
}
println(description)
// warm
```



## Range



## Loops

### For

### While



# Functions

## Named arguments

## Default parameter values

## Single-expression funcitons

```kotlin
fun sum(x: Int, y: Int) = x + y

fun main() {
    println(sum(1, 2))
    // 3
}
```



## Lambda expressions

Lambda expressions are written within curly braces `{}`.

```kotlin
fun main() {
    println({ string: String -> string.uppercase() }("hello"))
    // HELLO
}
```

- the parameter followed by an `->`
- the function body after the `->`



### Assign to variable



### Pass to another function



### Function types



### Return from a function



### Invoke separately



# Classes

Properties

Create instance

Access properties

Member functions

Data classes



# Null safety

Null safety is a combination of features that allow you to:

- explicitly declare when `null` values are allowed in your program.
- check for `null` values.
- use safe calls to properties or functions that may contain `null` values.
- declare actions to take if `null` values are detected.



## Nullable types

Kotlin supports nullable types which allows the possibility for the declared type to have `null` values. By default, a type is **not** allowed to accept `null` values. Nullable types are declared by explicitly adding `?` after the type declaration.

```kotlin
fun main() {
    // neverNull has String type
    var neverNull: String = "This can't be null"

    // Throws a compiler error
    neverNull = null

    // nullable has nullable String type
    var nullable: String? = "You can keep a null here"

    // This is OK  
    nullable = null

    // By default, null values aren't accepted
    var inferredNonNull = "The compiler assumes non-nullable"

    // Throws a compiler error
    inferredNonNull = null

    // notNull doesn't accept null values
    fun strLength(notNull: String): Int {                 
        return notNull.length
    }

    println(strLength(neverNull)) // 18
    println(strLength(nullable))  // Throws a compiler error
}
```



## Check for null values

You can check for the presence of `null` values within conditional expressions. In the following example, the `describeString()` function has an `if` statement that checks whether `maybeString` is **not** `null` and if its `length` is greater than zero:

```kotlin
fun describeString(maybeString: String?): String {
    if (maybeString != null && maybeString.length > 0) {
        return "String of length ${maybeString.length}"
    } else {
        return "Empty or null string"
    }
}

fun main() {
    var nullString: String? = null
    println(describeString(nullString))
    // Empty or null string
}
```



## Use safe calls

To safely access properties of an object that might contain a `null` value, use the safe call operator `?.`. The safe call operator returns `null` if the object's property is `null`. This is useful if you want to avoid the presence of `null` values triggering errors in your code.

In the following example, the `lengthString()` function uses a safe call to return either the length of the string or `null`:

```kotlin
fun lengthString(maybeString: String?): Int? = maybeString?.length

fun main() { 
    var nullString: String? = null
    println(lengthString(nullString))
    // null
}
```

Safe calls can be chained so that if any property of an object contains a `null` value, then `null` is returned without an error being thrown. For example:

```kotlin
person.company?.address?.country
```

The safe call operator can also be used to safely call an extension or member function. In this case, a null check is performed before the function is called. If the check detects a `null` value, then the call is skipped and `null` is returned.

In the following example, `nullString` is `null` so the invocation of [`.uppercase()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/uppercase.html) is skipped and `null` is returned:

```kotlin
fun main() {
    var nullString: String? = null
    println(nullString?.uppercase())
    // null
}
```





## Use Elvis operator

You can provide a default value to return if a `null` value is detected by using the **Elvis operator** `?:`.

```kotlin
fun main() {
    var nullString: String? = null
    println(nullString?.length ?: 0)
    // 0
}
```



# Kotlin for Android

[Android’s Kotlin-first approach  | Android Developers](https://developer.android.com/kotlin/first)



## Kotlin patterns with Andorid

### Use fragments

```kotlin
class LoginFragment : Fragment() {
    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?): View? {
    	return inflater.inflate(R.layout.login_fragment, container, false)
	}
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    	super.onViewCreated(view, savedInstanceState)
	}
}
```

1. Nullability and initialization

   上面重写方法中参数以 `?` 为后缀的类型，表示传递的参数可为 null，需要安全处理 null。

   Kotlin 中声明对象必须初始化对象的属性，当获取类的实例时，可以立即引用其任何可访问属性。但是，调用 `Framgent#onCeateView` 之前，`Fragment` 中的 `view` 对象尚未 `inflate`，因此需要延迟 `View` 属性的初始化。

   ```kotlin
   class LoginFragment : Fragment() {
   
       private lateinit var usernameEditText: EditText
       private lateinit var passwordEditText: EditText
       private lateinit var loginButton: Button
       private lateinit var statusTextView: TextView
   
       override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
           super.onViewCreated(view, savedInstanceState)
   
           usernameEditText = view.findViewById(R.id.username_edit_text)
           passwordEditText = view.findViewById(R.id.password_edit_text)
           loginButton = view.findViewById(R.id.login_button)
           statusTextView = view.findViewById(R.id.status_text_view)
       }
   
       ...
   }
   ```

   如果您在初始化属性之前访问该属性，Kotlin 会抛出一个 `UninitializedPropertyAccessException` .

   

2. SAM conversion

    [Single Abstract Method conversion](https://kotlinlang.org/docs/reference/java-interop.html#sam-conversions)

   ```kotlin
   loginButton.setOnClickListener {
       val authSuccessful: Boolean = viewModel.authenticate(
               usernameEditText.text.toString(),
               passwordEditText.text.toString()
       )
       if (authSuccessful) {
           // Navigate to next screen
       } else {
           statusTextView.text = requireContext().getString(R.string.auth_failed)
       }
   }
   ```

   

3. Companion objects

   类似于 Java static 关键字

   ```kotlin
   class LoginFragment : Fragment() {
   
       ...
   
       companion object {
           private const val TAG = "LoginFragment"
       }
   }
   ```

   

4. Property delegation

   初始化属性时，可以复用 Android 更常见的模式，例如在 `Fragment` 访问 `ViewModel`，为避免过多重复代码，可以使用 Kotlin 的属性委派语法。

   ```kotlin
   private val viewModel: LoginViewModel by viewModels()
   ```

   属性委派提供了一个通用实现，在整个应用中可以重复使用该实现。Android KTS 提供了一些属性委派。例如，`viewModels`，检索 `ViewModel` 范围限定为当前 `Fragment`。

   属性委派使用反射，会增加性能开销，但是语法简洁。



### Nullability

Kotlin 提供了严格的 nullability rules，可以维护整个应用的类型安全。默认情况下，对象的引用不能包含 null 值。若要将 null 值赋值为变量，base type 末尾必须添加 `?` 声明可为 null 的变量类型。

```kotlin
val name: String? = null
```

1. Interoperability 

   降低 NPE，减少 null 检查。

   如果使用 Kotlin 引用 Java 类中定义的未注释 `name` member，编译器不知道映射到 Kotlin 中的是 `String` 还是 `String?`，编译器会允许你分配任一类型的值。如果将其类型表示为 `String`并分配 null 值，会造成 NPE。

   为解决该问题，用 Java 编写代码时，应该使用可空性注解。

   ```java
   public class Account implements Parcelable {
       public final String name;
       public final String type;
       private final @Nullable String accessId;
   
       ...
   }
   ```

    `accessId` 用注解`@Nullable` ，表明它可以保存 null 值。Kotlin 会将其视为 `accessId` `String?` .

   

2. Handling nullability

   如果不确定 Java 类型，应该假定它是可为 null 的 `String`。需要使用 safe-call 运算符

   ```Kotlin
   val account = Account("name", "type")
   val accountName = account.name?.trim()
   ```

   `name` 为非 null 时，结果为前后不带空格的 `name` 值。为 null 时，结果为 `null`，执行此语句也不能抛出NPE。

   safe-call 运算符虽然避免了 NPE，但是确实将 null 值传递给下一个语句，可以该用 Elvis 运算符（`?:`）来立即处理 null 的情况。

   ```Kotlin
   val account = Account("name", "type")
   val accountName = account.name?.trim() ?: "Default name"
   ```

   如果 Elvis 运算符的左侧表达式结果为 null，则将右侧的值分配给 `accountName`。

   还可以用 Elvis运算符提前从函数 return

   ```Kotlin
   fun validateAccount(account: Account?) {
       val accountName = account?.name?.trim() ?: "Default name"
   
       // account cannot be null beyond this point
       account ?: return
   
       ...
   }
   ```

   

3. Property initialization

   你可能希望在一个 `Fragment` 中引用 `View`，这意味着必须先 inflate layout，Fragment 构建时不会发生 inflate，而是在 `Fragment#onCreateView` 时会 inflate。

   一种方法是将 View 声明可为 null 并尽快对其进行初始化，但是更好的解决方案是使用 `lateinit` `View` 初始化

   ```Kotlin
   class LoginFragment : Fragment() {
       private lateinit var statusTextView: TextView
   
       override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
               super.onViewCreated(view, savedInstanceState)
   
               statusTextView = view.findViewById(R.id.status_text_view)
               statusTextView.setText(R.string.auth_failed)
       }
   }
   ```

    `lateinit` 关键字允许你避免在构造对象时初始化属性。如果您的属性在初始化之前被引用，Kotlin 会抛出一个 `UninitializedPropertyAccessException` ，因此要尽快初始化属性。



## Add Kotlin to an existing app



