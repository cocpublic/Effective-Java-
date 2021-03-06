### 第24条：优先考虑静态成员类

A nested class is a class defined within another class. A nested class should exist only to serve its enclosing class. If a nested class would be useful in some other context, then it should be a top-level class. There are four kinds of nested classes: static member classes, nonstatic member classes, anonymous classes, and local classes. All but the first kind are known as inner classes. This item tells you when to use which kind of nested class and why.

嵌套类是定义在另一个类中的类。一个嵌套类的存在应该是为了服务它的外围类。如果一个嵌套类还可以用于别的上下文，那么它就应该是个顶层类（top-level class）。有4种嵌套类：静态成员类，非静态成员类，匿名类以及局部类。除了第一种，其余的都被称为内部类。本条目将告诉你何时用哪种嵌套类以及这么做的原因。

A static member class is the simplest kind of nested class. It is best thought of as an ordinary class that happens to be declared inside another class and has access to all of the enclosing class’s members, even those declared private. A static member class is a static member of its enclosing class and obeys the same accessibility rules as other static members. If it is declared private, it is accessible only within the enclosing class, and so forth.

静态成员类是最简单的嵌套类。我们可以把它看成是一个普通的类，只是这个类恰好在别的类的内部声明，而且可以访问外围类的所有成员，即使是那些私有成员。静态成员类是它的外围类的静态成员，与其它的静态成员一样有着相同的可访问性。如果它被声明成私有的，那么它就只能在它的外围类里被访问，等等。

One common use of a static member class is as a public helper class, useful only in conjunction with its outer class. For example, consider an enum describing the operations supported by a calculator \(Item 34\). The Operation enum should be a public static member class of the Calculator class. Clients of Calculator could then refer to operations using names like Calculator.Operation.PLUS and Calculator.Operation.MINUS.

静态成员类通常的一个用法是作为一个公有的辅助类，只有与它的外围类一起用时才有意义。例如，考虑一个枚举，它描述了计算器所支持的所有操作（条目34）。操作枚举应该是作为Calculator类的一个公有静态成员类。然后Calculator的客户端可以通过使用名字如Calculator.Operation.PLUS和Calculator.Operations.MINUS来引用操作。

Syntactically, the only difference between static and nonstatic member classes is that static member classes have the modifier static in their declarations. Despite the syntactic similarity, these two kinds of nested classes are very different. Each instance of a nonstatic member class is implicitly associated with an enclosing instance of its containing class. Within instance methods of a nonstatic member class, you can invoke methods on the enclosing instance or obtain a reference to the enclosing instance using the qualified this construct \[JLS, 15.8.4\]. If an instance of a nested class can exist in isolation from an instance of its enclosing class, then the nested class must be a static member class: it is impossible to create an instance of a nonstatic member class without an enclosing instance.

从语法上来说，静态成员类和非静态成员类之间的区别是，静态成员类在声明上有static标识符。这两种嵌套类虽然语法上很相似，这但彼此之间却有很大的不同。非静态成员类的每个实例都是隐式地和它的外围类的实例关联在一起。在非静态成员类的实例方法内部，你可以调用外围实例的方法，或者通过标识了this的构造来获取外围实例的引用。

The association between a nonstatic member class instance and its enclosing instance is established when the member class instance is created and cannot be modified thereafter. Normally, the association is established automatically by invoking a nonstatic member class constructor from within an instance method of the enclosing class. It is possible, though rare, to establish the association manually using the expression enclosingInstance.new MemberClass\(args\). As you would expect, the association takes up space in the nonstatic member class instance and adds time to its construction.

在非静态成员类实例被创建时，非静态成员类实例和它的外围实例的关联就被建立了，而且建立后就不能被修改了。一般情况下，在外围类的实例方法内部调用非静态成员类的构造器时，这种关联就被自动建立。虽然可以手动使用表达式enclosingInstance.new MemberClass\(args\)来建立这个关联，但是很少这么做。正如你所料，这个关联需要消耗非静态成员类实例的空间，而且增加了它的构建时间。

