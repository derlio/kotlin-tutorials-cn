Kotlin's object model is substantially different from Python's. Most importantly, classes are _not_ dynamically modifiable at runtime! (There are some limited exceptions to this, but you generally shouldn't do it. However, it _is_ possible to dynamically _inspect_ classes and objects at runtime with a feature called _reflection_ - this can be useful, but should be judiciously used.) All properties (attributes) and functions that might ever be needed on a class must be declared either directly in the class body or as [_extension functions_](extension-functionsproperties.html), so you should think carefully through your class design.


## 声明与实例化

Classes are declared with the `class` keyword. A basic class without any properties or functions of its own looks like this:

```kotlin
class Empty
```

You can then create an instance of this class in a way that looks similar to Python, as if the class were a function (but this is just syntactic sugar - unlike Python, classes in Kotlin aren't really functions):

```kotlin
val object = Empty()
```

Class names should use `UpperCamelCase`, just like in Python.


## 继承的内置函数

Every class that doesn't explicitly declare a parent class inherits from `Any`, which is the root of the class hierarchy (similar to `object` in Python) - more on [继承](inheritance.html) later. Via `Any`, every class automatically has the following functions:

* `toString()` returns a string representation of the object, similar to `__str__()` in Python (the default implementation is rather uninteresting, as it only returns the class name and something akin to the object's id)
* `equals(x)` checks if this object is equal to some other object `x` of any class (by default, this just checks if this object is the _same_ object as `x` - just like `is` in Python - but it can be overridden by subclasses to do custom comparisons of property values)
* `hashCode()` returns an integer that can be used by hash tables and for shortcutting complex equality comparisons (objects that are equal according to `equals()` must have the same hash code, so if two objects' hash codes are different, the objects cannot be equal)


## 属性

Empty classes aren't very interesting, so let's make a class with some _properties_:

```kotlin
class Person {
    var name = "Anne"
    var age = 32
}
```

Note that the type of a property must be explicitly specified. As opposed to Python, declaring a property directly inside the class does not create a class-level property, but an instance-level one: every instance of `Person` will have _its own_ `name` and `age`. Their values will start out in every instance as `"Anne"` and `32`, respectively, but the value in each instance can be modified independently of the others:

```kotlin
val a = Person()
val b = Person()
println("${a.age} ${b.age}") // Prints "32 32"
a.age = 42
println("${a.age} ${b.age}") // Prints "42 32"
```

To be fair, you'd get the same output in Python, but the mechanism would be different: both instances would start out without any attributes of their own (`age` and `name` would be attributes on the class), and the first printing would access the class attribute; only the assignment would cause an `age` attribute to appear on `a`. In Kotlin, there are no class properties in this example, and each instance starts out with both properties. If you need a class-level property, see the section on [伴生对象](objects-and-companion-objects.html#伴生对象).

Because the set of properties of an object is constrained to be exactly the set of properties that are declared at compile-time in the object's class, it's not possible to add new properties to an object or to a class at runtime, so e.g. `a.nationality = "Norwegian"` won't compile.

Property names should use `lowerCamelCase` instead of `snake_case`.


## 构造函数与初始化块

Properties that don't have a sensible default should be taken as constructor parameters. Like with Python's `__init__()`, Kotlin constructors and initializer blocks run automatically whenever an instance of an object is created (note that there's nothing that corresponds to `__new__()`).  A Kotlin class may have one _primary constructor_, whose parameters are supplied after the class name. The primary constructor parameters are available when you initialize properties in the class body, and also in the optional _initializer block_, which can contain complex initialization logic (a property can be declared without an initial value, in which case it must be initialized in `init`). Also, you'll frequently want to use `val` instead of `var` in order to make your properties immutable after construction.

```kotlin
class Person(firstName: String, lastName: String, yearOfBirth: Int) {
    val fullName = "$firstName $lastName"
    var age: Int
    
    init {
        age = 2018 - yearOfBirth
    }
}
```

If all you want to do with a constructor parameter value is to assign it to a property with the same name, you can declare the property in the primary constructor parameter list (the oneliner below is sufficient for both declaring the properties, declaring the constructor parameters, and initializing the properties with the parameters):

```kotlin
class Person(val name: String, var age: Int)
```

If you need multiple ways to initialize a class, you can create _secondary constructors_, each of which looks like a function whose name is `constructor`. Every secondary constructor must invoke another (primary or secondary) constructor by using the `this` keyword as if it were a function (so that every instance construction eventually calls the primary constructor).

```kotlin
class Person(val name: String, var age: Int) {
    constructor(name: String) : this(name, 0)
    constructor(yearOfBirth: Int, name: String)
        : this(name, 2018 - yearOfBirth)
}
```

(A secondary constructor can also have a body in curly braces if needs to do more than what the primary constructor does.) The constructors are distinguished from each other through the types of their parameters, like in ordinary function overloading. That's the reason we had to flip the parameter order in the last secondary constructor - otherwise, it would have been indistinguishable from the primary constructor (parameter names are not a part of a function's signature and don't have any effect on overload resolution). In the most recent example, we can now create a `Person` in three different ways:

```kotlin
val a = Person("Jaime", 35)
val b = Person("Jack") // age = 0
val c = Person(1995, "Lynne") // age = 23
```

Note that if a class has got a primary constructor, it is no longer possible to create an instance of it without supplying any parameters (unless one of the secondary constructors is parameterless).


## Setter 与 getter

A property is really a _backing field_ (kind of a hidden variable inside the object) and two accessor functions: one that gets the value of the variable and one that sets the value. You can override one or both of the accessors (an accessor that is not overridden automatically gets the default behavior of just returning or setting the backing field directly). Inside an accessor, you can reference the backing field with `field`. The setter accessor must take a parameter `value`, which is the value that is being assigned to the property. A getter body could either be a one-line expression preceded by `=` or a more complex body enclosed in curly braces, while a setter body typically includes an assignment and must therefore be enclosed in curly braces.  If you want to validate that the age is nonnegative:

```kotlin
class Person(age: Int) {
    var age = 0
        set(value) {
            if (value < 0) throw IllegalArgumentException(
                    "Age cannot be negative")
            field = value
        }

    init {
        this.age = age
    }
}
```

Annoyingly, the setter logic is not invoked by the initialization, which instead sets the backing field directly - that's why we have to use an initializer block in this example in order to verify that newly-created persons also don't get a negative age. Note the use of `this.age` in the initializer block in order to distinguish between the identically-named property and constructor parameter.

If for some reason you want to store a different value in the backing field than the value that is being assigned to the property, you're free to do that, but then you will probably want a getter to give the calling code back what they expect: if you say `field = value * 2` in the setter and `this.age = age * 2` in the initializer block, you should also have `get() = field / 2`.

You can also create properties that don't actually have a backing field, but just reference another property:

```kotlin
val isNewborn
    get() = age == 0
```

Note that even though this is a read-only property due to declaring it with `val` (in which case you may not provide a setter), its value can still change since it reads from a mutable property - you just can't assign to the property. Also, note that the property type is inferred from the return value of the getter.

The indentation in front of the accessors is due to convention; like elsewhere in Kotlin, it has no syntactic significance. The compiler can tell which accessors belong to which properties because the only legal place for an accessor is immediately after the property declaration (and there can be at most one getter and one setter) - so you can't split the property declaration and the accessor declarations. However, the order of the accessors doesn't matter.


## 成员函数

A function declared inside a class is called a _member function_ of that class. Like in Python, every invocation of a member function must be performed on an instance of the class, and the instance will be available during the execution of the function - but unlike Python, the function signature doesn't declare that: there is no explicit `self` parameter. Instead, every member function can use the keyword `this` to reference the current instance, without declaring it. Unlike Python, as long as there is no name conflict with an identically-named parameter or local variable, `this` can be omitted. If we do this inside a `Person` class with a `name` property:

```kotlin
fun present() {
    println("Hello, I'm $name!")
}
```

We can then do this:

```kotlin
val p = Person("Claire")
p.present() // Prints "Hello, I'm Claire!"
```

You could have said `${this.name}`, but that's redundant and generally discouraged. Oneliner functions can be declared with an `=`:

```kotlin
fun greet(other: Person) = println("Hello, ${other.name}, I'm $name!")
```

Apart from the automatic passing of the instance into `this`, member functions generally act like ordinary functions.

Because the set of member functions of an object is constrained to be exactly the set of member functions that are declared at compile-time in the object's class and base classes, it's not possible to add new member functions to an object or to a class at runtime, so e.g. `p.leave = fun() { println("Bye!") }` or anything of the sort won't compile.

Member function names should use `lowerCamelCase` instead of `snake_case`.


## Lateinit

Kotlin requires that every member property is initialized during instance construction. Sometimes, a class is intended to be used in such a way that the constructor doesn't have enough information to initialize all properties (such as when making a builder class or when using property-based dependency injection). In order to not have to make those properties nullable, you can use a _late-initialized property_:

```kotlin
lateinit var name: String
```

Kotlin will allow you to declare this property without initializing it, and you can set the property value at some point after construction (either directly or via a function). It is the responsibility of the class itself as well as its users to take care not to read the property before it has been set, and Kotlin allows you to write code that reads `name` as if it were an ordinary, non-nullable property. However, the compiler is unable to enforce correct usage, so if the property is read before it has been set, an `UninitializedPropertyAccessException` will be thrown at runtime.

Inside the class that declares a lateinit property, you can check if it has been initialized:

```kotlin
if (::name.isInitialized) println(name)
```

`lateinit` can only be used with `var`, not with `val`, and the type must be non-primitive and non-nullable.


## 中缀函数

You can designate a one-parameter member function or [extension function](extension-functionsproperties.html) for use as an infix operator, which can be useful if you're designing a DSL. The left operand will become `this`, and the right operand will become the parameter. If you do this inside a `Person` class that has got a `name` property:

```kotlin
infix fun marry(spouse: Person) {
    println("$name and ${spouse.name} are getting married!")
}
```

We can now do this (but it's still possible to call the function the normal way):

```kotlin
val lisa = Person("Lisa")
val anne = Person("Anne")
lisa marry anne // Prints "Lisa and Anne are getting married!"
```

All infix functions have the same [precedence](https://www.kotlincn.net/docs/reference/grammar.html#precedence) (which is shared with all the built-in infix functions, such as the bitwise functions `and`, `or`, `inv`, etc.): lower than the arithmetic operators and the `..` range operator, but higher than the Elvis operator `?:`, comparisons, logic operators, and assignments.


## 操作符

Most of the operators that are recognized by Kotlin's syntax have predefined textual names and are available for implementation in your classes, just like you can do with Python's double-underscore operator names. For example, the binary `+` operator is called `plus`. Similarly to the infix example, if you do this inside a `Person` class that has got a `name` property:

```kotlin
operator fun plus(spouse: Person) {
    println("$name and ${spouse.name} are getting married!")
}
```

With `lisa` and `anne` from the infix example, you can now do:

```kotlin
lisa + anne // Prints "Lisa and Anne are getting married!"
```

A particularly interesting operator is the function-call parenthesis pair, whose function name is `invoke` - if you implement this, you'll be able to call instances of your class as if they were functions. You can even overload it in order to provide different function signatures.

`operator` can also be used for certain other predefined functions in order to create fancy effects, such as [属性委托](inheritance.html#属性委托).

Since the available operators are hardcoded into the formal Kotlin syntax, you can not invent new operators, and overriding an operator does not affect its [precedence](https://www.kotlincn.net/docs/reference/grammar.html#precedence).


## 枚举类

Whenever you want a variable that can only take on a limited number of values where the only feature of each value is that it's distinct from all the other values, you can create an _enum class_:

```kotlin
enum class ContentKind {
    TOPIC,
    ARTICLE,
    EXERCISE,
    VIDEO,
}
```

There are exactly four instances of this class, named `ContentKind.TOPIC`, and so on. Instances of this class can be compared to each other with `==` and `!=`, and you can get all the allowable values with `ContentKind.values()`. You can also tack on more information to each instance if you need:

```kotlin
enum class ContentKind(val kind: String) {
    TOPIC("Topic"),
    ARTICLE("Article"),
    EXERCISE("Exercise"),
    VIDEO("Video"),
    ;

    override fun toString(): String {
        return kind
    }
}
```

Null safety is enforced as usual, so a variable of type `ContentKind` can not be null, unlike in Java.


## 数据类

Frequently - especially if you want a complex return type from a function or a complex key for a map - you'll want a quick and dirty class which only contains some properties, but is still comparable for equality and is usable as a map key. If you create a _data class_, you'll get automatic implementations of the following functions: `toString()` (which will produce a string containing all the property names and values), `equals()` (which will do a per-property `equals()`), `hashCode()` (which will hash the individual properties and combine the hashes), and the functions that are required to enable Kotlin to destructure an instance of the class into a declaration (`component1()`, `component2()`, etc.):

```kotlin
data class ContentDescriptor(val kind: ContentKind, val id: String) {
    override fun toString(): String {
        return kind.toString() + ":" + id
    }
}
```




---

[← 上一节：函数](functions.html) | [下一节：异常 →](exceptions.html)


---

*本资料英文原文的作者是 [Aasmund Eldhuset](https://eldhuset.net/)；其所有权属于[可汗学院（Khan Academy）](https://www.khanacademy.org/)，授权许可为 [CC BY-NC-SA 3.0 US（署名-非商业-相同方式共享）](https://creativecommons.org/licenses/by-nc-sa/3.0/us/)。请注意，这并不是可汗学院官方产品的一部分。中文版由[灰蓝天际](https://hltj.me/)译，遵循相同授权方式。*