## 函数类型

Like in Python, functions in Kotlin are first-class values - they can be assigned to variables and passed around as parameters. The type a function is a _function type_, which is indicated with a parenthesized parameter type list and an arrow to the return type. Consider this function:  

和Python一样，Kotlin中的函数也是一等公民。他们可以被赋值给变量，也可以作为参数进行传递。函数的类型被称为 _函数类型_ ，由括号及括号内的参数类型列表、->、 及返回值类型表示。思考如下函数：

```kotlin
fun safeDivide(numerator: Int, denominator: Int) =
    if (denominator == 0) 0.0 else numerator.toDouble() / denominator
```

It takes two `Int` parameters and returns a `Double`, so its type is `(Int, Int) -> Double`. We can reference the function itself by prefixing its name with `::`, and we can assign it to a variable (whose type would normally be inferred, but we show the type signature for demonstration):  

该函数有两个`Int`类型的参数，并返回`Double`类型，所以它的类型为`(Int, Int) -> Double`。我们可以通过在函数名前加`::`来引用函数本身，赋给另一个变量（通常变量的类型会被自动推导，这里为了展示写出类型签名）：

```kotlin
val f: (Int, Int) -> Double = ::safeDivide
```

When you have a variable or parameter of function type (sometimes called a _function reference_), you can call it as if it were an ordinary function, and that will cause the referenced function to be called:  

一个函数类型的变量或参数（也称 _函数引用_ ），可以像原始函数一样被调用，调用时被引用的原函数将被调用：

```kotlin
val quotient = f(3, 0)
```

It is possible for a class to implement a function type as if it were an interface. It must then supply an operator function called `invoke` with the given signature, and instances of that class may then be assigned to a variable of that function type:  

类可以像实现接口一样实现函数类型。只需要提供给定签名的名为`invoke`的操作符函数即可，实现后这个类的实例就可以被赋值给函数类型变量：

```kotlin
class Divider : (Int, Int) -> Double {
    override fun invoke(numerator: Int, denominator: Int): Double = ...
}
```


## 函数字面值：lambda 表达式与匿名函数