One common use of a nonstatic member class is to define an Adapter \[Gamma95\] that allows an instance of the outer class to be viewed as an instance of some unrelated class. For example, implementations of the Map interface typically use nonstatic member classes to implement their collection views, which are returned by Map’s keySet, entrySet, and values methods. Similarly, implementations of the collection interfaces, such as Set and List, typically use nonstatic member classes to implement their iterators:

非静态成员类的一个通常的用法是，定义一个适配器 \[Gamma95\]，而这个适配器让外围类的实例被看成是某个不相关类的实例。例如，Map接口的实现通常会使用非静态成员类来实现它们的集合视图，这些视图由Map的keySet方法，entrySet方法以及值方法返回。类似地，一些集合接口（如Set和List）的实现通常用非静态成员类来实现它们的迭代器：

```
// Typical use of a nonstatic member class
public class MySet<E> extends AbstractSet<E> {
... // Bulk of the class omitted
    @Override 
    public Iterator<E> iterator() {
        return new MyIterator();
    } 
    private class MyIterator implements Iterator<E> {
        ...
    }
}
```

**If you declare a member class that does not require access to an enclosing instance, always put the static modifier in its declaration**, making it a static rather than a nonstatic member class. If you omit this modifier, each instance will have a hidden extraneous reference to its enclosing instance. As previously mentioned, storing this reference takes time and space. More seriously, it can result in the enclosing instance being retained when it would otherwise be eligible for garbage collection \(Item 7\). The resulting memory leak can be catastrophic. It is often difficult to detect because the reference is invisible.

**如果你声明了一个不需要访问外围实例的成员类，那你总是应该static修饰符加到声明里去**，使得这个成员类是静态的。如果你不加这个修饰符，那么每个实例都将包含一个隐藏的外围实例的引用。就如前面说的那样，存储这个引用将耗费时间和空间。更严重的是，当这个外围实例已经满足垃圾回收（条目7）的条件时，非静态成员类实例会导致外围实例被保留。因此而导致的内存泄露是灾难性的。由于这个引用是不可见的，所以这个问题也通常很难被检测到。

A common use of private static member classes is to represent components of the object represented by their enclosing class. For example, consider a Map instance, which associates keys with values. Many Map implementations have an internal Entry object for each key-value pair in the map. While each entry is associated with a map, the methods on an entry \(getKey, getValue, and setValue\) do not need access to the map. Therefore, it would be wasteful to use a nonstatic member class to represent entries: a private static member class is best. If you accidentally omit the static modifier in the entry declaration, the map will still work, but each entry will contain a superfluous reference to the map, which wastes space and time.

私有静态成员类通常被用来展示代表外围类对象的组件。例如，考虑一个Map实例，在这个实例里，每个键关联一个值。在许多Map实现里，每个键值对都有一个对应的内部Entry对象。虽然每个Entry对象都与一个Map对象关联，但是Entry对象的方法（getKey，getValue和setValue）无需访问Map对象。因而，采用非静态成员类来代表Entry对象是很浪费的，使用私有静态成员类会更好。如果你不小心忘了在Entry的声明里添加static修饰符，该Map仍然可以工作，但每个Entry对象将包含多余的Map对象引用，浪费空间又浪费时间。

It is doubly important to choose correctly between a static and a nonstatic member class if the class in question is a public or protected member of an exported class. In this case, the member class is an exported API element and cannot be changed from a nonstatic to a static member class in a subsequent release without violating backward compatibility.

如果所讨论的类是某个导出类的公有或受保护成员，那么选择让它成为静态成员类还是非静态成员类是非常重要的。在这种情况下，成员类是导出API的元素，而且在将来的版本中，无法在不破坏向后兼容性的前提下将非静态成员类变更为静态成员类。

