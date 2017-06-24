# 管理任务

##### Android系统管理任务和返回栈的方式，就是把所有启动的Activity都放入到一个相同的任务当中，通过一个“后进先出”的栈来进行管理的。这种方式在绝大多数情况下都是没问题的，开发者也无须去关心任务中的Activity到底是怎么样存放在返回栈当中的。但是呢，如果你想打破这种默认的行为，比如说当启动一个新的Activity时，你希望它可以存在于一个独立的任务当中，而不是现有的任务当中。或者说，当启动一个Activity时，如果这个Activity已经存在于返回栈中了，你希望能把这个Activity直接移动到栈顶，而不是再创建一个它的实例。再或者，你希望可以将返回栈中除了最底层的那个Activity之外的其它所有Activity全部清除掉。这些功能甚至更多功能，都是可以通过在manifest文件中设置<activity>元素的属性，或者是在启动Activity时配置Intent的flag来实现的。

- taskAffinity
- launchMode
- allowTaskReparenting
- clearTaskOnLaunch
- alwaysRetainTaskState
- finishOnTaskLaunch

##### android中还有另外一种启动方式，在代码中给Intent设置flag（标志位），下面列举几个常用的flag

- Intent.FLAG_ACTIVITY_NEW_TASK

    - 当Intent对象包含这个标记时，系统会寻找或创建一个新的task来放置目标Activity，寻找时依据目标Activity的taskAffinity属性进行匹配，如果找到一个task的taskAffinity与之相同，就将目标Activity压入此task中，如果查找无果，则创建一个新的task，并将该task的taskAffinity设置为目标Activity的taskActivity，将目标Activity放置于此task
- Intent.FLAG_ACTIVITY_SINGLE_TOP

    - 该标志位表示使用singleTop模式启动一个Activity
-   Intent.FLAG_ACTIVITY_CLEAR_TOP

    - 该标志位表示使用singleTask模式启动一个Activity

    - 此时如果Activity B会接收到这个启动它的Intent，你可以决定是让Activity B调用onNewIntent()方法(不会创建新的实例)，还是将Activity B销毁掉并重新创建实例。如果Activity B没有在manifest中指定任何启动模式(也就是"standard"模式)，并且Intent中也没有加入一个FLAG_ACTIVITY_SINGLE_TOP flag，那么此时Activity B就会销毁掉，然后重新创建实例。而如果Activity B在manifest中指定了任何一种启动模式，或者是在Intent中加入了一个FLAG_ACTIVITY_SINGLE_TOP flag，那么就会调用Activity B的onNewIntent()方法。

-   Intent.FLAG_ACTIVITY_NO_HISTORY

    - 使用该模式来启动Activity，当该Activity启动其他Activity后，该Activity就被销毁了，不会保留在任务栈中。如A-B,B中以这种模式启动C，C再启动D，则任务栈只有ABD。

# 定义启动模式

-   使用manifest文件
当你在manifest文件中声明一个Activity的时候，你可以指定这个Activity在启动的时候该如何与任务进行关联。

-   使用Intent flag
当你调用startActivity()方法时，你可以在Intent中加入一个flag，从而指定新启动的Activity该如何与当前任务进行关联。

-   也就是说，如果Activity A启动了Activity B，Activity B可以定义自己该如何与当前任务进行关联，而Activity A也可以要求Activity B该如何与当前任务进行关联。如果Activity B在manifest中已经定义了该如何与任务进行关联，而Activity A同时也在Intent中要求了Activity B该怎么样与当前任务进行关联，那么此时Intent中的定义将覆盖manifest中的定义。

-   使用manifest文件

    -  Android系统中一个应用默认会创建一个任务栈。默认每启动一个界面都会创建一个新的Activity并添加进去，这样一个Activity的数据和信息状态都将会保留，造成数据冗余，重复数据太多，为了解决这些问题，android系统提供了一套Activity的启动模式来修改Activity的默认行为。