Like in Python, you can write _lambda expressions_: unnamed function declarations with a very compact syntax, which evaluate to callable function objects. In Kotlin, lambdas can contain multiple statements, which make them useful for [more complex tasks](functional-programming.html#接收者) than the single-expression lambdas of Python. The last statement must be an expression, whose result will become the return value of the lambda (unless `Unit` is the return type of the variable/parameter that the lambda expression is assigned to, in which case the lambda has no return value). A lambda expression is enclosed in curly braces, and begins by listing its parameter names and possibly their types (unless the types can be inferred from context):  

和Python一样，你可以写 _lambda 表达式_ : **使用紧凑语法的未命名函数声明，等价于可调用的函数对象**。比起Python的单表达式lambda，Kotlin的lambda可以包含多个语句，这样在[复杂的任务](functional-programming.html#接收者)中将更加有用。Lambda最后的语句必须是表达式，该表达式的结果将作为lambda的返回值（如果lambda被赋值的变量/参数返回值类型为`Unit`，则lambda不需要返回值）。Lambda表达由花括号包裹，并列出它的参数名和类型（如果参数类型可以被上下文推导得出，则可省略）： 

```kotlin
val safeDivide = { numerator: Int, denominator: Int ->
    if (denominator == 0) 0.0 else numerator.toDouble() / denominator
}
```

The type of `safeDivide` is `(Int, Int) -> Double`. Note that unlike function type declarations, the parameter list of a lambda expression must not be enclosed in parentheses.  

`safeDivide`的类型为`(Int, Int) -> Double`。注意，和函数类型声明不同，lambda表达式的参数列表不需要小括号。

Note that the other uses of curly braces in Kotlin, such as in function and class definitions and after `if`/`else`/`for`/`while` statements, are not lambda expressions (so it is _not_ the case that `if` is a function that conditionally executes a lambda function).  

注意Kotlin的函数中、类定义，以及`if`/`else`/`for`/`while`后的花括号，不是lambda表达式（当然，括号里不知道怎么翻译）。
  
```kotlin
val condition = true  
val lambda = if (condition) { a: Int -> print(a) } else { b: Int -> print(b) }
```

The return type of a lambda expression is inferred from the type of the last expression inside it (or from the function type of the variable/parameter that the lambda expression is assigned to). If a lambda expression is passed as a function parameter (which is the ordinary use) or assigned to a variable with a declared type, Kotlin can infer the parameter types too, and you only need to specify their names:  

Lambda表达式的返回类型由最后的表达式推导出（或者从lambda表达式被赋值的变量或参数推导出）。

```kotlin
val safeDivide: (Int, Int) -> Double = { numerator, denominator ->
    if (denominator == 0) 0.0 else numerator.toDouble() / denominator
}
```

Or:

```kotlin
fun callAndPrint(function: (Int, Int) -> Double) {
    println(function(2, 0))
}

callAndPrint({ numerator, denominator ->
    if (denominator == 0) 0.0 else numerator.toDouble() / denominator
})
```

A parameterless lambda does not need the arrow. A one-parameter lambda can choose to omit the parameter name and the arrow, in which case the parameter will be called `it`:  

没有参数的lambda不需要->。一个参数的lambda可以选择省略参数名和->，可以使用`it`引用这个参数:

```kotlin
val square: (Double) -> Double = { it * it }
```

If the type of the last parameter to a function is a function type and you want to supply a lambda expression, you can place the lambda expression _outside_ of the parameter parentheses. If the lambda expression is the only parameter, you can omit the parentheses entirely. This is very useful for [constructing DSLs](functional-programming.html#接收者).  

如果函数的最后一个参数为函数类型，你想提供一个lambda表达式，你可以把lambda表达式放在参数括号外面。如果lambda表达式是唯一的参数，你可以完全省略括号。这在[构建DSL](functional-programming.html#接收者)时非常有用。

```kotlin
fun callWithPi(function: (Double) -> Double) {
    println(function(3.14))
}

callWithPi { it * it }
```

If you want to be more explicit about the fact that you're creating a function, you can make an _anonymous function_, which is still an expression rather than a declaration:  

如果你想更清晰的表达你是在创建一个函数，你可以创建一个 _匿名函数_ ，它仍然是一个表达式，而不是一个声明：

```kotlin
callWithPi(fun(x: Double): Double { return x * x })
```

Or:

```kotlin
callWithPi(fun(x: Double) = x * x)
```

Lambda expressions and anonymous functions are collectively called _function literals_.  

Lambda表达式和匿名函数统称为 _函数字面量_ 。


## 集合推导

Kotlin can get quite close to the compactness of Python's `list`/`dict`/`set` comprehensions. Assuming that `people` is a collection of `Person` objects with a `name` property:

```kotlin
val shortGreetings = people
    .filter { it.name.length < 10 }
    .map { "Hello, ${it.name}!" }
```

corresponds to

```python
short_greetings = [
    f"Hello, {p.name}"  # In Python 2, this would be: "Hello, %s!" % p.name
    for p in people
    if len(p.name) < 10
]
```

In some ways, this is easier to read because the operations are specified in the order they are applied to the values. The result will be an immutable `List<T>`, where `T` is whichever type is produced by the transformations you use (in this case, `String`). If you need a mutable list, call `toMutableList()` at the end. If you want a set, call `toSet()` or `toMutableSet()` at the end. If you want to transform a collection into a map, call `associateBy()`, which takes two lambdas that specify how to extract the key and the value from each element: `people.associateBy({it.ssn}, {it.name})` (you can omit the second lambda if you want the entire element to be the value, and you can call `toMutableMap()` at the end if you want the result to be mutable).

These transformations can also be applied to `Sequence<T>`, which is similar to Python's generators and allows for lazy evaluation. If you have a huge list and you want to process it lazily, you can call `asSequence()` on it.

There's a vast collection of functional programming-style operations available in the [`kotlin.collections` package](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index.html).


## 接收者

The signature of a member function or an [extension function](extension-functionsproperties.html) begins with a _receiver_: the type upon which the function can be invoked. For example, the signature of `toString()` is `Any.() -> String` - it can be called on any non-null object (the receiver), it takes no parameters, and it returns a `String`. It is possible to write a lambda function with such a signature - this is called a _function literal with receiver_, and is extremely useful for building DSLs.  

类的成员函数或拓展函数的签名，都有一个 _接收者_ ：函数被哪个对象调用的对象的类型。例如，`toString()`的签名是`Any.() -> String` - 它可以在任何非空对象（接收者）上调用， 该方法不接受参数，返回`String`类型。如果一个lambda是这样的类型，则被称为 **有接收者的函数字面量**，

A function literal with receiver is perhaps easiest to think of as an extension function in the form of a lambda expression. The declaration looks like an ordinary lambda expression; what makes it take a receiver is the context - it must be passed to a function that takes a function with receiver as a parameter, or assigned to a variable/property whose type is a function type with receiver. The only way to use a function with receiver is to invoke it on an instance of the receiver class, as if it were a member function or extension function. For example:  

有接收者的函数字面量，可以简单的理解为以lambda表达式形式的扩展函数。它的声明就像普通的lambda表达式，是上下文决定了它需要有一个接收者 - 它必须被传给一个以有接收者的函数为参数的函数，或者是被赋值给有接收者的函数类型的变量或参数。唯一使用有接收者的函数的方法，是在接收者类的实例上直接调用，就像是它的一个成员方法或扩展函数，例如：

```kotlin
class Car(val horsepowers: Int)

val boast: Car.() -> String = { "I'm a car with $horsepowers HP!"}

val car = Car(120)
println(car.boast())
```

Inside a lambda expression with receiver, you can use `this` to refer to the receiver object (in this case, `car`). As usual, you can omit `this` if there are no naming conflicts, which is why we can simply say `$horsepowers` instead of `${this.horsepowers}`. So beware that in Kotlin, `this` can have different meanings depending on the context: if used inside (possibly nested) lambda expressions with receivers, it refers to the receiver object of the innermost enclosing lambda expression with receiver. If you need to "break out" of the function literal and get the "original" `this` (the instance the member function you're inside is executing on), mention the containing class name after `this@` - so if you're inside a function literal with receiver inside a member function of Car, use `this@Car`.

As with other function literals, if the function takes one parameter (other than the receiver object that it is invoked on), the single parameter is implicitly called `it`, unless you declare another name. If it takes more than one parameter, you must declare their names.

Here's a small DSL example for constructing tree structures:

```kotlin
class TreeNode(val name: String) {
    val children = mutableListOf<TreeNode>()

    fun node(name: String, initialize: (TreeNode.() -> Unit)? = null) {
        val child = TreeNode(name)
        children.add(child)
        if (initialize != null) {
            child.initialize()
        }
    }
}

fun tree(name: String, initialize: (TreeNode.() -> Unit)? = null): TreeNode {
    val root = TreeNode(name)
    if (initialize != null) {
        root.initialize()
    }
    return root
}

val t = tree("root") {
    node("math") {
        node("algebra")
        node("trigonometry")
    }
    node("science") {
        node("physics")
    }
}
```

The block after `tree("root")` is the first function literal with receiver, which will be passed to `tree()` as the `initialize` parameter. According to the parameter list of `tree()`, the receiver is of type `TreeNode`, and therefore, `tree()` can call `initialize()` on `root`. `root` then becomes `this` inside the scope of that lambda expression, so when we call `node("math")`, it implicitly says `this.node("math")`, where `this` refers to the same `TreeNode` as `root`. The next block is passed to `TreeNode.node()`, and is invoked on the first child of the `root` node, namely `math`, and inside it, `this` will refer to `math`.

If we had wanted to express the same thing in Python, it would have looked like this, and we would be hamstrung by the fact that lambda functions can only contain one expression, so we need explicit function definitions for everything but the oneliners:

```python
class TreeNode:
    def __init__(self, name):
        self.name = name
        self.children = []

    def node(self, name, initialize=None):
        child = TreeNode(name)
        self.children.append(child)
        if initialize:
            initialize(child)

def tree(name, initialize=None):
    root = TreeNode(name)
    if initialize:
        initialize(root)
    return root

def init_root(root):
    root.node("math", init_math)
    root.node("science",
              lambda science: science.node("physics"))

def init_math(math):
    math.node("algebra")
    math.node("trigonometry")

t = tree("root", init_root)
```

The official docs also have a very cool example with a [ DSL for constructing HTML documents](https://www.kotlincn.net/docs/reference/type-safe-builders.html).


## 内联函数

There's a little bit of runtime overhead associated with lambda functions: they are really objects, so they must be instantiated, and (like other functions) calling them takes a little bit of time too. If we use the `inline` keyword on a function, we tell the compiler to _inline_ both the function and its lambda parameters (if any) - that is, the compiler will copy the code of the function (and its lambda parameters) into _every_ callsite, thus eliminating the overhead of both the lambda instantiation and the calling of the function and the lambdas. This will happen unconditionally, unlike in C and C++, where `inline` is more of a hint to the compiler. This will cause the size of the compiled code to grow, but it may be worth it for certain small but frequently-called functions.

```kotlin
inline fun time(action: () -> Unit): Long {
    val start = Instant.now().toEpochMilli()
    action()
    return Instant.now().toEpochMilli() - start
}
```

Now, if you do:

```kotlin
val t = time { println("Lots of code") }
println(t)
```

The compiler will generate something like this (except that `start` won't collide with any other identifiers with the same name):

```kotlin
val start = Instant.now().toEpochMilli()
println("Lots of code")
val t = Instant.now().toEpochMilli() - start
println(t)
```

In an inline function definition, you can use `noinline` in front of any function-typed parameter to prevent the lambda that will be passed to it from also being inlined.


## 不错的工具函数


### `run()`、`let()` 与 `with()`

`?.` is nice if you want to call a function on something that might be null. But what if you want to call a function that takes a non-null parameter, but the value you want to pass for that parameter might be null? Try `run()`, which is an extension function on `Any?` that takes a lambda with receiver as a parameter and invokes it on the value that it's called on, and use `?.` to call `run()` only if the object is non-null:

```kotlin
val result = maybeNull?.run { functionThatCanNotHandleNull(this) }
```

If `maybeNull` is null, the function won't be called, and `result` will be null; otherwise, it will be the return value of `functionThatCanNotHandleNull(this)`, where `this` refers to  `maybeNull`. You can chain `run()` calls with `?.` - each one will be called on the previous result if it's not null:

```kotlin
val result = maybeNull
    ?.run { firstFunction(this) }
    ?.run { secondFunction(this) }
```

The first `this` refers to `maybeNull`, the second one refers to the result of `firstFunction()`, and `result` will be the result of `secondFunction()` (or null if `maybeNull` or any of the intermediate results were null).

A syntactic variation of `run()` is `let()`, which takes an ordinary function type instead of a function type with receiver, so the expression that might be null will be referred to as `it` instead of `this`.

Both `run()` and `let()` are also useful if you've got an expression that you need to use multiple times, but you don't care to come up with a variable name for it and make a null check:

```kotlin
val result = someExpression?.let {
   firstFunction(it)
   it.memberFunction() + it.memberProperty
}
```

Yet another version is `with()`, which you can also use to avoid coming up with a variable name for an expression, but only if you know that its result will be non-null:

```kotlin
val result = with(someExpression) {
   firstFunction(this)
   memberFunction() + memberProperty
}
```

In the last line, there's an implicit `this.` in front of both `memberFunction()` and `memberProperty` (if these exist on the type of `someExpression`). The return value is that of the last expression.


### `apply()` 与 `also()`

If you don't care about the return value from the function, but you want to make one or more calls involving something that might be null and then keep on using that value, try `apply()`, which returns the value it's called on. This is particularly useful if you want to work with many members of the object in question:

```kotlin
maybeNull?.apply {
    firstFunction(this)
    secondFunction(this)
    memberPropertyA = memberPropertyB + memberFunctionA()
}?.memberFunctionB()
```

Inside the `apply` block, `this` refers to `maybeNull`. There's an implicit `this` in front of `memberPropertyA`, `memberPropertyB`, and `memberFunctionA` (unless these don't exist on `maybeNull`, in which case they'll be looked for in the containing scopes). Afterwards, `memberFunctionB()` is also invoked on `maybeNull`.

If you find the `this` syntax to be confusing, you can use `also` instead, which takes ordinary lambdas:

```kotlin
maybeNull?.also {
    firstFunction(it)
    secondFunction(it)
    it.memberPropertyA = it.memberPropertyB + it.memberFunctionA()
}?.memberFunctionB()
```


### `takeIf()` 与 `takeUnless()`

If you want to use a value only if it satisfies a certain condition, try `takeIf()`, which returns the value it's called on if it satisfies the given predicate, and null otherwise. There's also `takeUnless()`, which inverts the logic. You can follow this with a `?.` to perform an operation on the value only if it satisfies the predicate. Below, we compute the square of some expression, but only if the expression value is at least 42:

```kotlin
val result = someExpression.takeIf { it >= 42 } ?.let { it * it }
```




---

[← 上一节：空安全](null-safety.html) | [下一节：包与导入 →](packages-and-imports.html)


---

*本资料英文原文的作者是 [Aasmund Eldhuset](https://eldhuset.net/)；其所有权属于[可汗学院（Khan Academy）](https://www.khanacademy.org/)，授权许可为 [CC BY-NC-SA 3.0 US（署名-非商业-相同方式共享）](https://creativecommons.org/licenses/by-nc-sa/3.0/us/)。请注意，这并不是可汗学院官方产品的一部分。中文版由[灰蓝天际](https://hltj.me/)译，遵循相同授权方式。*