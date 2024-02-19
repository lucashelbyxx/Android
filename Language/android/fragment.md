# 一、概览

[`Fragment`](https://developer.android.com/reference/androidx/fragment/app/Fragment) 表示应用界面中可重复使用的一部分。Fragment 定义和管理自己的布局，具有自己的生命周期，并且可以处理自己的输入事件。Fragment 不能独立存在，而是必须由 Activity 或另一个 Fragment 托管。Fragment 的视图层次结构会成为宿主的视图层次结构的一部分，或附加到宿主的视图层次结构。

您可以在同一 Activity 或多个 Activity 中使用同一 Fragment 类的多个实例，甚至可以将其用作另一个 Fragment 的子级。考虑到这一点，您应只为 Fragment 提供管理它自己的界面所需的逻辑。您应避免让一个 Fragment 依赖于另一个 Fragment 或从一个 Fragment 操控另一个 Fragment。

# 二、创建 Fragment

```java
class ExampleFragment extends Fragment {
    public ExampleFragment() {
        super(R.layout.example_fragment);
    }
}
```

强烈建议始终使用 FragmentContainerView 作为片段的容器，因为 FragmentContainerView 包含其他视图组（例如 FrameLayout）不提供的特定于片段的修复。

1、XML 添加 fragment

2、程序中添加 fragment

```xml
<!-- res/layout/example_activity.xml -->
<androidx.fragment.app.FragmentContainerView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/fragment_container_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

```java
public class ExampleActivity extends AppCompatActivity {
    public ExampleActivity() {
        super(R.layout.example_activity);
    }
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (savedInstanceState == null) {
            getSupportFragmentManager().beginTransaction()
                .setReorderingAllowed(true)
                .add(R.id.fragment_container_view, ExampleFragment.class, null)
                .commit();
        }
    }
}
```

当配置发生变化，重新创建activity时，saveInstanceState不再为null，不需要再次添加fragment，因为fragment会自动从saveInstanceState恢复。如果您的片段需要一些初始数据，则可以通过在对 FragmentTransaction.add() 的调用中提供一个 Bundle 将参数传递给您的片段。然后可以通过调用 requireArguments() 从片段中检索参数 Bundle，并且可以使用适当的 Bundle getter 方法来检索每个参数。

# 三、 Fragment 管理器

[`FragmentManager`](https://developer.android.com/reference/androidx/fragment/app/FragmentManager) 类负责在应用的 fragment 上执行一些操作，如添加、移除或替换操作，以及将操作添加到返回堆栈。

如果您使用的是 [Jetpack Navigation](https://developer.android.com/guide/navigation) 库，则您可能永远都不会与 `FragmentManager` 直接进行交互，因为该库会代表您来使用 `FragmentManager`。尽管如此，任何使用 fragment 的应用都在某种程度上使用了 `FragmentManager`，因此您需要了解它是什么以及如何工作。

## 1、访问 FragmentManager

**在 Activity 中访问**

每个 [`FragmentActivity`](https://developer.android.com/reference/androidx/fragment/app/FragmentActivity) 及其子类（如 [`AppCompatActivity`](https://developer.android.com/reference/androidx/appcompat/app/AppCompatActivity)）都可以通过 [`getSupportFragmentManager()`](https://developer.android.com/reference/androidx/fragment/app/FragmentActivity#getSupportFragmentManager()) 方法访问 `FragmentManager`。

**在 Fragment 中访问**

Fragment 也能够托管一个或多个子 fragment。在 fragment 内，可以通过 [`getChildFragmentManager()`](https://developer.android.com/reference/androidx/fragment/app/Fragment#getChildFragmentManager()) 获取对管理 fragment 子级的 `FragmentManager` 的引用。如果您需要访问其宿主 `FragmentManager`，可以使用 [`getParentFragmentManager()`](https://developer.android.com/reference/androidx/fragment/app/Fragment#getParentFragmentManager())。



## 2、使用 FragmentManager

`FragmentManager` 管理 fragment 返回堆栈。在运行时，`FragmentManager` 可以执行添加或移除 fragment 等返回堆栈操作来响应用户互动。每一组更改作为一个单元（称为 [`FragmentTransaction`](https://developer.android.com/reference/androidx/fragment/app/FragmentTransaction)）一起提交。

当用户按设备上的“返回”按钮时，或者当您调用 [`FragmentManager.popBackStack()`](https://developer.android.com/reference/androidx/fragment/app/FragmentManager#popBackStack()) 时，最上面的 fragment 事务会从堆栈中弹出。换句话说，事务是反转的。如果堆栈上没有更多 fragment 事务，并且您没有使用子 fragment，则返回事件会向上传递到 activity。

当您对事务调用 [`addToBackStack()`](https://developer.android.com/reference/androidx/fragment/app/FragmentTransaction#addToBackStack(java.lang.String)) 时，请注意，事务可以包括任意数量的操作，如添加多个 fragment、替换多个容器中的 fragment，等等。弹出返回堆栈时，所有这些操作会作为一项原子化操作反转。如果您在调用 `popBackStack()` 之前提交了其他事务，并且您没有对事务使用 `addToBackStack()`，则这些操作不会反转。因此，在一个 `FragmentTransaction` 中，应避免让影响返回堆栈的事务与不影响返回堆栈的事务交织在一起。

## 3、执行事务

如需在布局容器中显示 fragment，请使用 `FragmentManager` 创建 `FragmentTransaction`。在事务中，您随后可以对容器执行 [`add()`](https://developer.android.com/reference/androidx/fragment/app/FragmentTransaction#add(int, java.lang.Class, android.os.Bundle)) 或 [`replace()`](https://developer.android.com/reference/androidx/fragment/app/FragmentTransaction#replace(int, java.lang.Class, android.os.Bundle)) 操作。

```java
FragmentManager fragmentManager = getSupportFragmentManager();
fragmentManager.beginTransaction()
    .replace(R.id.fragment_container, ExampleFragment.class, null)
    .setReorderingAllowed(true)
    .addToBackStack("name") // name can be null
    .commit();
