## 声明

Functions are declared with the `fun` keyword. For the parameters, you must declare not only their names, but also their types, and you must declare the type of the value the function is intending to return. The body of the function is usually a _block_, which is enclosed in curly braces:

```kotlin
fun happyBirthday(name: String, age: Int): String {
    return "Happy ${age}th birthday, $name!"
}
```

Here, `name` must be a string, `age` must be an integer, and the function must return a string. However, you can also make a oneliner function, where the body simply is the expression whose result is to be returned. In that case, the return type is inferred, and an equals sign is used to indicate that it's a oneliner:

```kotlin
fun square(number: Int) = number * number
```

(Note that there is no `**` operator; non-square exponentiation should be done via `Math.pow()`.)

Function names should use `lowerCamelCase` instead of `snake_case`.


## 调用

Functions are called the same way as in Python:

```kotlin
val greeting = happyBirthday("Anne", 32)
```

If you don't care about the return value, you don't need to assign it to anything.


## 返回

As opposed to Python, omitting `return` at the end of a function does not implicitly return null; if you want to return null, you must do so with `return null`. If a function never needs to return anything, the function should have the return type `Unit` (or not declare a return type at all, in which case the return type defaults to `Unit`). In such a function, you may either have no `return` statement at all, or say just `return`. `Unit` is both a singleton object (which `None` in Python also happens to be) and the type of that object, and it represents "this function never returns any information" (rather than "this function sometimes returns information, but this time, it didn't", which is more or less the semantics of returning null).


## 重载

In Python, function names must be unique within a module or a class. In Kotlin, we can _overload_ functions: there can be multiple declarations of functions that have the same name. Overloaded functions must be distinguishable from each other through their parameter lists. (The types of the parameter list, together with the return type, is known as a function's _signature_, but the return type cannot be used to disambiguate overloaded functions.) For example, we can have both of these functions in the same file:

```kotlin
fun square(number: Int) = number * number
fun square(number: Double) = number * number
```

At the call sites, which function to use is determined from the type of the arguments:

```kotlin
square(4)    // Calls the first function; result is 16 (Int)
square(3.14) // Calls the second function; result is 9.8596 (Double)
```

While this example happened to use the same expression, that is not necessary - overloaded functions can do completely different things if need be (although your code can get confusing if you make functions that have very different behavior be overloads of each other).


## Vararg 与可选/命名参数

A function can take an arbitrary number of arguments, similarly to `*args` in Python, but they must all be of the same type. Unlike Python, you may declare other positional parameters after the variadic one, but there can be at most one variadic parameter. If its type is `X`, the type of the argument will be `XArray` if `X` is a primitive type and `Array<X>` if not.

```kotlin
fun countAndPrintArgs(vararg numbers: Int) {
    println(numbers.size)
    for (number in numbers) println(number)
}
```

There are no `**kwargs` in Kotlin, but you can define optional parameters with default values, and you may choose to name some or all of the parameters when you call the function (whether they've got default values or not). A parameter with a default value must still specify its type explicitly. Like in Python, the named arguments can be reordered at will at the call site:

```kotlin
fun foo(decimal: Double, integer: Int, text: String = "Hello") { ... }

foo(3.14, text = "Bye", integer = 42)
foo(integer = 12, decimal = 3.4)
```


In Python, the expression for a default value is evaluated once, at function definition time. That leads to this classic trap, where the developer hopes to get a new, empty list every time the function is called without a value for `numbers`, but instead, the same list is being used every time:

```python
def tricky(x, numbers=[]):  # Bug: every call will see the same list!
    numbers.append(x)
    print numbers
```

In Kotlin, the expression for a default value is evaluated every time the function is invoked. Therefore, you will avoid the above trap as long as you use an expression that produces a new list every time it is evaluated:

```kotlin
fun tricky(x: Int, numbers: MutableList<Int> = mutableListOf()) {
    numbers.add(x)
    println(numbers)
}
```

For this reason, you should probably not use a function with side effects as a default value initializer, as the side effects will happen on every call. If you just reference a variable instead of calling a function, the same variable will be read every time the function is invoked: `numbers: MutableList<Int> = myMutableList`. If the variable is immutable, each call will see the same value (but if the value itself is mutable, it might change between calls), and if the variable is mutable, each call will see the current value of the variable. Needless to say, these situations easily lead to confusion, so a default value initializer should be either a constant or a function call that always produces a new object with the same value.

You can call a variadic function with one array (but not a list or any other iterable) that contains all the variadic arguments, by _spreading_ it with the `*` operator (same syntax as Python):

```kotlin
val numbers = listOf(1, 2, 3)
countAndPrintArgs(*numbers.toIntArray())
```

Kotlin has inherited Java's fidgety array system, so primitive types have got their own array types and conversion functions, while any other type uses the generic `Array` type, to which you can convert with `.toTypedArray()`.

However, you can't spread a map into a function call and expect the values in the map to be passed to the parameters named by the keys - the names of the parameters must be known at compile time. If you need runtime-defined parameter names, your function must either take a map or take `vararg kwargs: Pair<String, X>` (where `X` is the "lowest common denominator" of the parameter types, in the worst case `Any?` - be prepared to have to typecast the parameter values, and note that you'll lose type safety). You can call such a function like this: `foo("bar" to 42, "test" to "hello")`, since `to` is an [中缀函数](classes.html#中缀函数) that creates a `Pair`.




---

[← 上一节：循环](loops.html) | [下一节：类 →](classes.html)


---

*本资料英文原文的作者是 [Aasmund Eldhuset](https://eldhuset.net/)；其所有权属于[可汗学院（Khan Academy）](https://www.khanacademy.org/)，授权许可为 [CC BY-NC-SA 3.0 US（署名-非商业-相同方式共享）](https://creativecommons.org/licenses/by-nc-sa/3.0/us/)。请注意，这并不是可汗学院官方产品的一部分。中文版由[灰蓝天际](https://hltj.me/)译，遵循相同授权方式。*