As you would expect, an anonymous class has no name. It is not a member of its enclosing class. Rather than being declared along with other members, it is simultaneously declared and instantiated at the point of use. Anonymous classes are permitted at any point in the code where an expression is legal. Anonymous classes have enclosing instances if and only if they occur in a nonstatic context. But even if they occur in a static context, they cannot have any static members other than constant variables, which are final primitive or string fields initialized to constant expressions \[JLS, 4.12.4\].

正如你所料，匿名类可以没有名字。匿名类并不是它的外围类的一个成员。它不仅与其它成员一起被声明，而且它在被使用时同时被声明和初始化。在代码里任意一个表达式合法的地方，匿名类都是可以使用的。当且仅当匿名类出现在非静态的上下文当中时，匿名类才有外围实例。但即使它们出现在静态的上下文当中，也不能拥有除了常量型变量的任何的静态成员，这些常量型变量是final的基本类型，或者初始化常量表达式的字符串属性\[JLS，4.12.4\]。

There are many limitations on the applicability of anonymous classes. You can’t instantiate them except at the point they’re declared. You can’t perform instanceof tests or do anything else that requires you to name the class. You can’t declare an anonymous class to implement multiple interfaces or to extend a class and implement an interface at the same time. Clients of an anonymous class can’t invoke any members except those it inherits from its supertype. Because anonymous classes occur in the midst of expressions, they must be kept short—about ten lines or fewer—or readability will suffer.

在使用匿名类时也有很多限制。除了在它们被声明的时候之外，你无法去初始化它们。你无法进行instanceof测试或者任何需要你指明类名的操作。你无法声明一个匿名类实现了多个接口或者扩展了一个类并同时实现一个接口。除了从父类型继承过来的成员，匿名类的客户端无法调用任何其它成员。由于匿名类在表达式里出现，它们必须是简短的，大约10行或者更少，否则会影响可阅读性。

Before lambdas were added to Java \(Chapter 6\), anonymous classes were the preferred means of creating small function objects and process objects on the fly, but lambdas are now preferred \(Item 42\). Another common use of anonymous classes is in the implementation of static factory methods\(see intArrayAsList in Item 20\).

在lambda表达式加入Java（第6章）之前，匿名类是创建小的函数对象和处理对象的首选方式，但现在lambda表达式是首选了（条目42）。匿名类的另一个常见用法是实现静态工厂方法（见条目20的intArrayAsList）。

Local classes are the least frequently used of the four kinds of nested classes. A local class can be declared practically anywhere a local variable can be declared and obeys the same scoping rules. Local classes have attributes in common with each of the other kinds of nested classes. Like member classes, they have names and can be used repeatedly. Like anonymous classes, they have enclosing instances only if they are defined in a nonstatic context, and they cannot contain static members. And like anonymous classes, they should be kept short so as not to harm readability.

在四种嵌套类里，局部类是最不常用的。在可以声明局部变量的地方就可以声明局部类，而且两者遵循相同的作用域规则。局部类与其它三种嵌套类有着共同的属性。像类成员一样，它们有名字而且能被重复使用。像匿名类一样，只有当它们在非静态上下文中被定义时，它们才有外围实例，而且它们不能包含静态成员。像匿名类一样，它们最好尽量的简短，以免影响可阅读性。

To recap, there are four different kinds of nested classes, and each has its place. If a nested class needs to be visible outside of a single method or is too long to fit comfortably inside a method, use a member class. If each instance of a member class needs a reference to its enclosing instance, make it nonstatic; otherwise, make it static. Assuming the class belongs inside a method, if you need to create instances from only one location and there is a preexisting type that characterizes the class, make it an anonymous class; otherwise, make it a local class.

我们回顾一下，有4种不同的嵌套类，每个都有其用途。如果一个嵌套类必须在方法外部可见，或者放在方法内部会显得太长时，就使用成员类。如果成员类的实例需要拥有该类的外围类的引用，就将其做成非静态；不然，就将其做成静态。假设一个类应当在方法内部，若你需要只从一个地方创建实例而且已经存在一个类型能说明这个类的特征，那么将其做成匿名类；否则，就将其做成局部类。