```

`ExampleFragment` 会替换当前在布局容器中的 fragment（如有），该布局容器由 `R.id.fragment_container` ID 进行标识。将 fragment 的类提供给 [`replace()`](https://developer.android.com/reference/androidx/fragment/app/FragmentTransaction#replace(int, java.lang.Class, android.os.Bundle)) 方法可让 `FragmentManager` 使用其 [`FragmentFactory`](https://developer.android.com/reference/androidx/fragment/app/FragmentFactory) 处理实例化。[`setReorderingAllowed(true)`](https://developer.android.com/reference/androidx/fragment/app/FragmentTransaction#setReorderingAllowed(boolean)) 可优化事务中涉及的 fragment 的状态变化，以使动画和过渡正常运行。

调用 [`addToBackStack()`](https://developer.android.com/reference/androidx/fragment/app/FragmentTransaction#addToBackStack(java.lang.String)) 会将事务提交到返回堆栈。用户稍后可以通过按“返回”按钮反转事务，并恢复上一个 fragment。如果您在一个事务中添加或移除了多个 fragment，弹出返回堆栈时，所有这些操作都会撤消。在 `addToBackStack()` 调用中提供的可选名称使您能够使用 [`popBackStack()`](https://developer.android.com/reference/androidx/fragment/app/FragmentManager#popBackStack(java.lang.String, int)) 弹回到该特定事务。

如果您在执行移除 fragment 的事务时未调用 `addToBackStack()`，则提交事务时会销毁已移除的 fragment，用户无法返回到该 fragment。如果您在移除某个 fragment 时调用了 `addToBackStack()`，则该 fragment 只会 `STOPPED`，稍后当用户返回时它会 `RESUMED`。请注意，在这种情况下，其视图会被销毁。

查找现有 fragment

您可以使用 [`findFragmentById()`](https://developer.android.com/reference/androidx/fragment/app/FragmentManager#findFragmentById(int)) 获取对布局容器中当前 fragment 的引用。从 XML 扩充时，可使用 `findFragmentById()` 按给定的 ID 查找 fragment；在 `FragmentTransaction` 中添加时，可使用它按容器 ID 进行查找。

```java
FragmentManager fragmentManager = getSupportFragmentManager();
fragmentManager.beginTransaction()
    .replace(R.id.fragment_container, ExampleFragment.class, null)
    .setReorderingAllowed(true)
    .addToBackStack(null)
    .commit();

...

ExampleFragment fragment =
        (ExampleFragment) fragmentManager.findFragmentById(R.id.fragment_container);
```

您也可以为 fragment 分配一个唯一的标记，并使用 [`findFragmentByTag()`](https://developer.android.com/reference/androidx/fragment/app/FragmentManager#findFragmentByTag(java.lang.String)) 获取引用。您可以在布局中定义的 fragment 上使用 `android:tag` XML 属性来分配标记，也可以在 `FragmentTransaction` 中的 `add()` 或 `replace()` 操作期间分配标记。

```java
FragmentManager fragmentManager = getSupportFragmentManager();
fragmentManager.beginTransaction()
    .replace(R.id.fragment_container, ExampleFragment.class, null, "tag")
    .setReorderingAllowed(true)
    .addToBackStack(null)
    .commit();

...