- Standard

    - 又称标准模式，也是系统默认模式，在这样的模式下，每启动一个Activity都会创建一个Activity的实例。并加入到任务栈中。

-  singleTop

    - 又称栈顶模式，在这种模式下如果新建的Activity位于任务栈的栈顶，Activity会被复用，会调用onNewIntent方法。
- singleTask

    - 这种启动模式表示，系统会创建一个新的任务，并将启动的Activity放入这个新任务的栈底位置。但是，如果现有任务当中已经存在一个该Activity的实例了，那么系统就不会再创建一次它的实例，而是会直接调用它的onNewIntent()方法。声明成这种启动模式的Activity，在同一个任务当中只会存在一个实例。注意这里我们所说的启动Activity，都指的是启动其它应用程序中的Activity，因为"singleTask"模式在默认情况下只有启动其它程序的Activity才会创建一个新的任务，启动自己程序中的Activity还是会使用相同的任务

- singleInstance

    - 这种模式下，系统会创建一个新的任务栈存放Activity。

-  处理affinity

    -  affinity可以用于指定一个Activity更加愿意依附于哪一个任务，在默认情况下，同一个应用程序中的所有Activity都具有相同的affinity，所以，这些Activity都更加倾向于运行在相同的任务当中。当然了，你也可以去改变每个Activity的affinity值，通过<activity>元素的taskAffinity属性就可以实现了。

- 当调用startActivity()方法来启动一个Activity时，默认是将它放入到当前的任务当中。但是，如果在Intent中加入了一个FLAG_ACTIVITY_NEW_TASK flag的话(或者该Activity在manifest文件中声明的启动模式是"singleTask")，系统就会尝试为这个Activity单独创建一个任务。但是规则并不是只有这么简单，系统会去检测要启动的这个Activity的affinity和当前任务的affinity是否相同，如果相同的话就会把它放入到现有任务当中，如果不同则会去创建一个新的任务。而同一个程序中所有Activity的affinity默认都是相同的，这也是前面为什么说，同一个应用程序中即使声明成"singleTask"，也不会为这个Activity再去创建一个新的任务了。

- 当把Activity的allowTaskReparenting属性设置成true时，Activity就拥有了一个转移所在任务的能力。具体点来说，就是一个Activity现在是处于某个任务当中的，但是它与另外一个任务具有相同的affinity值，那么当另外这个任务切换到前台的时候，该Activity就可以转移到现在的这个任务当中。

# 清空返回栈

-  如何用户将任务切换到后台之后过了很长一段时间，系统会将这个任务中除了最底层的那个Activity之外的其它所有Activity全部清除掉。当用户重新回到这个任务的时候，最底层的那个Activity将得到恢复。这个是系统默认的行为，因为既然过了这么长的一段时间，用户很有可能早就忘记了当时正在做什么，那么重新回到这个任务的时候，基本上应该是要去做点新的事情了。

-  当然，既然说是默认的行为，那就说明我们肯定是有办法来改变的，在<activity>元素中设置以下几种属性就可以改变系统这一默认行为：

-  alwaysRetainTaskState
    如果将最底层的那个Activity的这个属性设置为true，那么上面所描述的默认行为就将不会发生，任务中所有的Activity即使过了很长一段时间之后仍然会被继续保留。

-   clearTaskOnLaunch
    如果将最底层的那个Activity的这个属性设置为true，那么只要用户离开了当前任务，再次返回的时候就会将最底层Activity之上的所有其它Activity全部清除掉。简单来讲，就是一种和alwaysRetainTaskState完全相反的工作模式，它保证每次返回任务的时候都会是一种初始化状态，即使用户仅仅离开了很短的一段时间。

-   finishOnTaskLaunch
    这个属性和clearTaskOnLaunch是比较类似的，不过它不是作用于整个任务上的，而是作用于单个Activity上。如果某个Activity将这个属性设置成true，那么用户一旦离开了当前任务，再次返回时这个Activity就会被清除掉。