ExampleFragment fragment = (ExampleFragment) fragmentManager.findFragmentByTag("tag");
```

**有关子 fragment 和同级 fragment 的特殊注意事项**

在任何给定的时间点，只允许一个 `FragmentManager` 控制 fragment 返回堆栈。如果应用在屏幕上同时显示多个同级 fragment，或者应用使用子 fragment，则必须指定一个 `FragmentManager` 来处理应用的主要导航。

将导航结构视为一系列层，其中 activity 作为最外层，封装下面的每一层子 fragment。每一层都必须有一个主要导航 fragment。当发生返回事件时，最内层控制导航行为。一旦最内层再也没有可从其弹回的 fragment 事务，控制权就会向外回一层，此过程会一直重复，直至到达 activity 为止。

请注意，当同时显示两个或更多 fragment 时，其中只有一个可以是主要导航 fragment。如果将某个 fragment 设为主要导航 fragment，会移除对先前 fragment 的指定。在上例中，如果您将详情 fragment 设为主要导航 fragment，就会移除对主 fragment 的指定。

## 4、支持多个返回堆栈

## 5、为 fragment 提供依赖项

添加 fragment 时，您可以手动实例化 fragment 并将其添加到 `FragmentTransaction`。

```java
// Instantiate a new instance before adding
ExampleFragment myFragment = new ExampleFragment();
fragmentManager.beginTransaction()
    .add(R.id.fragment_view_container, myFragment)
    .setReorderingAllowed(true)
    .commit();
```

当您提交 fragment 事务时，您创建的 fragment 实例就是使用的实例。不过，在[配置更改](https://developer.android.com/guide/topics/resources/runtime-changes)期间，activity 及其所有 fragment 都会被销毁，然后使用最适用的 [Android 资源](https://developer.android.com/guide/topics/resources/providing-resources#BestMatch)重新创建。`FragmentManager` 会为您处理所有这些操作。它会重新创建 fragment 的实例，将其附加到宿主，并重新创建返回堆栈状态。

默认情况下，`FragmentManager` 会使用框架提供的 [`FragmentFactory`](https://developer.android.com/reference/androidx/fragment/app/FragmentFactory) 来实例化 fragment 的新实例。此默认工厂使用反射来查找和调用 fragment 的无参数构造函数。这意味着，您无法使用此默认工厂为 fragment 提供依赖项。这也意味着，默认情况下，在重新创建过程中，不会使用您首次创建 fragment 时所用的任何自定义构造函数。

# 四、Fragment transactions 事务

在运行时，FragmentManager 可以添加、删除、替换和使用片段执行其他操作以响应用户交互。您提交的每组片段更改称为一个事务，您可以使用 FragmentTransaction 类提供的 API 指定在事务中执行的操作。您可以将多个操作分组到一个事务中——例如，一个事务可以添加或替换多个片段。当您在同一屏幕上显示多个同级片段时，此分组可能很有用，例如拆分视图。

可以将每个事务保存到由 FragmentManager 管理的回退堆栈中，从而允许用户在片段更改中向后导航——类似于在活动中向后导航。

您可以通过调用 beginTransaction() 从 FragmentManager 获取 FragmentTransaction 的实例。

**允许对片段状态更改进行重新排序**

为了行为兼容性，默认情况下不启用重新排序标志。但是，需要允许 FragmentManager 正确执行您的 FragmentTransaction，特别是当它在后台堆栈上操作并运行动画和转换时。启用该标志可确保如果多个事务一起执行，任何中间片段（即添加然后立即替换的片段）不会经历生命周期更改或执行其动画或转换。请注意，此标志会影响事务的初始执行和使用 popBackStack() 撤消事务。

**添加、移除 Fragment**

要将片段添加到 FragmentManager，请在事务上调用 add()。此方法接收片段的容器 ID，以及您希望添加的片段的类名。添加的片段被移动到 RESUMED 状态。强烈建议容器是一个 FragmentContainerView，它是视图层次结构的一部分。

要从主机中删除片段，请调用 remove()，传入通过 findFragmentById() 或 findFragmentByTag() 从片段管理器检索到的片段实例。如果片段的视图先前已添加到容器中，则此时视图将从容器中删除。移除的片段被移动到 DESTROYED 状态。

使用 replace() 将容器中的现有片段替换为您提供的新片段类的实例。调用 replace() 等效于在容器中调用 remove() 片段并将新片段添加到同一容器中。

强烈建议始终使用采用类而不是片段实例的片段操作，以确保创建片段的相同机制也用于从保存状态恢复片段。默认情况下，在 FragmentTransaction 中所做的更改不会添加到后台堆栈。要保存这些更改，您可以在 FragmentTransaction 上调用 addToBackStack()。

**提交是异步的**
调用 commit() 不会立即执行事务。相反，事务被安排在主 UI 线程上运行，只要它能够这样做。但是，如有必要，您可以调用 commitNow() 立即在您的 UI 线程上运行片段事务。
请注意，commitNow 与 addToBackStack 不兼容。或者，您可以通过调用 executePendingTransactions() 来执行所有由 commit() 调用提交但尚未运行的挂起的 FragmentTransaction。这种方法与 addToBackStack 兼容。对于绝大多数用例，commit() 就是您所需要的。

**操作顺序很重要**
在 FragmentTransaction 中执行操作的顺序很重要，尤其是在使用 setCustomAnimations() 时。此方法将给定的动画应用于其后的所有片段操作。

```java
getSupportFragmentManager().beginTransaction()
        .setCustomAnimations(enter1, exit1, popEnter1, popExit1)
        .add(R.id.container, ExampleFragment.class, null) // gets the first animations
        .setCustomAnimations(enter2, exit2, popEnter2, popExit2)
        .add(R.id.container, ExampleFragment.class, null) // gets the second animations
        .commit()
```

**限制片段的生命周期**

**显示和隐藏片段的视图**
使用 FragmentTransaction 方法 show() 和 hide() 显示和隐藏已添加到容器的片段的视图。这些方法设置片段视图的可见性，而不影响片段的生命周期。
虽然您不需要使用片段事务来切换片段中视图的可见性，但这些方法对于您希望更改可见性状态与后台堆栈上的事务相关联的情况很有用。

**附加和分离片段**
FragmentTransaction 方法 detach() 将片段与 UI 分离，破坏其视图层次结构。片段保持在与放入后堆栈时相同的状态 (STOPPED)。这意味着片段已从 UI 中删除，但仍由片段管理器管理。
attach() 方法重新附加之前分离的片段。这会导致其视图层次结构被重新创建、附加到 UI 并显示。
由于 FragmentTransaction 被视为单个原子操作集，因此在同一事务中对同一片段实例的分离和附加调用有效地相互抵消，从而避免了片段 UI 的破坏和立即重建。使用单独的事务，如果使用 commit()，则由 executePendingOperations() 分隔，如果要分离然后立即重新附加片段。

# 使用 DialogFragment 显示对话框

DialogFragment 是一个特殊的片段子类，专为创建和托管对话框而设计。严格来说，您不需要在片段中托管对话框，但这样做允许 FragmentManager 管理对话框的状态并在发生配置更改时自动恢复对话框。

## 1、创建 `DialogFragment`

```java
public class PurchaseConfirmationDialogFragment extends DialogFragment {
   @NonNull
   @Override
   public Dialog onCreateDialog(@Nullable Bundle savedInstanceState) {
       return new AlertDialog.Builder(requireContext())
               .setMessage(getString(R.string.order_confirmation))
               .setPositiveButton(getString(R.string.ok), (dialog, which) -> {} )
               .create();
   }

   public static String TAG = "PurchaseConfirmationDialog";
}
```

## 2、显示 DialogFragment

不必手动创建 FragmentTransaction 来显示您的 DialogFragment。相反，使用 show() 方法来显示您的对话框。您可以传递对 FragmentManager 和 String 的引用以用作 FragmentTransaction 标记。从 Fragment 中创建 DialogFragment 时，您必须使用 Fragment 的子 FragmentManager 以确保在配置更改后正确恢复状态。非空标签允许您稍后使用 findFragmentByTag() 检索 DialogFragment。

```java
// From another Fragment or Activity where you wish to show this
// PurchaseConfirmationDialogFragment.
new PurchaseConfirmationDialogFragment().show(
       getChildFragmentManager(), PurchaseConfirmationDialog.TAG);
```

因为 DialogFragment 在配置更改后会自动恢复，所以请考虑仅根据用户操作或 findFragmentByTag() 返回 null 时调用 show()，表明该对话框不存在。

## 3、DialogFragment 生命周期

DialogFragment 遵循标准的片段生命周期。此外 DialogFragment 还有一些额外的生命周期回调。最常见的如下：
onCreateDialog() - 覆盖这个回调，为片段提供一个对话框来管理和显示。
onDismiss() - 如果您需要在 Dialog 关闭时执行任何自定义逻辑，例如释放资源、取消订阅可观察资源等，请覆盖此回调。
onCancel() - 如果您需要在取消对话框时执行任何自定义逻辑，请覆盖此回调。
DialogFragment 还包含关闭或设置 DialogFragment 可取消性的方法：
dismiss() - 关闭片段及其对话框。如果片段被添加到后退堆栈，则弹出所有后退堆栈状态直到并包括此条目。否则，将提交一个新事务以删除该片段。
setCancellable() - 控制显示的对话框是否可取消。应该使用此方法，而不是直接调用 Dialog.setCancelable(boolean)。
请注意，在将 DialogFragment 与 Dialog 一起使用时，您不会覆盖 onCreateView() 或 onViewCreated()。对话框不仅仅是视图——它们有自己的窗口。因此，覆盖 onCreateView() 是不够的。此外，永远不会在自定义 DialogFragment 上调用 onViewCreated() ，除非您已覆盖 onCreateView() 并提供非空视图。
